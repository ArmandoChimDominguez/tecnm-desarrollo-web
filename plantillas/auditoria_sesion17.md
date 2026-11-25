# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Control de Acceso Basado en Roles (RBAC)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Armando Chim Dominguez
**Matrícula:** 23531079 
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Verificación manual de permisos de usuario (Admin/User)  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> La principal dificultad fue entender cómo manejar el rol dentro del token JWT y cómo interceptar la petición en el endpoint `/api/admin/panel`. Al principio intenté comparar el rol pasándolo como un parámetro de consulta o un header simple, pero el profesor exigía que el rol viajara dentro del token firmado. Tuve que usar `jwt.decode()` para extraer el payload y luego hacer un `if` manual para verificar si el rol era "admin". Además, tuve que asegurarme de que si el token faltaba o era inválido, el servidor devolviera un 401, y si era un usuario normal (rol "user"), devolviera un 403.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Middleware de roles dinámico basado en claims JWT |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
"Actúa como experto en seguridad y FastAPI. Implementa un control de acceso basado en roles (RBAC). Crea un endpoint `/api/admin/panel` que solo sea accesible si el usuario tiene el rol 'admin' (validado a través del JWT). Si el rol es distinto, devuelve un HTTP 403 Forbidden. Si no hay token o es inválido, devuelve HTTP 401. Usa una dependencia de FastAPI (`Depends`) para verificar el rol y mantener el código limpio."
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

app = FastAPI(title="API Científico - Sesión 17")

SECRET_KEY = "mi_clave_secreta"
ALGORITHM = "HS256"
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/token")

usuarios_db = {
    "admin": {"password": "admin123", "rol": "admin"},
    "ana": {"password": "secreta", "rol": "user"}
}

class LoginRequest(BaseModel):
    usuario: str
    password: str

# Dependencia para verificar rol
def verificar_admin(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        if payload.get("rol") != "admin":
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN, 
                detail="Acceso denegado: Se requiere rol de administrador"
            )
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Token inválido")

@app.post("/api/token")
def login(datos: LoginRequest):
    if datos.usuario in usuarios_db and usuarios_db[datos.usuario]["password"] == datos.password:
        rol = usuarios_db[datos.usuario]["rol"]
        expiracion = datetime.datetime.utcnow() + datetime.timedelta(minutes=30)
        token = jwt.encode({"sub": datos.usuario, "rol": rol, "exp": expiracion}, SECRET_KEY, algorithm=ALGORITHM)
        return {"access_token": token, "token_type": "bearer"}
    raise HTTPException(status_code=401, detail="Credenciales incorrectas")

@app.get("/api/perfil")
def perfil(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return {"usuario": payload.get("sub"), "rol": payload.get("rol")}
    except JWTError:
        raise HTTPException(status_code=401, detail="Token inválido")

@app.get("/api/admin/panel")
def panel_admin(payload: dict = Depends(verificar_admin)):
    return {"panel": "admin"}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de acceso denegado por rol insuficiente (HTTP 403)

* ☑ Respuestas semánticas correctas (200 OK, 401 Unauthorized, 403 Forbidden).
* ☑ No existen respuestas HTTP 500 Internal Server Error no capturadas.
* ☑ Encabezados Content-Type: application/json correctamente configurados.
## 🖥️ Salida de la Verificación

```text
==============================================================================
VERIFICACIÓN DEL RESULTADO · 6 casos · app.py
==============================================================================

✅ 01. El administrador obtiene su token
✅ 02. El usuario normal obtiene su token
✅ 03. El admin entra al panel
✅ 04. El usuario normal recibe 403
✅ 05. Sin token recibe 401
✅ 06. El perfil funciona para el usuario normal

------------------------------------------------------------------------------
RESULTADO: 6/6 casos correctos. ¡Resultado logrado!
```

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> La IA omitió inicialmente la validación del header en el endpoint y trató de hacer la comprobación dentro de la misma función, lo que acoplaba la lógica de seguridad con la lógica de negocio. Además, el rol no estaba siendo extraído del JWT, lo que permitía que un usuario normal pudiera falsificar su rol enviándolo en un header.
>
> Implementé una función de dependencia (verificar_admin) usando Depends. Esto permite reutilizar la validación de roles en cualquier endpoint sin repetir código y asegura que el rol se extraiga directamente del JWT firmado, no de un header manipulable. Además, el error 403 se devuelve con el JSON estructurado y el error 401 para tokens inválidos.

---

## Reflexión de la Práctica

### ¿Qué resultado te costó más alcanzar y por qué?

El resultado que más me costó fue lograr que el usuario normal (**ana**) recibiera un **HTTP 403 Forbidden** al intentar acceder al panel de administración. Inicialmente, el servidor devolvía **HTTP 200 OK** porque únicamente verificaba la existencia de un token válido y no comprobaba el rol del usuario. Para solucionarlo, tuve que extraer el campo `rol` desde el payload del JWT y añadir una validación explícita que comprobara si el valor era `"admin"`. También fue necesario diferenciar correctamente entre un error **401 Unauthorized** (cuando no existe token o es inválido) y un error **403 Forbidden** (cuando el token es válido pero el usuario no tiene permisos suficientes).

### ¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?

Decidí utilizar **Depends** para crear una dependencia llamada `verificar_admin`, responsable de validar tanto la autenticación como la autorización del usuario. Aunque el contrato únicamente exigía proteger un endpoint, esta solución permite reutilizar la lógica de validación en futuros endpoints sin duplicar código. Además, opté por almacenar el rol dentro del payload del JWT en lugar de consultarlo en una base de datos en cada petición, lo que simplifica la implementación y mantiene el sistema desacoplado y sin estado (*stateless*).

### ¿Qué error detectó la verificación que tú no habías previsto?

La verificación detectó que las solicitudes sin token debían responder con **HTTP 401 Unauthorized**, mientras que mi implementación inicial devolvía **HTTP 403 Forbidden** en todos los casos. Para corregirlo, diferencié los errores de autenticación de los de autorización. Implementé el manejo adecuado de excepciones para que los tokens ausentes o inválidos produzcan un **401**, reservando el **403** exclusivamente para usuarios autenticados que no poseen el rol requerido para acceder al recurso solicitado.

---

**Firma digital:** https://github.com/ArmandoChimDominguez
**Firma digital:** [Usuario de GitHub]
