# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Hashing Seguro de Contraseñas (Bcrypt / Argon2)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cifrado manual con Salting y Factor de Costo  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender que la contraseña nunca debe guardarse ni devolverse. Al principio quería devolver el usuario con su contraseña para "confirmar", pero el contrato prohíbe eso. Tuve que usar `passlib` para hashear la contraseña con `bcrypt.hash()` y guardar solo ese hash. Además, en el login no se puede deshashear, sino usar `bcrypt.verify()` para comparar la contraseña con el hash guardado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Hashing con passlib / password_hash() de PHP |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como experto en seguridad web y FastAPI. Necesito un sistema de registro y login. Usa `passlib` con `bcrypt` para hashear contraseñas. En el registro, guarda solo el hash en un diccionario y devuelve únicamente el usuario. En el login, verifica la contraseña con `pwd_context.verify()` y devuelve 401 si es incorrecta. Nunca devuelvas el hash ni la contraseña en la respuesta."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel, Field
from passlib.context import CryptContext

app = FastAPI(title="API Científico - Sesión 14")

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class RegistroRequest(BaseModel):
    usuario: str = Field(..., min_length=1)
    password: str = Field(..., min_length=6)

class LoginRequest(BaseModel):
    usuario: str
    password: str

usuarios_db = {}

@app.post("/api/registro", status_code=status.HTTP_201_CREATED)
def registro(datos: RegistroRequest):
    if datos.usuario in usuarios_db:
        raise HTTPException(status_code=400, detail="El usuario ya existe")
    
    hash_password = pwd_context.hash(datos.password)
    usuarios_db[datos.usuario] = hash_password
    return {"usuario": datos.usuario}

@app.post("/api/login")
def login(datos: LoginRequest):
    hash_guardado = usuarios_db.get(datos.usuario)
    
    if not hash_guardado or not pwd_context.verify(datos.password, hash_guardado):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Credenciales incorrectas")
    
    return {"usuario": datos.usuario, "autenticado": True}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas contra ataques de Rainbow Tables y Timing

* ☑ Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* ☑ No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* ☑ Encabezados `Content-Type: application/json` correctamente configurados.
---

---
🖥️ Salida de la Verificación
==============================================================================
VERIFICACIÓN DEL RESULTADO · 3 casos · app_ia.py
==============================================================================

* ✅ 01. El registro responde sin filtrar la contraseña
* ✅ 02. El login correcto autentica
* ✅ 03. La contraseña incorrecta se rechaza

------------------------------------------------------------------------------
RESULTADO: 3/3 casos correctos. ¡Resultado logrado!

---
---

* ✅ Se usa un algoritmo de hashing para contraseñas

------------------------------------------------------------------------------
RESULTADO: 1/1 comprobaciones correctas. ¡Resultado logrado!

---
Reporte Crítico
¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?

La IA omitió inicialmente la validación de usuario repetido en el registro. Si un atacante intentaba registrar el mismo usuario, el servidor sobrescribía el hash existente, lo que permitía cambiar la contraseña de otro usuario.

Corrección aplicada: Agregué la validación if datos.usuario in usuarios_db en el registro para devolver un HTTP 400 si el usuario ya existe. También me aseguré de que la respuesta del registro solo incluya el campo usuario, sin exponer el hash ni la contraseña.

Firma digital: [ArmandoChimDominguez]
