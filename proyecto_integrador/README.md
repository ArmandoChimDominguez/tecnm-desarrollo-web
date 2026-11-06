# Proyecto Integrador · [Nombre del proyecto]

**Curso:** Desarrollo de Aplicaciones Web (DAB-2201)  
**Equipo:** Armando Chim Dominguez y Pedro Pablo Pool Che  
**Dominio elegido:** Reporte → Seguimiento
**Repositorio:** [enlace al repositorio]  
**Docente:** MCC. Iván Márquez Larios (`ijmarquezl`)

> Guía oficial del proyecto: [`GUIA.md`](GUIA.md)

---

## 1. ¿Qué resuelve?

El Sistema de Reporte Ciudadano es una aplicación web que permite a los ciudadanos registrar problemas que afectan a su comunidad, como baches, luminarias dañadas, acumulación de basura y otros problemas relacionados con los servicios públicos.

Su objetivo es facilitar la comunicación entre los ciudadanos y las autoridades municipales, permitiendo registrar, consultar y dar seguimiento a los reportes realizados.

De esta manera, se busca mejorar la organización de los reportes y ayudar a identificar los problemas que necesitan atención.

## 2. Alcance

- Entidad principal: Reporte
- Entidad secundaria: Seguimienti
- Usuario: registro, login y rol `user`/`admin`
- Interfaz: Una SPA con inicio de sesión, lista de reportes, formulario para crear reportes y opción para eliminarlos.
- Funcionalidad principal: Los ciudadanos podrán crear y consultar sus reportes, mientras que el administrador podrá consultar todos los reportes.
- Fuera de alcance: Aplicación móvil, notificaciones en tiempo real, pagos, carga de fotografías y conexión con servicios externos.

## 3. Stack

- Backend: [FastAPI / PHP 8]
- Base de datos: [SQLite / PostgreSQL]
- ORM / migraciones: [SQLAlchemy + Alembic / otro]
- Frontend: HTML, CSS y JavaScript vanilla
- Pruebas: [pytest + httpx / Postman + Newman]

## 4. Cómo ejecutarlo

### Variables de entorno

Copia `.env.example` a `.env` y completa:

```bash
DATABASE_URL=sqlite:///proyecto.db
SECRET_KEY=cambia-esta-clave
ENTORNO=dev
```

### Instalación

```bash
python -m venv .venv
source .venv/bin/activate       # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Base de datos

```bash
alembic upgrade head
```

### Servidor

```bash
uvicorn src.main:app --reload --port 8000
```

Abre `http://127.0.0.1:8000/docs`.

### Pruebas

```bash
pytest -q
```

## 5. Endpoints

| Método | Ruta | Qué hace | Auth |
| :--- | :--- | :--- | :---: |
| POST | `/api/registro` | Crea un usuario | No |
| POST | `/api/token` | Login y JWT | No |
| GET | `/api/[padres]` | Lista los míos; admin ve todos | Sí |
| POST | `/api/[padres]` | Crea un registro principal | Sí |
| GET | `/api/[padres]/{id}/[hijos]` | Lista los hijos del padre | Sí |
| POST | `/api/[padres]/{id}/[hijos]` | Crea un hijo | Sí |
| GET | `/api/[hijos]/{id}` | Consulta un hijo | Sí |
| DELETE | `/api/[hijos]/{id}` | Elimina (dueño o admin) | Sí |
| GET | `/health` | Estado del servicio | No |

> En el Hito 1 se usa un usuario fijo; desde el Hito 2, la columna **Auth** indica token real.

Completa la tabla con tus rutas reales y el ejemplo de solicitud/respuesta.

## 6. Demo

- **URL local:** `http://127.0.0.1:8000`
- **URL pública (opcional):** [enlace]
- **Video de respaldo (opcional):** [enlace]
- **Guion de 5 minutos:** salud → login → crear → listar → eliminar → error controlado → cierre.

## 7. Hitos y etiquetas

| Hito | Fecha | Tag | Estado |
| :--- | :--- | :--- | :--- |
| 1. Contrato y mock | Sesión 10 | `v0.1-hito1` | [ ] |
| 2. Capas y seguridad | Sesión 20 | `v0.2-hito2` | [ ] |
| 3. Persistencia y OWASP | Sesión 30 | `v0.3-hito3` | [ ] |
| 4. Producto | Sesión 40 | `v1.0-final` | [ ] |

## 8. Decisiones y evidencias

- Decisiones por hito: [`docs/decisiones.md`](docs/decisiones.md) (plantilla)
- Propuesta: [`docs/00-propuesta.md`](docs/00-propuesta.md)
- Checkpoints: usa [`docs/checkpoint.md`](docs/checkpoint.md) y guarda cada uno como `docs/checkpoint-0X.md`.
- Evidencias: [`docs/evidencias/`](docs/evidencias/)

## 9. Equipo

| Integrante | Rol principal | Commits | Contacto |
| :--- | :--- | :---: | :--- |
| [Nombre 1] | Backend y datos | [ ] | [ ] |
| [Nombre 2] | Frontend y calidad | [ ] | [ ] |

Acuerdo de equipo: [`docs/equipo.md`](docs/equipo.md).

## 10. IA y autoría

- [ ] El código fue comprendido y probado por ambos.
- [ ] Las consultas a IA están documentadas en `docs/decisiones.md`.
- [ ] No se copió código de otro equipo ni del docente sin citar.
- [ ] No hay secretos ni datos personales en el repositorio.

## 11. Checklist de entrega

- [ ] `pytest` en verde.
- [ ] `/docs` y `/health` responden.
- [ ] README actualizado.
- [ ] Tag del hito creado y empujado.
- [ ] `docs/decisiones.md` con la página del hito.
- [ ] Demo de 5 minutos ensayada.
