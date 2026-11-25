# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Inyección de Dependencias (FastAPI Depends / PHP Container)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inyector de dependencias manual por funciones  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
>  La principal dificultad fue entender el flujo de la inyección de dependencias. Al principio, intentaba crear la instancia de `RepositorioMemoria()` dentro de cada endpoint, lo que provocaba que el estado se perdiera (el libro creado en el `POST` no aparecía en el `GET`). Tuve que comprender que debía crear una función externa (`get_repositorio()`) que devolviera una instancia global, y luego usar `Depends` en la firma del endpoint para que FastAPI se encargara de pasarla como argumento.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Contenedor IoC con Depends() en endpoints |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como experto en FastAPI. Necesito implementar inyección de dependencias. Define una interfaz `IRepositorio` y una clase `RepositorioMemoria` que la implemente. Crea una función `get_repositorio()` que devuelva una instancia global. Los endpoints `/api/libros` y `/api/diagnostico` deben recibir el repositorio con `Depends(get_repositorio)`. El diagnóstico debe devolver el nombre de la clase (`RepositorioMemoria`). Usa `Field(min_length=1)` para validar el título."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI, Depends, status
from pydantic import BaseModel, Field
from abc import ABC, abstractmethod

app = FastAPI(title="API Científico - Sesión 12")

class LibroRequest(BaseModel):
    titulo: str = Field(..., min_length=1, description="Título del libro")
    anio: int

# Interfaz del Repositorio
class IRepositorio(ABC):
    @abstractmethod
    def guardar(self, titulo: str, anio: int): ...
    @abstractmethod
    def obtener_todos(self): ...

# Implementación concreta
class RepositorioMemoria(IRepositorio):
    def __init__(self):
        self._libros = {}
        self._contador_id = 1
    def guardar(self, titulo: str, anio: int):
        libro = {"id": self._contador_id, "titulo": titulo, "anio": anio}
        self._libros[self._contador_id] = libro
        self._contador_id += 1
        return libro
    def obtener_todos(self):
        return list(self._libros.values())

# Instancia global y función inyectora
_repo_instancia = RepositorioMemoria()

def get_repositorio() -> IRepositorio:
    return _repo_instancia

# Endpoints inyectados
@app.get("/api/diagnostico")
def diagnostico(repo: IRepositorio = Depends(get_repositorio)):
    return {"repositorio": repo.__class__.__name__}

@app.post("/api/libros", status_code=status.HTTP_201_CREATED)
def crear_libro(libro: LibroRequest, repo: IRepositorio = Depends(get_repositorio)):
    return repo.guardar(libro.titulo, libro.anio)

@app.get("/api/libros")
def listar_libros(repo: IRepositorio = Depends(get_repositorio)):
    return {"libros": repo.obtener_todos()}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de ciclo de vida de dependencias por request

* ☑ Respuestas semánticas correctas (200 OK, 201 Created, 422 Unprocessable).
* ☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
* ☑ Encabezados Content-Type: application/json correctamente configurados.
* ☑ El endpoint usa Depends.
* ☑ Existe la función de inyección get_repositorio.

## 🖥️ Salida de la Verificación

```text
==============================================================================
VERIFICACIÓN DEL RESULTADO · 4 casos · app_ia.py
==============================================================================

✅ 01. El diagnóstico revela el repositorio inyectado
✅ 02. El endpoint inyectado crea un libro
✅ 03. El repositorio conserva el estado
✅ 04. La validación sigue en el borde

------------------------------------------------------------------------------
RESULTADO: 4/4 casos correctos. ¡Resultado logrado!


from verificar import revisar_archivo, resumen

resultados = []
resultados.append(revisar_archivo("app.py", exigir=["Depends"], descripcion="El endpoint usa Depends"))
resultados.append(revisar_archivo("app.py", exigir=["get_repositorio"], descripcion="Existe la función de inyección"))
resumen(resultados)


✅ El endpoint usa Depends
✅ Existe la función de inyección

------------------------------------------------------------------------------
RESULTADO: 2/2 comprobaciones correctas. ¡Resultado logrado!
```

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente la validación del título vacío en el modelo LibroRequest. Si un usuario enviaba un título vacío, el servidor lo aceptaba y lo guardaba, violando el contrato de validación en el borde. Además, el repositorio era una instancia local, lo que rompía la persistencia entre peticiones.
>
> Agregué Field(min_length=1) al modelo LibroRequest para que Pydantic rechace automáticamente los títulos vacíos con un 422. Para la inyección, definí una instancia global _repo_instancia y una función get_repositorio(), asegurando que el mismo estado se comparta en todas las peticiones. También implementé una interfaz abstracta IRepositorio para no acoplar el endpoint a la implementación concreta.

---
**Firma digital:** ArmandoChimDominguez
