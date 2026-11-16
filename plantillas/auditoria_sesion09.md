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
> La principal dificultad fue entender el ciclo de vida de una petición en FastAPI usando `@app.middleware("http")`. Descubrí que la función `call_next(request)` es crucial, porque es el puente que permite que la petición continúe hacia el endpoint. Si no la llamábamos con `await`, la petición se quedaba "colgada" y el servidor nunca respondía. Además, tuve que asegurarme de calcular la diferencia de tiempo e inyectar el header `X-Process-Time` antes de que la respuesta saliera al cliente, e incluir lógica para generar un `X-Request-ID` con `uuid` si el cliente no lo enviaba.

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
> ""Actúa como experto en FastAPI. Necesito un middleware (`@app.middleware("http")`) que mida el tiempo de procesamiento usando `time.perf_counter()` y agregue las cabeceras `X-Process-Time` y `X-Request-ID`. Si el cliente envía un `X-Request-ID`, debes conservarlo; si no, genera uno con `uuid`. Asegúrate de que el middleware aplique a todas las respuestas, incluyendo los 404."
> 
## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
import time
import uuid
from fastapi import FastAPI, Request
from typing import Optional

app = FastAPI(title="API Científico - Sesión 09")

@app.get("/api/libros")
def listar_libros():
    return {"libros": []}

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    start_time = time.perf_counter()
    
    # Validamos si el cliente ya envió un ID de traza
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    
    # Ejecutamos el endpoint (y capturamos la respuesta incluso si es un error 404)
    response = await call_next(request)
    
    # Calculamos el tiempo de procesamiento
    process_time = time.perf_counter() - start_time
    
    # Inyectamos las cabeceras
    response.headers["X-Process-Time"] = f"{process_time:.6f}"
    response.headers["X-Request-ID"] = request_id
    
    return response
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de inyección de encabezados de rastreo

* ☑ Una respuesta correcta trae tiempo y traza → HTTP 200.
* ☑ El X-Request-ID del cliente se conserva → HTTP 200.
* ☑ El middleware también cubre los 404 → HTTP 404.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente cubrir el caso de los 404. Si el endpoint no existe, FastAPI lanza una excepción interna, y si no se maneja correctamente en el middleware, se pierde la oportunidad de inyectar los headers de trazabilidad. Además, no se validaba el X-Request-ID enviado por el cliente, lo que podría permitir ataques de inyección de cabeceras (Header Injection) si un atacante envía un ID con saltos de línea.
>
>  El middleware captura la respuesta a través de call_next(request) y aplica los headers independientemente del código de estado (200, 404, 500). Para el X-Request-ID, limité su validación asegurando que solo se acepten caracteres alfanuméricos y guiones, y en caso de no ser válido, se genera uno nuevo con uuid4(). Esto garantiza la trazabilidad sin exponer el servidor a ataques de inyección.
---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue hacer que el middleware se aplicara también a los 404. Al principio pensaba que el middleware solo se ejecutaba si la ruta existía, pero descubrí que `await call_next(request)` devuelve una `Response` incluso si es un error 404. Lo difícil fue entender que debía inyectar los headers **después** de `call_next` para que se aplicaran a cualquier respuesta.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar `time.perf_counter()` en lugar de `time.time()` para medir el tiempo de procesamiento, ya que `perf_counter` es más precisa para medir intervalos cortos y no se ve afectada por los cambios de hora del sistema. Además, limité el formato del `X-Process-Time` a 6 decimales para que sea consistente y fácil de leer.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que si el cliente enviaba un `X-Request-ID` con espacios o caracteres especiales, el header se inyectaba tal cual en la respuesta, lo cual podía romper la sintaxis de HTTP. Lo corregí validando el valor y, si no cumplía con el formato esperado, generaba un nuevo ID con `uuid4()`.
>
---
**Firma digital:** https://github.com/ArmandoChimDominguez
