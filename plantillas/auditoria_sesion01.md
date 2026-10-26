# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Encuadre, Diagnóstico HTTP y Servidor en Colab  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Armando Chim Dominguez]  
**Matrícula:** [23531079]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Procesamiento manual de JSON de petición sin IA  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La dificuultad principal que encontre al elaborar la practrica fue como entendr como poder recibir 
los datos de las peticiones y comprobar que el JSON tuviera la información correcta.
También ocupaba que el servidor respondiera de manera correcta cuando los datos estuvieran mal escritos.
Por eso, utilicé FastAPI y Pydantic para facilitar la recepción y validación de los mensajes.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Contrato OpenAPI básico para endpoints POST |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
Actúa como un asistente de programación especializado en Python y FastAPI.

Contexto:
Estoy realizando una práctica de Desarrollo de Aplicaciones Web en Google Colab.

Requisitos:
1. Crear la ruta GET /api/estado, que devuelva:
   {"servidor": "ok", "curso": "DAB-2201"}
   con código HTTP 200.

2. Crear la ruta POST /api/eco, que reciba un JSON con la
   propiedad "mensaje" y devuelva el mensaje recibido junto
   con "recibido": true.

3. Si el JSON está mal formado, debe responder con HTTP 422
   sin provocar un error HTTP 500.

4. Si se utiliza GET en /api/eco, debe responder con HTTP 405.

Utiliza Pydantic para validar los datos recibidos.
Entrega el código completo de app.py y explica brevemente
cómo funciona.
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de códigos de estado HTTP 200, 400 y 422

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 

---
**Firma digital:** [Usuario de GitHub]
