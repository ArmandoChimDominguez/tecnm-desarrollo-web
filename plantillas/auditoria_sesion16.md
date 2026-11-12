# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** OAuth2 Password Flow y Header Authorization Bearer  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Extracción manual del encabezado Authorization  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender que OAuth2 no maneja el login como un JSON normal, sino como un formulario (application/x-www-form-urlencoded). Al intentar extraer el token manualmente, me di cuenta de que el header Authorization viene en un formato específico (Bearer <token>) y que debía separar la palabra "Bearer" del token en sí. Además, tuve que interceptar manualmente el error cuando no había token o era inválido, para asegurarme de devolver el código 401 junto con el encabezado WWW-Authenticate: Bearer.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Esquema OAuth2PasswordBearer y endpoint /token |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---
"Actúa como experto en seguridad y FastAPI. Implementa OAuth2 Password Flow. Crea un endpoint /token que reciba un formulario (OAuth2PasswordRequestForm) y devuelva un access_token y token_type: bearer. Protege el endpoint /api/perfil usando OAuth2PasswordBearer. Si el token es inválido o falta, responde con HTTP 401 e incluye el header WWW-Authenticate: Bearer."

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt

app = FastAPI(title="OAuth2 Password Flow - Científico")

SECRET_KEY = "mi_clave_secreta_super_segura"
ALGORITHM = "HS256"

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
usuarios_db = {"ana": "secreta"}

@app.post("/token")
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    if form_data.username in usuarios_db and usuarios_db[form_data.username] == form_data.password:
        token = jwt.encode({"sub": form_data.username}, SECRET_KEY, algorithm=ALGORITHM)
        return {"access_token": token, "token_type": "bearer"}
    raise HTTPException(status_code=401, detail="Credenciales incorrectas")

@app.get("/api/perfil")
def leer_perfil(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        usuario = payload.get("sub")
        if usuario is None:
            raise HTTPException(status_code=401, detail="Token inválido")
        return {"usuario": usuario}
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token inválido",
            headers={"WWW-Authenticate": "Bearer"},
        )
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso no autorizado sin token (HTTP 401)

* ☑ Respuestas semánticas correctas (200 OK, 401 Unauthorized).
* ☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
* ☑ Encabezados Content-Type: application/json correctamente configurados.
* ☑ Se usa el flujo OAuth2 de FastAPI (OAuth2PasswordBearer y OAuth2PasswordRequestForm).
### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente el encabezado WWW-Authenticate: Bearer en la respuesta 401. Aunque el código devolvía el error correcto, sin este encabezado los clientes OAuth2 no saben qué esquema de autenticación deben usar.
>
>  Agregué el parámetro headers={"WWW-Authenticate": "Bearer"} dentro de la HTTPException para cumplir con la RFC 6750. Además, verifiqué que el login se hiciera mediante el formulario (no JSON) usando OAuth2PasswordRequestForm y que la protección del perfil usara OAuth2PasswordBearer, cumpliendo al 100% con el contrato de la sesión.

---

---

### Responde con tus palabras

*¿Qué resultado te costó más alcanzar y por qué?*
> El resultado que más me costó fue hacer que el login funcionara con un **formulario** en lugar de JSON. Estaba acostumbrado a enviar `{"username": "ana", "password": "secreta"}` en el cuerpo, pero OAuth2 exige `application/x-www-form-urlencoded`. Entender que `OAuth2PasswordRequestForm` se encarga de procesar ese formulario y que `OAuth2PasswordBearer` extrae el token del header `Authorization` fue clave para que el test harness pasara los 5 casos.

*¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?*
> Decidí usar un **JWT real firmado con `python-jose`** en lugar de un token simple de texto plano. Aunque el contrato solo pedía "un token", usar JWT garantiza que el token no pueda ser alterado por el cliente y sea seguro en un entorno real. Además, manejé la excepción `JWTError` para devolver un 401 controlado si la firma es inválida.

*¿Qué error detectó la verificación que tú no habías previsto?*
> La verificación detectó que un esquema de autenticación distinto (como `Basic`) también debe devolver 401. Al principio no había previsto que el header `Authorization` podría venir con otra palabra que no fuera "Bearer", y el test harness me recordó que debo validar el esquema estrictamente para cumplir con el estándar OAuth2.

---
**Firma digital:** https://github.com/ArmandoChimDominguez
