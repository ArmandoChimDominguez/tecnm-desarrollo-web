# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Headers, Content-Type y Negociación de Contenido  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez  
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lectura y emisión manual de encabezados HTTP  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender el orden de ejecución de FastAPI. Al querer validar el Content-Type manualmente, me di cuenta de que si declaraba el modelo de Pydantic directamente en la firma de la función, FastAPI intentaba parsear el cuerpo antes de que mi código pudiera leer el encabezado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Invariantes de negociación (Content-Type vs Accept) |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como un Arquitecto de software senior especialista en APIs RESTful. Necesito implementar un endpoint en FastAPI llamado POST /api/recurso. El endpoint recibe un JSON con la clave dato (string). Requisitos estrictos: 1. Si el header Content-Type de la petición NO es application/json, el servidor debe devolver un error 415 Unsupported Media Type. 2. Si el header es correcto pero el campo dato viene vacío, devolver 400 Bad Request. 3. Si todo es correcto, devolver 200 OK. Es crucial que la validación del Content-Type ocurra ANTES de que FastAPI intente validar el cuerpo con Pydantic, para evitar errores 422 prematuros. Entrégame solo el código modular y asegúrate de manejar los códigos HTTP semánticos."
```
---
"Actúa como un Arquitecto de software senior especialista en APIs RESTful. Necesito implementar un endpoint en FastAPI llamado POST /api/recurso. El endpoint recibe un JSON con la clave dato (string). Requisitos estrictos: 1. Si el header Content-Type de la petición NO es application/json, el servidor debe devolver un error 415 Unsupported Media Type. 2. Si el header es correcto pero el campo dato viene vacío, devolver 400 Bad Request. 3. Si todo es correcto, devolver 200 OK. Es crucial que la validación del Content-Type ocurra ANTES de que FastAPI intente validar el cuerpo con Pydantic, para evitar errores 422 prematuros. Entrégame solo el código modular y asegúrate de manejar los códigos HTTP semánticos."

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI, Request, HTTPException, status
from pydantic import BaseModel

app = FastAPI(title="API Científico - Sesión 03")

class PayloadRequest(BaseModel):
    dato: str

@app.post("/api/recurso")
async def procesar_recurso(request: Request):
    # 1. Validación MANUAL del Content-Type (Antes de Pydantic)
    content_type = request.headers.get("Content-Type", "")
    
    if "application/json" not in content_type:
        raise HTTPException(
            status_code=status.HTTP_415_UNSUPPORTED_MEDIA_TYPE, 
            detail="El Content-Type debe ser application/json"
        )
    
    # 2. Leemos el cuerpo manualmente y lo validamos con Pydantic
    try:
        body = await request.json()
        payload = PayloadRequest(**body)
    except Exception:
        raise HTTPException(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY, 
            detail="El cuerpo no es un JSON válido o no cumple con el esquema"
        )
    
    # 3. Validación semántica del dato
    if not payload.dato.strip():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST, 
            detail="Dato vacío"
        )
        
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tipos de contenido no soportados (415)

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió la prioridad de validación de los encabezados HTTP. En su primera iteración, el modelo sugirió declarar el objeto PayloadRequest directamente en la firma de la función. Esto provocó que, al enviar un Content-Type: text/plain, FastAPI intentara parsear el cuerpo con Pydantic antes de revisar el header, devolviendo un HTTP 422 en lugar del HTTP 415 requerido.

Modifiqué el código de la IA removiendo el DTO de la firma de la función e inyectando el objeto Request de Starlette. De esta forma, mi código ahora intercepta el header Content-Type primero.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
