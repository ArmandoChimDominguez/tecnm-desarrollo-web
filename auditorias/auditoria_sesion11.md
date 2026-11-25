# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Arquitectura en Capas: Router, Servicio y Capa de Datos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez  
**Matrícula:** 23531079  
**Track:** Python / FastAPI  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Separación manual de controladores y lógica  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender la separación de responsabilidades. Al principio, me costó comprender que el `router.py` no debía importar el `repositorio.py`, sino que debía pasar por el `servicio.py`. También fue un reto asegurarme de que las variables globales (como el contador de IDs) se mantuvieran correctamente en el repositorio para que el `POST` devolviera el ID correcto y el `GET` mantuviera el estado de la lista de libros entre peticiones.

---


## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Patrón Service Layer desacoplado |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como Arquitecto backend experto en patrones de diseño. Necesito refactorizar una API de libros en 4 capas. 
1. 'repositorio.py': Solo guarda y recupera libros en un diccionario en memoria (sin conocer HTTP). 
2. 'servicio.py': Contiene la lógica de negocio, llama al repositorio. 
3. 'router.py': Define un APIRouter con los endpoints GET y POST, validando con Pydantic (min_length=1 para el título). 
4. 'app.py': Crea la app FastAPI e incluye el router con include_router. 
Mantén los códigos HTTP semánticos: 200 para GET, 201 para POST y 422 para datos inválidos."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
# Código devuelto por el LLM (Archivos clave)

# --- router.py ---
from fastapi import APIRouter, status
from pydantic import BaseModel, Field
import servicio

router = APIRouter()

class LibroRequest(BaseModel):
    titulo: str = Field(..., min_length=1)
    anio: int

@router.get("/api/libros")
def listar():
    return {"libros": servicio.obtener_libros()}

@router.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear(libro: LibroRequest):
    return servicio.agregar_libro(libro.titulo, libro.anio)

# --- servicio.py ---
import repositorio

def agregar_libro(titulo: str, anio: int) -> dict:
    return repositorio.crear(titulo, anio)

def obtener_libros() -> list:
    return repositorio.obtener_todos()

# --- repositorio.py ---
from typing import Dict, List

_db_libros: Dict[int, dict] = {}
_contador_id: int = 1

def crear(titulo: str, anio: int) -> dict:
    global _contador_id
    libro = {"id": _contador_id, "titulo": titulo, "anio": anio}
    _db_libros[_contador_id] = libro
    _contador_id += 1
    return libro

def obtener_todos() -> List[dict]:
    return list(_db_libros.values())
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de aislamiento de errores en lógica de negocio

* [☑] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [☑] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [☑] Encabezados `Content-Type: application/json` correctamente configurados.


LO MISMP PARA ESTE



## 🖥️ Salida de la Verificación
 
```text
==============================================================================
VERIFICACIÓN DEL RESULTADO · 4 casos · app.py
==============================================================================
 
✅ 01. El listado inicia vacío
✅ 02. El servicio crea un libro
✅ 03. El listado conserva el contrato
✅ 04. La validación del router sigue funcionando
 
------------------------------------------------------------------------------
RESULTADO: 4/4 casos correctos. ¡Resultado logrado!
 
✅ Existe la capa de router
✅ La app incluye el router
✅ Existe la capa de servicio
✅ Existe la capa de repositorio
 
------------------------------------------------------------------------------
RESULTADO: 4/4 comprobaciones correctas. ¡Resultado logrado!
```

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente la validación del titulo en el router.py con min_length=1. Al no tener esa validación, el test del caso 4 (datos inválidos con anio: "mil novecientos") fallaba porque Pydantic no lo detectaba antes de llegar al servicio.
>
> Agregué Field(..., min_length=1) al modelo LibroRequest en el router, y aseguré que el anio se validara automáticamente con Pydantic. De esta forma, el router rechaza las peticiones inválidas con HTTP 422 antes de tocar la lógica de negocio, manteniendo las capas limpias y seguras.

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue la verificación de los archivos (Fase 3). Al principio no entendía por qué el verificador pedía `APIRouter` en el router y `include_router` en la app. Cuando lo revisé, me di cuenta de que el profesor quería comprobar que realmente estábamos usando las herramientas de modularización de FastAPI y no solo escondiendo el código en archivos separados sin usarlos.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar un diccionario en el repositorio (`db_libros = {}`) en lugar de una lista, igual que en el hito anterior, para poder buscar por ID en tiempo constante. Sin embargo, en esta sesión el contrato no lo pedía, así que lo hice pensando en el rendimiento futuro. También mantuve las variables de estado en el repositorio y no en el servicio, para que el servicio fuera puramente lógica de negocio.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que el router no estaba validando el `titulo` correctamente. Aunque Pydantic convierte `anio: "mil novecientos"` en un 422, el `titulo` vacío pasaba. Lo corregí agregando `Field(min_length=1)` en el modelo `LibroRequest` para que cualquier título vacío fuera rechazado automáticamente con un 422.
>
---
**Firma digital:** ArmandoChimDominguez
