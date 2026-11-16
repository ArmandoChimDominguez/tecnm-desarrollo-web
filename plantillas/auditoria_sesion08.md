# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Respuestas Semánticas y Excepciones HTTP Personalizadas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lanzamiento manual de excepciones estructuradas  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La dificultad fue entender que al lanzar una excepción simple (como `HTTPException`), FastAPI devuelve un JSON con el campo `detail`. Sin embargo, el contrato de esta sesión exigía un "sobre de error" con una estructura específica: `{"error": {"codigo": "...", "mensaje": "..."}}`. Lograr esto a mano requería interceptar los errores de validación de Pydantic (422) y los errores de negocio (404) para que devolvieran exactamente ese formato, evitando que el servidor devolviera la estructura por defecto de FastAPI o un error 500 con el *traceback* expuesto.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Manejador global de HTTPException y JSON response |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---
"Actúa como experto en FastAPI. Necesito un manejo de errores uniforme. Define una excepción `LibroNoEncontrado` y regístrala con un handler que responda 404 con el JSON `{"error": {"codigo": "NO_ENCONTRADO", "mensaje": ...}}`. Haz lo mismo para los errores de validación (422) con `codigo="VALIDACION"`. Asegúrate de que el título vacío lance 422 usando `Field(min_length=1)` y de que la API nunca devuelva un 500."
## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
from fastapi import FastAPI, Request, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field

app = FastAPI(title="API Científico - Sesión 08")

db_libros = {1: {"id": 1, "titulo": "Rayuela", "anio": 1963}}

class LibroRequest(BaseModel):
    # El min_length=1 hace que Pydantic lance 422 automáticamente si el título está vacío
    titulo: str = Field(..., min_length=1)
    anio: int

class LibroNoEncontrado(Exception):
    def __init__(self, libro_id: int):
        self.libro_id = libro_id

@app.exception_handler(LibroNoEncontrado)
async def handler_libro_no_encontrado(request: Request, exc: LibroNoEncontrado):
    return JSONResponse(
        status_code=status.HTTP_404_NOT_FOUND,
        content={"error": {"codigo": "NO_ENCONTRADO", "mensaje": f"El libro {exc.libro_id} no existe"}}
    )

@app.exception_handler(RequestValidationError)
async def handler_validacion(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={"error": {"codigo": "VALIDACION", "mensaje": "Revisa los datos enviados"}}
    )

@app.get("/api/libros/{libro_id}")
def obtener_libro(libro_id: int):
    if libro_id not in db_libros:
        raise LibroNoEncontrado(libro_id)
    return db_libros[libro_id]

@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(libro: LibroRequest):
    nuevo = {"id": len(db_libros) + 1, **libro.model_dump()}
    db_libros[nuevo["id"]] = nuevo
    return nuevo
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de captura de errores sin crashing (HTTP 500)

* ☑ Un libro existente responde 200 → HTTP 200.
* ☑ Un libro inexistente responde 404 con sobre de error → HTTP 404.
* ☑ Los datos inválidos responden 422 con sobre de error → HTTP 422.
* ☑ Un libro válido sigue respondiendo 201 → HTTP 201.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente la validación del título vacío en el POST. Si un usuario enviaba {"titulo": ""}, el código lo aceptaba como válido y lo guardaba, en lugar de rechazarlo con un 422. Además, no existía un manejador global para RequestValidationError, por lo que FastAPI devolvía su JSON por defecto (con el campo detail), violando el contrato del "sobre de error".
> 
> Agregué min_length=1 al campo titulo del modelo Pydantic, lo que hace que FastAPI lance automáticamente un 422 cuando el título está vacío. Además, implementé un manejador global para RequestValidationError que intercepta cualquier error de validación y lo devuelve con la estructura {"error": {"codigo": "VALIDACION", "mensaje": "..."}}, cumpliendo al 100% con el contrato.

---

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue hacer que el error 422 devolviera el sobre de error correcto. FastAPI, por defecto, devuelve los errores de validación con el formato `{"detail": [...]}`, pero el contrato exigía `{"error": {"codigo": "VALIDACION"}}`. Tuve que registrar un manejador global para `RequestValidationError` para interceptar ese error y reformatearlo.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí registrar dos manejadores globales: uno para la excepción de negocio `LibroNoEncontrado` y otro para `RequestValidationError`. Aunque el contrato solo decía "lanza un 404", manejar los errores de forma centralizada garantiza que todos los errores (incluyendo los inesperados) tengan el mismo formato, facilitando el consumo de la API por parte del cliente.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que si el título estaba vacío, el código lo aceptaba como válido y devolvía un 201. Lo corregí agregando `min_length=1` al campo `titulo` en el modelo Pydantic, lo que obliga a que el título tenga al menos un carácter y lance un 422 automáticamente si está vacío.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
