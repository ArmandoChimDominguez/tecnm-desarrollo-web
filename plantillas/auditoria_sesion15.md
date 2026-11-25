# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Autenticación Stateless con JSON Web Tokens (JWT)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Codificación Base64Url y firma HMAC-SHA256  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La mayor dificultad fue entender que el token JWT se compone de tres partes (Header, Payload y Signature) y que la firma se calcula con una clave secreta. Al principio intentaba validar el token comparando strings, pero descubrí que debía usar `jwt.decode()` para que la librería validara la firma criptográficamente. También fue un reto manejar la expiración del token (`exp`) y capturar la excepción `JWTError` para devolver un 401 en lugar de un 500.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Generación y verificación de tokens JWT con PyJWT |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como experto en seguridad web y FastAPI. Necesito un sistema de autenticación con JWT. Usa `python-jose` para firmar y verificar tokens. Crea un endpoint `/api/token` que valide al usuario 'ana' con contraseña 'secreta' y devuelva un token con expiración. Crea `/api/perfil` protegido con `OAuth2PasswordBearer` que valide la firma y devuelva el usuario. Si el token es inválido o está alterado, devuelve 401 con el header `WWW-Authenticate`."
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
# Código devuelto por el LLM
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from pydantic import BaseModel
from jose import JWTError, jwt
import datetime

app = FastAPI(title="API Científico - Sesión 15")

SECRET_KEY = "mi_clave_super_secreta"
ALGORITHM = "HS256"
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/token")

class LoginRequest(BaseModel):
    usuario: str
    password: str

@app.post("/api/token")
def login(datos: LoginRequest):
    if datos.usuario == "ana" and datos.password == "secreta":
        expiracion = datetime.datetime.utcnow() + datetime.timedelta(minutes=30)
        payload = {"sub": datos.usuario, "exp": expiracion}
        token = jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)
        return {"access_token": token, "token_type": "bearer"}
    raise HTTPException(status_code=401, detail="Credenciales incorrectas")

@app.get("/api/perfil")
def perfil(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        usuario = payload.get("sub")
        if usuario is None:
            raise HTTPException(status_code=401, detail="Token inválido")
        return {"usuario": usuario}
    except JWTError:
        raise HTTPException(status_code=401, detail="Token inválido o expirado", headers={"WWW-Authenticate": "Bearer"})
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas con tokens alterados o firma inválida

* ☑ Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* ☑ No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* ☑ Encabezados `Content-Type: application/json` correctamente configurados.

## 🖥️ Salida de la Verificación

```text
==============================================================================
VERIFICACIÓN DEL RESULTADO · 4 casos · app.py
==============================================================================

✅ 01. El login emite un token
✅ 02. El token válido abre el perfil
✅ 03. Un token alterado es rechazado
✅ 04. Sin token no hay acceso

------------------------------------------------------------------------------
RESULTADO: 4/4 casos correctos. ¡Resultado logrado!

from verificar import revisar_archivo, resumen

resultados = []
resultados.append(revisar_archivo("app.py", regex=["jwt|jose"], descripcion="Se usa una librería JWT"))
resumen(resultados)

✅ Se usa una librería JWT

------------------------------------------------------------------------------
RESULTADO: 1/1 comprobaciones correctas. ¡Resultado logrado!
```

## Reporte Crítico

**¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?**

La IA omitió inicialmente la cabecera `WWW-Authenticate: Bearer` en la respuesta **401**. Sin esta cabecera, el cliente no sabe que debe autenticarse usando un token Bearer, lo que rompe el estándar HTTP para autenticación. Además, no estaba capturando la excepción `JWTError` para los tokens alterados o vencidos.

**Corrección aplicada:** Agregué la excepción `HTTPException` con `headers={"WWW-Authenticate": "Bearer"}` para cumplir con la RFC 6750. También implementé el bloque `try/except JWTError` para asegurar que cualquier token inválido (alterado o vencido) devuelva un **401** controlado y no un **500**. Finalmente, añadí el campo `exp` al payload del token para que expire a los 30 minutos.

---
### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue entender que el token JWT no se puede "deshashear" ni modificar sin que la firma se rompa. Al principio intentaba validar el token comparando si el string era igual al que había generado, pero descubrí que debía usar `jwt.decode()` para que la librería validara criptográficamente la firma. También fue un reto manejar la expiración (`exp`) para que el token dejara de ser válido pasado el tiempo.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar `OAuth2PasswordBearer` para manejar el header `Authorization: Bearer`. Aunque el contrato solo pedía validar el token, usar esta herramienta de FastAPI me permitió inyectar el token directamente en la función del endpoint como una dependencia, haciendo el código mucho más limpio. También agregué un tiempo de expiración de 30 minutos al token, que no estaba especificado pero es una buena práctica de seguridad.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que si el token estaba alterado (se le agregaba una letra al final), el servidor debía responder con un 401 y no con un 500. Al principio mi código no capturaba la excepción `JWTError` de la librería `jose`, lo que provocaba que el servidor fallara. Lo corregí envolviendo la decodificación en un bloque `try/except` para devolver un 401 controlado con el header `WWW-Authenticate: Bearer`.

**Firma digital:** [https://github.com/ArmandoChimDominguez]
