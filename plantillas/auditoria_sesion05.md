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
> La principal dificultad fue entender cómo Pydantic maneja la coerción automática. Al declarar `anio: int`, Pydantic convierte automáticamente `"1966"` a `1966`, lo cual es útil, pero también significa que no podemos relajarnos con los tipos. Para que el contrato fuera estricto, tuve que usar `ConfigDict(extra="forbid")` para bloquear campos extra y `Field` para definir las restricciones. Sin embargo, hacerlo "a mano" me obligó a entender cómo Pydantic intercepta los datos antes de llegar a mi lógica.


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
"Actúa como un Arquitecto backend experto en Pydantic v2. Necesito implementar un endpoint POST /api/producto/optimizado en FastAPI. El DTO debe recibir un nombre (mínimo 3 caracteres) y una cantidad. Requisitos esDiseña un DTO de entrada para un libro que exija `titulo` (str), `anio` (int, con coerción de strings numéricos), `precio` (float) y `disponible` (bool). Bloquea cualquier campo extra con `extra="forbid"`. Define un DTO de salida que devuelva el `id` y los datos. Asegúrate de que el 422 se dispare ante datos inválidos."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
%%writefile app_ia.py
from fastapi import FastAPI, status
from pydantic import BaseModel, Field, ConfigDict
from typing import List

app = FastAPI(title="API Científico - Sesión 05")

# DTO de Entrada optimizado
class LibroRequest(BaseModel):
    titulo: str = Field(..., description="Título del libro")
    anio: int = Field(..., description="Año de publicación (se acepta coerción de string)")
    precio: float = Field(..., description="Precio del libro")
    disponible: bool = Field(..., description="Disponibilidad del libro")
    
    # Bloquea campos extra (como 'color')
    model_config = ConfigDict(extra="forbid")

# DTO de Salida
class LibroResponse(BaseModel):
    id: int
    titulo: str
    anio: int
    precio: float
    disponible: bool

db_libros = []
contador_id = 1

@app.post("/api/libros", response_model=LibroResponse, status_code=status.HTTP_201_CREATED)
def crear_libro(libro: LibroRequest):
    global contador_id
    nuevo_libro = {
        "id": contador_id,
        "titulo": libro.titulo,
        "anio": libro.anio,
        "precio": libro.precio,
        "disponible": libro.disponible
    }
    db_libros.append(nuevo_libro)
    contador_id += 1
    return nuevo_libro

@app.get("/api/libros")
def listar_libros():
    return {"libros": db_libros}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de tipos incompatibles y coerción

☑ Un libro válido se crea con 201 → HTTP 201.
☑ La cadena numérica se convierte a entero → HTTP 201.
☑ Un año no numérico produce 422 → HTTP 422.
☑ Un campo extra produce 422 → HTTP 422.
☑ El listado devuelve los libros creados → HTTP 200.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente el uso de ConfigDict(extra="forbid"). Sin esta configuración, el DTO aceptaba campos extra (como color: "rojo") y los ignoraba, lo que puede llevar a una brecha de seguridad si un atacante intenta inyectar campos no deseados.
>
> Forma en la que lo solucione. Agregué model_config = ConfigDict(extra="forbid") al DTO de entrada. Esto fuerza a que cualquier campo no definido en el modelo cause un error 422. Además, verifiqué que el response_model se usara en el endpoint para que la respuesta solo incluyera los campos del DTO de salida, evitando devolver datos que no corresponden al contrato.

---

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue hacer que Pydantic rechazara los campos extra. Al principio no sabía cómo decirle a Pydantic que "no aceptara nada más" que los campos definidos. Descubrí que `ConfigDict(extra="forbid")` es la forma correcta en Pydantic v2. Además, entender la coerción (que `"1966"` se convierta a `1966` pero `"mil novecientos"` falle) me tomó tiempo, porque confiaba en que Pydantic lo manejaría automáticamente, pero tuve que asegurarme de que el tipo `int` estuviera bien definido.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar `response_model=LibroResponse` en el endpoint POST para que FastAPI automáticamente filtre la respuesta y solo devuelva los campos definidos en el DTO de salida. Aunque el contrato solo decía "devuelve los datos del libro", usar un DTO de salida garantiza que si en el futuro el modelo cambia, la respuesta no se rompa y no exponga campos sensibles.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que sin `extra="forbid"`, el JSON con el campo `color` era aceptado sin problemas (devolvía 201 en lugar de 422). Lo corregí agregando la configuración `ConfigDict(extra="forbid")` en la clase `LibroRequest`, lo que obliga a que el contrato sea estricto y rechace cualquier campo desconocido.
>
---

**Firma digital:** https://github.com/ArmandoChimDominguez
