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
> La dificultad fue entender que al lanzar un `HTTPException` de FastAPI, el framework automáticamente lo formatea a JSON. Sin embargo, si hay un error en el código que no sea una excepción controlada (como un error de sintaxis o división por cero), el servidor de Python tira un error 500 y ensucia la consola. Aprendí que las excepciones manuales son útiles, pero no cubren los errores inesperados del sistema.

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
"Actúa como experto en FastAPI. Necesito implementar un manejador global de excepciones usando `@app.exception_handler(Exception)` y `JSONResponse`. La idea es que si ocurre un error interno (como una división por cero), el servidor no se caiga ni devuelva un HTML feo, sino que retorne un JSON estructurado con el código 500, un mensaje amigable y la ruta afectada."
## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
from fastapi import FastAPI, HTTPException, Request, status
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI(title="API Científico - Sesión 08")

class PayloadRequest(BaseModel):
    dato: str

# --- RUTA DE COMPATIBILIDAD PARA LA PRUEBA DEL PROFESOR ---
@app.post("/api/recurso")
def procesar_recurso(payload: PayloadRequest):
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}

# --- RETO DEL CIENTÍFICO: Manejador Global de Excepciones ---
# Esto captura CUALQUIER error no controlado (que normalmente daría un 500 feo)
@app.exception_handler(Exception)
async def manejador_global_errores(request: Request, exc: Exception):
    # Retornamos un JSON estructurado en lugar de dejar que el servidor colapse
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "status": "error",
            "mensaje": "Ha ocurrido un error interno en el servidor.",
            "detalle_tecnico": str(exc), # Útil para debug
            "ruta_afectada": request.url.path
        }
    )

# Ruta para probar el manejador global (provocamos un error a propósito)
@app.get("/api/error-critico")
def provocar_error():
    # Esto lanzará un ZeroDivisionError, que será capturado por el manejador global
    resultado = 10 / 0 
    return {"resultado": resultado}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de captura de errores sin crashing (HTTP 500)

* ☑ Respuestas semánticas correctas (200 OK, 500 Internal Server Error controlado).
* ☑ No existen caídas del servidor (Crashing) por errores en tiempo de ejecución.
* ☑ Encabezados Content-Type: application/json correctamente configurados en errores.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió la captura de excepciones genéricas (Exception). El framework por defecto maneja los errores de validación (422) y los errores controlados (HTTPException), pero si el servidor sufre un error interno, devuelve un Internal Server Error sin estructura JSON (en muchos casos) y deja al cliente sin saber qué pasó.
> 
> Implementé un decorador global @app.exception_handler(Exception) que intercepta cualquier error inesperado y lo devuelve como un JSONResponse con código 500 estructurado. Esto garantiza que la API siempre se comunique con el cliente de forma predecible (en formato JSON) y evita que el servidor caiga (crashing) ante fallas inesperadas.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
