# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Tema:** Validación Estricta, Constraints y Params Query/Body  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Nombre Completo]  
**Matrícula:** [No. de Control]  
**Track:** [ Python/FastAPI | PHP 8+ ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Filtro manual de rangos y longitudes de texto  
*Describe qué dificultad encontraste al procesar la petición HTTP o validar campos a mano:*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal (Contrato OpenAPI / DTO)
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | FastAPI 0.110+ (Python 3.10+) o PHP 8.2 con servidor embebido |
| **Rol (R)** | Arquitecto de software senior especialista en APIs RESTful y Seguridad Web |
| **Especificación (E)** | Validaciones Pydantic Field(ge=18, min_length=3) |
| **Output (O)** | Solo código modular, esquemas Pydantic/DTO y códigos HTTP semánticos |

### 2.2 Prompt Estructurado
```text
[Pega aquí el prompt formal que redactaste para la IA]
```

---

## 3. Código Devuelto por el Modelo
```python
# Código devuelto por el LLM
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness HTTP)
*Pruebas ejecutadas:* Pruebas de desbordamiento de cadenas y números negativos

* [ ] Respuestas semánticas correctas (`200 OK`, `201 Created`, `400 Bad Request`, `422 Unprocessable`).
* [ ] No existen respuestas HTTP `500 Internal Server Error` no capturadas.
* [ ] Encabezados `Content-Type: application/json` correctamente configurados.

### Reporte Crítico
*¿Qué código de estado HTTP o falla de seguridad omitió la IA y cómo lo corregiste?*
> 

---
Responde con tus palabras:
¿Qué resultado te costó más alcanzar y por qué?
> El resultado que más me costó fue entender la diferencia entre validar con Field (para el cuerpo de la petición) y validar con Query (para los parámetros de la URL). Al principio intenté poner las restricciones del límite en el modelo de Pydantic, pero el verificador fallaba porque el Query Param se maneja por separado en FastAPI. Comprendí que debo importar Query explícitamente para aplicar restricciones como ge=1, le=100.
¿Qué decisión de diseño tomaste que no estaba escrita en el contrato?
>Usé una lista global (db_libros = []) para simular una base de datos en memoria. Además, decidí usar status.HTTP_201_CREATED explícitamente en el decorador @app.post en lugar de solo retornar un diccionario, para garantizar que el código HTTP sea el correcto semánticamente.
¿Qué error detectó la verificación que tú no habías previsto?
>La verificación detectó que si no ponía el return dentro de la función listar_libros, el servidor fallaba al intentar devolver la lista. Aunque el contrato no lo pedía explícitamente, el hecho de no indentar correctamente el return hacía que la API fallara. Lo corregí asegurándome de que toda la lógica de respuesta estuviera dentro de la función.




**Firma digital:** https://github.com/ArmandoChimDominguez
