# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Gestión en GitHub Web y Métodos HTTP (GET, POST, PUT, DELETE)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez  
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Enrutamiento manual por verbo HTTP  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La dificultad que encontré fue validar que el dato recibido no estuviera vacío y configurar la respuesta correcta cuando la información no cumplía con lo solicitado. Para solucionarlo, utilicé una validación con strip() y una respuesta HTTP 400 cuando el dato estaba vacío. También comprobé que la petición POST procesara correctamente la información recibida.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Especificación semántica de verbos e idempotencia |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado

Actúa como un arquitecto de software especializado en APIs REST usando Python y FastAPI.

Necesito crear una API para administrar un recurso en la ruta /api/recurso.

La API debe permitir las siguientes operaciones:
- GET: consultar el recurso.
- POST: crear un recurso.
- PUT: actualizar el recurso.
- DELETE: eliminar el recurso.

Utiliza Pydantic para validar los datos recibidos y verifica que el campo "dato" no esté vacío.

Usa códigos HTTP adecuados:
- 200 OK para operaciones exitosas que corresponda.
- 201 Created cuando se cree un recurso.
- 400 Bad Request cuando el dato esté vacío.
- 404 Not Found cuando el recurso no exista.
- 409 Conflict cuando se intente crear un recurso que ya existe.
- 405 Method Not Allowed cuando se utilice un método HTTP no permitido.
- 422 Unprocessable Entity cuando los datos recibidos no cumplan con el esquema.

Asegúrate de que PUT y DELETE sean idempotentes, es decir, que repetir la misma petición no produzca cambios adicionales en el estado del recurso.

Entrega el código completo, organizado y listo para ejecutarse en Google Colab.

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
```

%%writefile app_ia.py
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

app = FastAPI(title="API Optimizada con IA")


# Validar los datos recibidos
class PayloadRequest(BaseModel):
    dato: str


# Recurso almacenado temporalmente
recurso = None


# GET: consultar el recurso
@app.get("/api/recurso")
def consultar_recurso():
    if recurso is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Recurso no encontrado"
        )

    return recurso


# POST: crear un recurso
@app.post("/api/recurso", status_code=status.HTTP_201_CREATED)
def crear_recurso(payload: PayloadRequest):
    global recurso

    if not payload.dato.strip():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Dato vacío"
        )

    if recurso is not None:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="El recurso ya existe"
        )

    recurso = {"dato": payload.dato}

    return {
        "status": "success",
        "mensaje": f"Creado: {payload.dato}"
    }


# PUT: actualizar el recurso
@app.put("/api/recurso")
def actualizar_recurso(payload: PayloadRequest):
    global recurso

    if not payload.dato.strip():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Dato vacío"
        )

    recurso = {"dato": payload.dato}

    return {
        "status": "success",
        "mensaje": f"Actualizado: {payload.dato}"
    }


# DELETE: eliminar el recurso
@app.delete("/api/recurso")
def eliminar_recurso():
    global recurso

    recurso = None

    return {
        "status": "success",
        "mensaje": "Recurso eliminado"
    }

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de métodos no permitidos (HTTP 405)

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> Al revisar el código, identifiqué que era necesario comprobar qué sucedía cuando se utilizaba un método HTTP no permitido. Para verificarlo, agregué una prueba con el método PATCH, que no está definido en la ruta /api/recurso, y comprobé que FastAPI respondiera con el código HTTP 405.

También consideré la importancia de utilizar códigos HTTP adecuados para cada operación y de comprobar que las peticiones PUT y DELETE fueran idempotentes. Estas pruebas adicionales ayudan a verificar que la API responda correctamente ante diferentes situaciones.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
