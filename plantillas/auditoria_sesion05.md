# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Pydantic v2 / DTOs en PHP y Tipado Estático  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* DTO manual con validación de tipos primitivos  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender cómo manejar la coerción automática de Python. Cuando declaraba un modelo básico esperando un entero (cantidad: int), Python automáticamente convertía un string numérico como "5" a un entero 5, permitiendo que datos mal tipados pasaran la validación. Para resolver esto de forma manual, tuve que obligar al DTO a recibir todo como str y luego usar métodos como .isdigit() y bloques try/except dentro de la función para validar y convertir los datos, lanzando los errores 400/422 manualmente si algo fallaba.


---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Esquema Pydantic v2 con validadores personalizados |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como un Arquitecto backend experto en Pydantic v2. Necesito implementar un endpoint POST /api/producto/optimizado en FastAPI. El DTO debe recibir un nombre (mínimo 3 caracteres) y una cantidad. Requisitos estrictos: 1. La cantidad debe ser un entero estricto (usar StrictInt) para evitar que strings como '5' sean aceptados por coerción, y debe ser mayor a 0 (gt=0). 2. Usa un @field_validator para limpiar el nombre (quitar espacios y capitalizarlo). Entrégame solo el código modular con los códigos HTTP semánticos."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI
from pydantic import BaseModel, Field, field_validator, StrictInt

app = FastAPI(title="API Científico - Sesión 05")

# DTO para la prueba del profesor
class PayloadRequest(BaseModel):
    dato: str

# DTO Optimizado con Pydantic v2
class ProductoOptimizado(BaseModel):
    nombre: str = Field(..., min_length=3)
    cantidad: StrictInt = Field(..., gt=0)
    precio: float = Field(..., gt=0)

    @field_validator('nombre')
    @classmethod
    def nombre_sin_espacios(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('El nombre no puede estar vacío')
        return v.strip().title()

@app.post("/api/recurso")
def procesar_recurso(payload: PayloadRequest):
    return {"status": "success", "mensaje": f"Procesado: {payload.dato}"}

@app.post("/api/producto/optimizado")
def crear_producto_optimizado(prod: ProductoOptimizado):
    return {
        "status": "success", 
        "mensaje": "Validado por Pydantic v2",
        "producto": prod.nombre,
        "cantidad": prod.cantidad
    }
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de tipos incompatibles y coerción

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente el uso del tipado estricto (StrictInt) para la cantidad. Esto es una falla de seguridad y lógica de negocio, ya que Pydantic v2 por defecto realiza "coerción de tipos"; es decir, si el usuario envía "cantidad": "100" (un string), el servidor lo acepta y lo convierte a entero. Esto puede causar problemas graves si se guarda en una base de datos o se procesa matemáticamente (ej. inyecciones de datos o errores lógicos).
>
> Forma en la que lo solucione fue implementé el tipo StrictInt en la definición del modelo, lo que obliga a que el JSON de entrada contenga un número entero real, rechazando cualquier string con un HTTP 422. Además, agregué el validador @field_validator para limpiar el campo nombre, asegurando que no se guarden datos con espacios en blanco o formatos inconsistentes. Finalmente, verifiqué que el código de estado 500 nunca se ejecutara ante datos mal formados.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
