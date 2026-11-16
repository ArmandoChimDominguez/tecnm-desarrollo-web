# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hito 1: Mock API RESTful con OpenAPI y Validation Harness  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Router y esquemas de validación  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue coordinar todos los endpoints del CRUD sin que el servidor fallara. Al hacerlo manualmente, tuve que usar bucles `for` para buscar los libros por su ID y validar con `if` si el título estaba vacío o el año era inválido. Además, manejar el contador de IDs de forma global (`global contador_id`) y asegurarme de que el endpoint DELETE devolviera un `204 No Content` (sin cuerpo) en lugar de un `200`, fue un reto porque si devolvía algo en el cuerpo, la prueba fallaba.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.13) ejecutándose en Google Colab. |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web. |
| **Especificación (E)** | Contrato formal de API de primer hito semestral. CRUD completo en `/api/libros`. Validar `titulo` (no vacío) y `anio` (1400-2100) con 422. Retornar 404 si el ID no existe. Retornar 405 si se usa un método no permitido. El modelo de salida debe llamarse `Libro` para que se registre en `components.schemas` de OpenAPI. |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos. |

### 2.2 Prompt Estructurado
```text
"Actúa como experto en FastAPI. Diseña una Mock API de libros con CRUD completo. Usa `Field` para validar que el título no esté vacío y que el año esté entre 1400 y 2100. Usa un diccionario `db_libros` para simular la base de datos. Define un modelo de respuesta llamado exactamente `Libro` y úsalo en `response_model` para que aparezca en OpenAPI. Si el ID no existe en GET, PUT o DELETE, lanza HTTP 404. El endpoint POST debe devolver 201 y el DELETE 204."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, Field

app = FastAPI(title="Mock API Científico - Hito 1")

class LibroRequest(BaseModel):
    titulo: str = Field(..., min_length=1, description="Título del libro")
    anio: int = Field(..., ge=1400, le=2100, description="Año de publicación")

# IMPORTANTE: El modelo se llama "Libro" para que aparezca en OpenAPI
class Libro(BaseModel):
    id: int
    titulo: str
    anio: int

db_libros = {}
contador_id = 1

@app.get("/api/libros")
def listar_libros():
    return {"libros": list(db_libros.values())}

@app.post("/api/libros", status_code=status.HTTP_201_CREATED, response_model=Libro)
def crear_libro(libro: LibroRequest):
    global contador_id
    nuevo_libro = {
        "id": contador_id,
        "titulo": libro.titulo,
        "anio": libro.anio
    }
    db_libros[contador_id] = nuevo_libro
    contador_id += 1
    return nuevo_libro

@app.get("/api/libros/{libro_id}", response_model=Libro)
def obtener_libro(libro_id: int):
    if libro_id not in db_libros:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
    return db_libros[libro_id]

@app.put("/api/libros/{libro_id}", response_model=Libro)
def actualizar_libro(libro_id: int, libro: LibroRequest):
    if libro_id not in db_libros:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
    
    db_libros[libro_id]["titulo"] = libro.titulo
    db_libros[libro_id]["anio"] = libro.anio
    return db_libros[libro_id]

@app.delete("/api/libros/{libro_id}", status_code=status.HTTP_204_NO_CONTENT)
def eliminar_libro(libro_id: int):
    if libro_id not in db_libros:
        raise HTTPException(status_code=404, detail="Libro no encontrado")
    del db_libros[libro_id]
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Suite integral de 10 pruebas HTTP destructivas

* ☑ Listado inicial vacío → HTTP 200.
* ☑ Creación válida → HTTP 201.
* ☑ Consulta del libro creado → HTTP 200.
* ☑ Actualización del libro → HTTP 200.
* ☑ Eliminación del libro → HTTP 204.
* ☑ El libro eliminado ya no existe → HTTP 404.
* ☑ Validación de datos inválidos → HTTP 422.
* ☑ Método no permitido → HTTP 405.
* ☑ Contrato OpenAPI disponible → HTTP 200.
### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente usar response_model en los endpoints de creación y actualización. Sin este parámetro, el esquema Libro no aparecía en components.schemas de OpenAPI, lo que hacía que el caso 9 fallara. Además, la respuesta podía incluir campos que no estaban en el contrato si el diccionario interno crecía.
>
> Agregué response_model=Libro en los endpoints de POST, GET y PUT para que FastAPI filtrara la respuesta y documentara automáticamente el esquema. También me aseguré de que el contador de IDs fuera global (global contador_id) y de que el DELETE no devolviera cuerpo para cumplir con el 204 No Content.

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue hacer que el caso 9 (OpenAPI) pasara. Al principio, el esquema `Libro` no aparecía en la documentación, y el verificador fallaba. Descubrí que debía usar `response_model=Libro` en los endpoints. Además, el caso 8 (Método no permitido) fue un alivio, porque FastAPI rechaza automáticamente el `PATCH` con un 405 sin necesidad de escribir código extra.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar un diccionario (`db_libros = {}`) en lugar de una lista para simular la base de datos. Esto me permitió buscar los libros por su ID en tiempo constante (`O(1)`), en lugar de iterar sobre una lista. También usé un modelo `Libro` separado del `LibroRequest` para que la documentación de OpenAPI fuera más clara.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que si el modelo de salida no se llamaba exactamente `Libro`, el caso 9 fallaba porque OpenAPI generaba un esquema con otro nombre (`LibroResponse`). Lo corregí renombrando el modelo de respuesta a `Libro` y asegurándome de usarlo en `response_model`.
>
---
**Firma digital:** ArmandoChimDominguez
