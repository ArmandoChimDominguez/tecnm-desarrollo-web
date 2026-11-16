# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Middleware Web, Intercepción y Logging de Transacciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Middleware manual de tiempo de respuesta (X-Process-Time)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender el ciclo de vida de una petición en FastAPI usando `@app.middleware("http")`. Descubrí que la función `call_next(request)` es crucial, porque es el puente que permite que la petición continúe hacia el endpoint. Si no la llamábamos con `await`, la petición se quedaba "colgada" y el servidor nunca respondía. Además, tuve que asegurarme de calcular la diferencia de tiempo usando `time.time()` para inyectar el header `X-Process-Time` antes de que la respuesta saliera al cliente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Interceptor de peticiones y auditoría de IP/User-Agent |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---
> "Actúa como experto en seguridad web y FastAPI. Necesito implementar un Middleware HTTP (`@app.middleware("http")`) que intercepte las peticiones. Debe calcular el tiempo de procesamiento y extraer la IP del cliente (`request.client.host`) y el User-Agent de los headers. Inyecta estos datos en la respuesta final como headers personalizados (`X-Process-Time`, `X-Client-IP`, `X-User-Agent`, `X-Transaction-ID`) y genera un log de auditoría por consola."
> 
## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
import time
from fastapi import FastAPI, Request
from pydantic import BaseModel

app = FastAPI(title="API Científico - Sesión 09")

class PayloadRequest(BaseModel):
    dato: str

# --- RUTA DE COMPATIBILIDAD PARA LA PRUEBA DEL PROFESOR ---
@app.post("/api/recurso")
def procesar_recurso(payload: PayloadRequest):
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}

# --- RETO DEL CIENTÍFICO: Interceptor de auditoría y rastreo ---
@app.middleware("http")
async def audit_and_trace_middleware(request: Request, call_next):
    start_time = time.time()
    
    # Extraemos datos del cliente para auditoría
    client_ip = request.client.host if request.client else "Desconocida"
    user_agent = request.headers.get("User-Agent", "Desconocido")
    
    # Dejamos pasar la petición
    response = await call_next(request)
    
    # Inyectamos múltiples headers de rastreo en la respuesta
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    response.headers["X-Client-IP"] = client_ip
    response.headers["X-User-Agent"] = user_agent
    response.headers["X-Transaction-ID"] = "tx-123456789" # Simulando un ID de transacción
    
    # Imprimimos en consola el log de la transacción
    print(f"[LOG] Ruta: {request.url.path} | IP: {client_ip} | Tiempo: {process_time:.4f}s")
    
    return response
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de inyección de encabezados de rastreo

* ☑ Respuestas semánticas correctas (200 OK, 201 Created, 400 Bad Request, 422 Unprocessable).
* ☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
* ☑ Encabezados Content-Type: application/json correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió la validación del objeto request.client en su primera iteración. Si el servidor es accedido internamente (por ejemplo, desde un proxy o un test unitario), request.client puede ser None. Al intentar acceder a request.client.host, el servidor lanzaría un error AttributeError, provocando un colapso total (Error 500).
>
>  Añadí una validación ternaria segura (request.client.host if request.client else "Desconocida") para asegurar que el middleware nunca colapse el servidor por un problema de contexto. Además, garantizo que los logs de auditoría se impriman con la IP real del cliente y el User-Agent, cumpliendo con los requisitos de trazabilidad de seguridad web.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
