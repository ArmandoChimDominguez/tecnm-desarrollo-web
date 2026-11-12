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
> La principal dificultad fue comprender cómo los parámetros del decorador (tags, summary, description) afectan directamente la estructura del JSON de OpenAPI que genera FastAPI. Al hacerlo manualmente, es fácil cometer errores de tipeo en las descripciones o etiquetas que, aunque no rompen el servidor, hacen que la interfaz de Swagger UI se vea desordenada o pierda claridad para el usuario final. Comprendí que la documentación debe tratarse con el mismo rigor que la lógica de negocio, y que para que un modelo aparezca en `components.schemas`, es obligatorio usarlo en algún endpoint.

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
"Actúa como un Arquitecto backend experto en documentación de APIs. Necesito configurar una aplicación FastAPI llamada 'Biblioteca API', versión '1.0.0', con una descripción clara. Define un modelo Pydantic `Libro` con `titulo`, `anio` y `autor`, usando `Field` para inyectar ejemplos y descripciones en Swagger. Crea un endpoint `GET /api/libros` que devuelva `{"libros": []}` y un `POST /api/libros` que use el modelo `Libro` para que se registre en la documentación OpenAPI."


### 3. Código Devuelto por el Modelo

```python
from fastapi import FastAPI, status
from pydantic import BaseModel, Field
from typing import List

# 1. Metadatos de la API
app = FastAPI(
    title="Biblioteca API",
    version="1.0.0",
    description="API de libros del curso DAB-2201 con documentación optimizada"
)

# 2. Modelo con descripciones y ejemplos para Swagger
class Libro(BaseModel):
    titulo: str = Field(..., description="Título del libro", examples=["Rayuela"])
    anio: int = Field(..., description="Año de publicación", examples=[1963])
    autor: str = Field(..., description="Autor del libro", examples=["Julio Cortázar"])

# 3. Modelo de respuesta para envolver la lista (requerido por el contrato)
class LibroListResponse(BaseModel):
    libros: List[Libro]

# 4. Endpoints
@app.get("/api/libros", response_model=LibroListResponse)
def listar_libros():
    # El contrato exige devolver exactamente esta estructura
    return {"libros": []}

@app.post("/api/libros", response_model=Libro, status_code=status.HTTP_201_CREATED)
def crear_libro(libro: Libro):
    return libro
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Verificación de renderizado de interfaz Swagger UI

* ☑ OpenAPI responde con metadatos → HTTP 200.
* ☑ Swagger UI está disponible → HTTP 200.
* ☑ ReDoc está disponible → HTTP 200.
* ☑ El endpoint documentado responde → HTTP 200.


### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente usar response_model en el endpoint GET /api/libros. Al no declararlo, el modelo Libro no se registraba en components.schemas de OpenAPI, lo que hacía que el verificador fallara al no encontrar el esquema. Además, la respuesta no tenía la estructura {"libros": []} que exigía el contrato.
>
> Implementé un modelo de respuesta LibroListResponse que envuelve la lista de libros. De esta forma, el JSON de salida es exactamente {"libros": []} y el modelo Libro se registra automáticamente en OpenAPI. También me aseguré de que la metadata (title, version, description) coincidiera con el contrato y de que /docs y /redoc estuvieran disponibles por defecto.

---

---

Responde con tus palabras
¿Qué resultado te costó más alcanzar y por qué?

El resultado que más me costó fue hacer que el modelo Libro apareciera en components.schemas. Al principio no entendía por qué no se registraba en OpenAPI, hasta que comprendí que debía usar el modelo en un endpoint con response_model. Una vez que agregué el POST que usa Libro, el esquema apareció automáticamente en Swagger.

¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?

Decidí crear un modelo de respuesta LibroListResponse para envolver la lista. Aunque el contrato solo pedía que /api/libros devolviera {"libros": []}, envolver la lista en un modelo garantiza que el JSON de salida sea exactamente el esperado y que el esquema de OpenAPI documente correctamente el nivel superior de la respuesta.

¿Qué error detectó la verificación que tú no habías previsto?

La verificación detectó que sin el response_model en el endpoint, FastAPI no incluía el modelo Libro en la sección de esquemas de OpenAPI. Lo corregí asegurándome de que todos los modelos utilizados en las respuestas fueran declarados explícitamente con response_model.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
