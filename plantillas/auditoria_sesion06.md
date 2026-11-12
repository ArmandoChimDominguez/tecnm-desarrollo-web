# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Validación Estricta, Constraints y Params Query/Body  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez 
**Matrícula:** 23531079
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Filtro manual de rangos y longitudes de texto  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
>La dificultad fue tener que llenar la función con múltiples bloques `if` para comprobar cada regla de negocio (longitud del título, rango del año, que el precio sea positivo y que el límite sea mayor a 1). Además, al hacerlo a mano, debía asegurarme de lanzar la excepción `HTTPException` con el código 422 correctamente. Un pequeño error de indentación en la función de listado provocaba que la API fallara, demostrando lo frágil que es hacer las validaciones manuales en lugar de delegarlas al DTO.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Validaciones Pydantic Field(ge=18, min_length=3) |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text


```

---
"Actúa como un Arquitecto backend experto en Pydantic v2. Necesito implementar una API de libros con validaciones estrictas. Para el endpoint `POST /api/libros`, el DTO debe exigir: `titulo` (entre 3 y 80 caracteres), `anio` (entre 1450 y 2100) y `precio` (mayor a 0) usando el objeto `Field`. Para el endpoint `GET /api/libros`, el parámetro de consulta `limite` debe validar que sea mayor o igual a 1 y menor o igual a 100 usando el objeto `Query`. No agregues validaciones con `if`, usa solo las herramientas de Pydantic y FastAPI."

## 3. Código Devuelto por el Modelo
%%writefile app_ia.py
from fastapi import FastAPI, Query, status
from pydantic import BaseModel, Field

app = FastAPI(title="API Científico - Sesión 06")

# 1. DTO con reglas de negocio inyectadas en Field
class LibroRequest(BaseModel):
    titulo: str = Field(..., min_length=3, max_length=80, description="Título del libro")
    anio: int = Field(..., ge=1450, le=2100, description="Año de publicación")
    precio: float = Field(..., gt=0, description="Precio mayor a cero")

db_libros = []

@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(libro: LibroRequest):
    # Ya no hay ifs, Pydantic ya validó todo
    nuevo_libro = {
        "id": len(db_libros) + 1,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio
    }
    db_libros.append(nuevo_libro)
    return nuevo_libro

@app.get("/api/libros")
def listar_libros(limite: int = Query(..., ge=1, le=100, description="Cantidad de libros a mostrar")):
    # Query ya validó el límite
    return db_libros[:limite]
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de desbordamiento de cadenas y números negativos

* ☑ Un libro dentro de los límites se crea → HTTP 201.
* ☑ Un título demasiado corto produce 422 → HTTP 422.
* ☑ Un año fuera de rango produce 422 → HTTP 422.
* ☑ Un precio no positivo produce 422 → HTTP 422.
* ☑ El límite de paginación debe ser al menos 1 → HTTP 422.
* ☑ Un límite válido responde 200 → HTTP 200.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente incluir el parámetro Query con los límites estrictos para la paginación, limitándose a recibir un entero normal sin restricciones, lo cual permitía que un atacante solicitara la base de datos completa (límite=0 o valores negativos). Esto es una falla de seguridad (Denial of Service o Data Leak).

Implementé el objeto Query de FastAPI inyectando las reglas ge=1 y le=100 directamente en la firma de la función. Ahora, si el cliente intenta enviar ?limite=0 o ?limite=9999, FastAPI intercepta la petición antes de entrar a la lógica y responde automáticamente con un HTTP 422 Unprocessable Entity, protegiendo los recursos del servidor. También se auditó que el DTO de creación de libros rechazara títulos cortos, años fuera de rango y precios negativos sin escribir un solo if.

---
### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue entender la diferencia entre validar con `Field` (para el cuerpo de la petición) y validar con `Query` (para los parámetros de la URL). Al principio intenté poner las restricciones del límite en el modelo de Pydantic, pero el verificador fallaba porque el Query Param se maneja por separado en FastAPI. Comprendí que debo importar `Query` explícitamente para aplicar restricciones como `ge=1, le=100` y que la lógica de validación no debe repetirse en el endpoint.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Usé una lista global (`db_libros = []`) para simular una base de datos en memoria y añadí un contador automático para el `id` de cada libro. Además, decidí usar `status.HTTP_201_CREATED` explícitamente en el decorador `@app.post` en lugar de solo retornar un diccionario, para garantizar que el código HTTP sea el correcto semánticamente.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que si no indentaba correctamente el `return` dentro de la función `listar_libros`, el servidor fallaba al intentar devolver la lista. Aunque el contrato no lo pedía explícitamente, el hecho de no tener la sangría adecuada hacía que la API lanzara un error interno. Lo corregí asegurándome de que toda la lógica de respuesta estuviera dentro de la función y con la indentación correcta.




**Firma digital:** https://github.com/ArmandoChimDominguez
