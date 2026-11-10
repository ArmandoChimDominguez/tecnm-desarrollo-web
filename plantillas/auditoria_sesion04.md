# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Anatomía RESTful, URIs Nombres y Recursos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez 
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diseño de URIs jerárquicas sin verbos en la ruta  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La mayor dificultad fue dejar de pensar en "acciones" (como /getLibro o /crearLibro) y empezar a diseñar rutas basadas puramente en "recursos" (/api/libros y /api/libros/{id}). Además, al usar parámetros de ruta, tuve que validar manualmente el ID capturado; si el ID no existía en mi diccionario, el servidor fallaba con un error interno (500) si no lo controlaba. Aprendí que es obligatorio interceptar esta situación y lanzar manualmente una excepción HTTPException con el código 404 para indicar que el recurso no fue encontrado.



---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Mapeo formal de recursos y parámetros de ruta |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text

```
"Actúa como un Arquitecto backend experto en APIs RESTful. Necesito una API en FastAPI para gestionar libros usando URIs jerárquicas limpias (sin verbos como 'get' o 'crear'). Implementa GET /api/libros para listar todos y GET /api/libros/{libro_id} para obtener uno solo. Utiliza el objeto Path de FastAPI para asegurar que el ID sea un entero positivo. Si el libro no existe en el diccionario global, devuelve un HTTP 404 Not Found. Entrégame solo el código modular con las mejores prácticas."
---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
```
from fastapi import FastAPI, HTTPException, Path, status
from pydantic import BaseModel

app = FastAPI(title="API RESTful - Sesión 04")

class LibroRequest(BaseModel):
    titulo: str

class PayloadRequest(BaseModel):
    dato: str

db_libros = {}
contador_id = 1

@app.post("/api/recurso")
def procesar_recurso(payload: PayloadRequest):
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}

@app.get("/api/libros")
def listar_libros():
    return list(db_libros.values())

@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(libro: LibroRequest):
    global contador_id
    nuevo_libro = {"id": contador_id, "titulo": libro.titulo}
    db_libros[contador_id] = nuevo_libro
    contador_id += 1
    return nuevo_libro

@app.get("/api/libros/{libro_id}")
def obtener_libro(libro_id: int = Path(..., ge=1, description="ID positivo")):
    if libro_id not in db_libros:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND, 
            detail=f"El libro con ID {libro_id} no existe"
        )
    return db_libros[libro_id]
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de peticiones a recursos inexistentes (404)

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA inicialmente omitió la validación estricta de los parámetros de ruta. Aunque devolvía el 404 correctamente, permitía que se enviaran IDs negativos o cero, lo que rompía la lógica del recurso. Además, al usar int simple, no había un control sobre esos valores inválidos.

Mi correcion fue la siguiente
Implementé el uso de Path(..., ge=1) importado de FastAPI. Esto asegura que el servidor rechace automáticamente (con un HTTP 422) cualquier petición a IDs menores a 1 antes de entrar a la lógica. Asimismo, me aseguré de que si el ID es válido pero no existe, el código lance explícitamente la excepción HTTP_404_NOT_FOUND para evitar un error 500 o una respuesta vacía.

---
**Firma digital:** (https://github.com/ArmandoChimDominguez)
