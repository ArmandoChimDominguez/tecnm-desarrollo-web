# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Especificación OpenAPI (Swagger) y ReDoc  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Anotación manual de metadata de endpoints  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue comprender cómo los parámetros del decorador (tags, summary, description) afectan directamente la estructura del JSON de OpenAPI que genera FastAPI. Al hacerlo manualmente, es fácil cometer errores de tipeo en las descripciones o etiquetas que, aunque no rompen el servidor, hacen que la interfaz de Swagger UI se vea desordenada o pierda claridad para el usuario final. Comprendí que la documentación debe tratarse con el mismo rigor que la lógica de negocio.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Contrato Swagger/OpenAPI v3.1 interactivo |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---
"Actúa como un Arquitecto backend experto en documentación de APIs. Necesito configurar una aplicación FastAPI con documentación OpenAPI v3.1 profesional. Agrega metadata global (título, versión 2.1.0, contacto). Utiliza json_schema_extra en el modelo PayloadRequest para inyectar un ejemplo de JSON en Swagger UI. Además, documenta un endpoint GET usando tags, summary, description y response_description para que la interfaz sea totalmente interactiva."

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI(
    title="API Científico - Sesión 07",
    description="API optimizada con documentación OpenAPI v3.1 interactiva.",
    version="2.1.0",
    contact={"name": "Soporte DAB-2201", "email": "soporte@tecnm.mx"}
)

class PayloadRequest(BaseModel):
    dato: str = Field(..., description="Texto a procesar", examples=["Texto de prueba"])

    model_config = {
        "json_schema_extra": {
            "examples": [{"dato": "Ejemplo de JSON para Swagger"}]
        }
    }

@app.post("/api/recurso", tags=["Compatibilidad"])
def procesar_recurso(payload: PayloadRequest):
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}

@app.get(
    "/api/libros/{libro_id}", 
    tags=["Libros"],
    summary="Obtiene un libro por su ID",
    description="Retorna los detalles de un libro. Si el ID no existe, lanza un 404.",
    response_description="Los datos del libro solicitado."
)
def obtener_libro(libro_id: int):
    if libro_id == 1:
        return {"id": 1, "titulo": "Cien Años de Soledad"}
    return {"id": libro_id, "titulo": "Libro de prueba"}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Verificación de renderizado de interfaz Swagger UI

* ☑ Respuestas semánticas correctas (200 OK, 201 Created, 400 Bad Request, 422 Unprocessable).
* ☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
* ☑ Encabezados Content-Type: application/json correctamente configurados.
* ☑ El JSON de OpenAPI (/openapi.json) se genera correctamente y contiene los esquemas y metadatos.


### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente el uso de json_schema_extra para definir ejemplos explícitos de los datos de entrada en el DTO. Al no definir ejemplos, la interfaz de Swagger UI mostraba un JSON vacío o con campos genéricos ("string"), lo que induce a errores al desarrollador al momento de usar la opción "Try it out" y puede generar peticiones mal formadas que resulten en errores 422 innecesarios.
>
> Mi solucion fue la siguinte:Implementé el bloque model_config con json_schema_extra dentro del modelo PayloadRequest. Ahora, Swagger UI renderiza un ejemplo exacto y listo para ejecutar ({"dato": "Ejemplo de JSON para Swagger"}). Además, me aseguré de agregar la metadata de contacto y versión a nivel de aplicación, garantizando que el contrato OpenAPI cumpla con los estándares profesionales y sea fácil de auditar.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
