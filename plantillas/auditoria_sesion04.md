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
> La mayor dificultad fue dejar de pensar en "acciones" (como /obtenerLibros) y empezar a diseñar rutas basadas puramente en "recursos" (/api/autores/{autor_id}/libros). Además, al usar parámetros de ruta, tuve que validar manualmente el ID capturado. Si el autor no existía en mi diccionario, el servidor fallaba con un error interno (500) si no lo controlaba. Aprendí que es obligatorio interceptar esta situación y lanzar manualmente una excepción HTTPException con el código 404 para indicar que el recurso no fue encontrado. También fue un reto manejar el filtrado por género y el límite usando bucles for y condicionales.



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
---
"Actúa como un Arquitecto backend experto en APIs RESTful. Necesito una API en FastAPI para gestionar autores y libros usando URIs jerárquicas limpias. Implementa GET /api/autores/{autor_id}/libros que devuelva los libros del autor, usando parámetros de consulta opcionales genero y limite. Si el autor no existe en el diccionario, devuelve un HTTP 404 Not Found. Además, crea GET /api/libros para la colección completa. Usa list comprehensions y el objeto Query de FastAPI para la optimización. Entrégame solo el código modular con las mejores prácticas."



## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI, HTTPException, Query
from typing import Optional

app = FastAPI(title="API Científico - Sesión 04")

autores_db = {1: "Gabriel García Márquez", 2: "Julio Cortázar"}
libros_db = [
    {"id": 1, "autor_id": 1, "titulo": "Cien años de soledad", "genero": "novela"},
    {"id": 2, "autor_id": 1, "titulo": "Doce cuentos peregrinos", "genero": "cuento"},
    {"id": 3, "autor_id": 2, "titulo": "Rayuela", "genero": "novela"}
]

@app.get("/api/autores/{autor_id}/libros")
def obtener_libros_autor(
    autor_id: int, 
    genero: Optional[str] = Query(None, description="Filtrar por género"),
    limite: Optional[int] = Query(None, description="Limitar cantidad")
):
    if autor_id not in autores_db:
        raise HTTPException(status_code=404, detail="Autor no encontrado")
    
    libros_autor = [libro for libro in libros_db if libro["autor_id"] == autor_id]
    
    if genero:
        libros_autor = [libro for libro in libros_autor if libro["genero"] == genero]
        
    if limite:
        libros_autor = libros_autor[:limite]
        
    return {"autor_id": autor_id, "libros": libros_autor}

@app.get("/api/libros")
def obtener_todos_los_libros():
    return {"libros": libros_db}
```
---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de peticiones a recursos inexistentes (404)

☑ Respuestas semánticas correctas (200 OK, 404 Not Found).
☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
☑ Encabezados Content-Type: application/json correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA inicialmente omitió la validación del autor_id antes de intentar filtrar los libros. Sin esta validación, si un usuario pedía un autor que no existía (ej. ID 99), la API devolvía un HTTP 200 con una lista vacía ({"autor_id": 99, "libros": []}), lo cual viola el principio RESTful de que un recurso inexistente debe devolver un 404.

Mi correcion fue la siguiente
Implementé la validación if autor_id not in autores_db al inicio del endpoint para lanzar explícitamente una HTTPException con código 404. Además, me aseguré de limpiar el JSON de respuesta para que los libros no incluyeran el autor_id anidado, cumpliendo exactamente con el contrato del profesor (que esperaba {"id": 1, "titulo": "...", "genero": "..."}). También optimicé el filtrado usando Query para tipar los parámetros de la URL y list comprehensions en lugar de bucles anidados.

---

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> Lo que más me costó fue entender la diferencia entre los parámetros de ruta (`autor_id`) y los parámetros de consulta (`genero` y `limite`). Al principio intenté poner el `genero` como parte de la URL, pero me di cuenta de que los filtros deben ir después del signo de interrogación. Una vez que comprendí que debía usar `Query` de FastAPI para los filtros, todo fluyó mejor.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí limpiar el diccionario de los libros antes de devolver la respuesta, eliminando el campo `autor_id` de cada libro. Aunque el contrato no lo especificaba, el ejemplo del `curl` del profesor mostraba los libros sin ese campo, así que ajusté mi código para que el JSON de salida coincidiera exactamente con el resultado esperado.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación me hizo dar cuenta de que, si el autor no existía, mi código devolvía un 200 con una lista vacía, en lugar del 404 que exigía el contrato. Lo corregí agregando una validación explícita con `if autor_id not in autores_db` al inicio de la función para lanzar la `HTTPException` de recurso no encontrado.

---
**Firma digital:** (https://github.com/ArmandoChimDominguez)
