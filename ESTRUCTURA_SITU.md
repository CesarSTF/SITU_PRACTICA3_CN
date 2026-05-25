# SITU — Sistema de Transporte Urbano

**Ubicación:** Loja, Ecuador
**Propósito:** Aplicación web para la gestión y simulación del sistema de transporte urbano. Permite administrar pasajeros, buses, tarjetas de acceso, registrar viajes y simular el proceso de acceso/pago.
**Arquitectura:** Monolito Django (Modelo-Vista-Template), listo para despliegue en la nube (Heroku).

---

## 1. Tecnologías empleadas

### Backend

| Tecnología | Versión | Rol |
|---|---|---|
| **Python** | 3.13.3 (`runtime.txt`) | Lenguaje de programación |
| **Django** | 5.2.14 instalado (req: ≥5.2, <6.0) | Framework web MVT |
| **Gunicorn** | 25.3.0 instalado (req: ≥23, <26) | Servidor WSGI para producción |
| **Pillow** | ≥11, <12 | Procesamiento de imágenes (fotos de pasajeros) |
| **dj-database-url** | ≥2.3, <3.0 | Parseo de `DATABASE_URL` para conectar a PostgreSQL |

### Frontend

| Tecnología | Versión | Rol |
|---|---|---|
| **HTML5** | — | Plantillas con Django Template Language (DTL) |
| **CSS3** | — | Estilos personalizados (`login.css`) |
| **Bootstrap** | 4.5.3 (CDN) | Framework CSS responsivo (navbar, tablas, modales, cards) |
| **Animate.css** | 4.1.1 (CDN) | Animaciones CSS (título "SITU" con `animate__pulse`) |
| **jQuery** | 3.5.1 slim (CDN) | Manipulación del DOM y componentes Bootstrap |
| **Popper.js** | 1.16.1 (CDN) | Dependencia de Bootstrap para tooltips/popovers |

### Base de datos

| Tecnología | Uso |
|---|---|
| **SQLite** (`db.sqlite3`) | Motor por defecto en desarrollo local |
| **PostgreSQL** (vía `dj_database_url`) | Motor cuando se configura `DATABASE_URL` en producción |
| **Django ORM** | Capa de abstracción de base de datos |

### Middleware / Almacenamiento

| Tecnología | Función |
|---|---|
| **WhiteNoise** 6.x | Sirve archivos estáticos en producción sin necesidad de nginx |
| **CompressedManifestStaticFilesStorage** | Versiones cacheables de estáticos con hash |

### Despliegue

| Herramienta | Propósito |
|---|---|
| **Heroku** (target) | Plataforma cloud (config vía `Procfile` + `runtime.txt`) |
| **Gunicorn** | Comando web: `gunicorn ProyectoSITU.wsgi:application` |
| **Variables de entorno** | DEBUG, SECRET_KEY, ALLOWED_HOSTS, CSRF_TRUSTED_ORIGINS, DATABASE_URL |

---

## 2. Estructura completa del proyecto

```
SITU-main/
│
├── manage.py
│       Entry point CLI de Django. Ejecuta comandos de administración
│       (runserver, migrate, collectstatic, etc.). Define la variable
│       de entorno DJANGO_SETTINGS_MODULE = 'ProyectoSITU.settings'.
│
├── requirements.txt
│       Dependencias Python para pip:
│       · Django>=5.2,<6.0
│       · Pillow>=11.0,<12.0
│       · dj-database-url>=2.3,<3.0
│       · gunicorn>=23.0,<26.0
│       · whitenoise>=6.8,<7.0
│
├── runtime.txt
│       Define la versión de Python para Heroku: python-3.13.3
│
├── Procfile
│       Comando web para plataformas con buildpacks (Heroku):
│       web: gunicorn ProyectoSITU.wsgi:application
│
├── .env.example
│       Plantilla de variables de entorno:
│       · DEBUG=False
│       · SECRET_KEY=change-me
│       · ALLOWED_HOSTS=localhost,127.0.0.1
│       · CSRF_TRUSTED_ORIGINS=
│       · DATABASE_URL=
│
├── .gitignore
│       Ignora: __pycache__/, *.py[cod], .venv/, venv/, db.sqlite3,
│       img/, staticfiles/, media/, .env, .DS_Store, .vscode/
│
├── db.sqlite3
│       Base de datos SQLite (usada en desarrollo cuando DATABASE_URL
│       está vacío).
│
├── README.md
│       Documentación básica del proyecto: estructura, variables de
│       entorno y comandos útiles.
│
├── ProyectoSITU/                     ★ Configuración del proyecto Django
│   ├── __init__.py
│   │       Marca el directorio como paquete Python.
│   │
│   ├── settings.py                   Configuración principal (156 líneas)
│   │   ├── Funciones helper:
│   │   │   · env_bool(name, default)  → parsea booleanos laxos
│   │   │   · env_list(name, default)  → parsea listas separadas por coma
│   │   ├── SECRET_KEY: desde variable de entorno o fallback local
│   │   ├── DEBUG: desde variable de entorno (por defecto True)
│   │   ├── ALLOWED_HOSTS: desde variable de entorno
│   │   ├── CSRF_TRUSTED_ORIGINS: desde variable de entorno
│   │   ├── INSTALLED_APPS (7 apps):
│   │   │   · django.contrib.admin
│   │   │   · django.contrib.auth
│   │   │   · django.contrib.contenttypes
│   │   │   · django.contrib.sessions
│   │   │   · django.contrib.messages
│   │   │   · django.contrib.staticfiles
│   │   │   · appSITUweb
│   │   ├── MIDDLEWARE (8 middlewares, en orden):
│   │   │   · SecurityMiddleware
│   │   │   · WhiteNoiseMiddleware        ← Sirve estáticos en producción
│   │   │   · SessionMiddleware
│   │   │   · CommonMiddleware
│   │   │   · CsrfViewMiddleware
│   │   │   · AuthenticationMiddleware
│   │   │   · MessageMiddleware
│   │   │   · XFrameOptionsMiddleware
│   │   ├── ROOT_URLCONF: ProyectoSITU.urls
│   │   ├── TEMPLATES:
│   │   │   · BACKEND: django.template.backends.django.DjangoTemplates
│   │   │   · DIRS: [BASE_DIR / "templates"]
│   │   │   · APP_DIRS: True
│   │   ├── DATABASES:
│   │   │   · Default: SQLite (BASE_DIR / 'db.sqlite3')
│   │   │   · Si DATABASE_URL está definida → la parsea con dj_database_url
│   │   ├── Password validators estándar de Django
│   │   ├── INTERNATIONALIZATION:
│   │   │   · LANGUAGE_CODE = 'en-us'
│   │   │   · TIME_ZONE = 'UTC'
│   │   ├── STATICFILES_DIRS: [BASE_DIR / "templates/static"]
│   │   ├── STATIC_ROOT: BASE_DIR / "staticfiles"
│   │   ├── STORAGES:
│   │   │   · default: FileSystemStorage
│   │   │   · staticfiles: whitenoise.storage.CompressedManifestStaticFilesStorage
│   │   ├── MEDIA_URL = "/media/"
│   │   ├── MEDIA_ROOT = BASE_DIR
│   │   └── DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
│   │
│   ├── urls.py                       Enrutador principal (34 líneas)
│   │   ├── ''            → home_view         → index.html
│   │   ├── 'admin/'     → admin.site.urls   → panel admin
│   │   ├── 'pasajeros/' → pasajeros         → pasajeros.html (GET lista, POST crear)
│   │   ├── 'pasajerosEdit/<id>'  → pasajerosEdit   → pasajerosEdit.html
│   │   ├── 'pasajerosDelete/<id>' → pasajerosDelete → redirect a pasajeros
│   │   ├── 'pasajerosCreate/'    → pasajerosCreate → agregar.html
│   │   └── if DEBUG: + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
│   │
│   ├── wsgi.py
│   │       WSGI config. Expone application para Gunicorn.
│   │       DJANGO_SETTINGS_MODULE = 'ProyectoSITU.settings'
│   │
│   └── asgi.py
│           ASGI config. Expone application para servidores ASGI.
│           No utilizado actualmente (no hay WebSockets ni channels).
│
├── appSITUweb/                       ★ Aplicación Django principal
│   ├── __init__.py
│   │       Marca el directorio como paquete Python.
│   │
│   ├── apps.py
│   │       class AppsituwebConfig(AppConfig):
│   │           default_auto_field = 'django.db.models.BigAutoField'
│   │           name = 'appSITUweb'
│   │
│   ├── models.py                     ★ 5 modelos (48 líneas)
│   │   ├── Pasajero
│   │   │   · cedula: CharField(10), NOT NULL
│   │   │   · nombre: CharField(10), NOT NULL
│   │   │   · imagen: ImageField(upload_to='img/%Y/%m/%d/')
│   │   │   · apellido: CharField(30)
│   │   │   · email: EmailField()
│   │   │   · __str__ → cedula
│   │   │
│   │   ├── Tarjeta
│   │   │   · codigo: CharField(6), NOT NULL
│   │   │   · monto: CharField(3), NOT NULL
│   │   │   · idPasajero: ForeignKey(Pasajero, CASCADE)
│   │   │   · __str__ → "Tarjeta: {codigo} | Pasajero: {idPasajero} | Monto: {monto}"
│   │   │
│   │   ├── Bus
│   │   │   · placa: CharField(7), NOT NULL
│   │   │   · cooperativa: CharField(10), NOT NULL
│   │   │   · numero: DecimalField(3,0)
│   │   │   · idTarjeta: ManyToManyField(Pasajero, through='Viaje')
│   │   │   · __str__ → placa
│   │   │
│   │   ├── Viaje (tabla intermedia M2M)
│   │   │   · pasajero: ForeignKey(Pasajero, CASCADE)
│   │   │   · bus: ForeignKey(Bus, CASCADE)
│   │   │   · costo: DecimalField(4,2)
│   │   │   · cantidad: IntegerField()
│   │   │   · fecha_viaje: DateTimeField(auto_now_add=True)
│   │   │   · efectivo: BooleanField(default=True)
│   │   │   · tipo: CharField(20), choices=[('comodo','Comodo'), ('incomodo','Incomodo')]
│   │   │   · __str__ → "Pasajero: {cedula} | Nombre: {nombre} | Precio: {costo} | Bus: {placa} | No: {numero}"
│   │   │
│   │   └── SimularAccesoPago
│   │       · numero: CharField(7), NOT NULL
│   │       · fecha_viaje: DateTimeField(auto_now_add=True)
│   │       · viaje: ForeignKey(Viaje, CASCADE)
│   │       · tarjeta: ForeignKey(Tarjeta, CASCADE)
│   │       · __str__ → "Pasajero: {viaje.pasajero.nombre}"
│   │
│   ├── forms.py
│   │       class PasajeroFormulario(ModelForm):
│   │           model = Pasajero
│   │           fields = ["cedula", "nombre", "apellido", "email", "imagen"]
│   │
│   ├── views.py                      ★ 5 vistas función (51 líneas)
│   │   ├── home_view(request)
│   │   │       Renderiza index.html sin contexto adicional.
│   │   │
│   │   ├── pasajeros(request)
│   │   │       GET:  instancia PasajeroFormulario vacío, consulta todos los
│   │   │             pasajeros y renderiza pasajeros.html.
│   │   │       POST: procesa PasajeroFormulario con request.POST + request.FILES,
│   │   │             guarda si válido y renderiza la misma página.
│   │   │
│   │   ├── pasajerosEdit(request, id)
│   │   │       GET:  busca Pasajero por id (get_object_or_404), precarga el
│   │   │             formulario con instance y renderiza pasajerosEdit.html.
│   │   │       POST: procesa formulario con los datos recibidos, guarda y
│   │   │             redirige a "pasajeros".
│   │   │
│   │   ├── pasajerosDelete(request, id)
│   │   │       Busca, elimina y redirige a "pasajeros".
│   │   │       No tiene confirmación propia (el confirm JS está en el template).
│   │   │
│   │   └── pasajerosCreate(request)
│   │           GET:  renderiza agregar.html con formulario vacío.
│   │           POST: procesa formulario, guarda y redirige a "pasajeros".
│   │
│   ├── admin.py
│   │       Registra los 5 modelos en el admin de Django con list_display:
│   │       · AdminPasajero: __str__, nombre, apellido, cedula, email
│   │       · AdminTarjeta: __str__, codigo, monto
│   │       · AdminBus: __str__, placa, numero
│   │       · AdminViaje: __str__, bus, costo, fecha_viaje, efectivo
│   │       · AdminSimularAccesoPago: __str__, numero, fecha_viaje
│   │
│   ├── tests.py
│   │       Test stub. Solo importa django.test.TestCase.
│   │
│   └── migrations/
│       ├── __init__.py
│       │       Marca el directorio como paquete Python.
│       │
│       └── 0001_initial.py
│               Migración inicial generada con Django 4.0.6 (2022-07-13).
│               Crea las 5 tablas y la relación M2M entre Bus y Pasajero
│               a través de Viaje.
│
├── templates/                        ★ Plantillas HTML (Django Template Language)
│   ├── base.html                     Layout base (52 líneas)
│   │   ├── DOCTYPE html, lang="es"
│   │   ├── Meta: charset UTF-8, viewport
│   │   ├── Título: bloque title
│   │   ├── CDN CSS:
│   │   │   · Bootstrap 4.5.3 (cdn.jsdelivr.net)
│   │   │   · Animate.css 4.1.1 (cdnjs.cloudflare.com)
│   │   │   · login.css (estático local)
│   │   ├── Navbar dark:
│   │   │   · Brand: "SISTEMA DE TRANSPORTE URBANO SITU LOJA"
│   │   │   · Links: Inicio (home), Pasajeros (pasajeros)
│   │   │   · Toggler responsive con collapse
│   │   ├── Container: bloque body
│   │   └── CDN JS (al final del body):
│   │       · jQuery 3.5.1 slim
│   │       · Popper.js 1.16.1
│   │       · Bootstrap 4.5.3 JS
│   │
│   ├── index.html                    Página de inicio
│   │   ├── Extiende base.html
│   │   ├── title = "Inicio"
│   │   ├── Título "SITU" animado (animate__pulse animate__infinite)
│   │   │   con fuente "Comic Sans MS", tamaño 50px
│   │   ├── Subtítulo "LOGO"
│   │   ├── Imagen: logo.png (350x220)
│   │   └── Mensaje: "Bienvenido al sistema de prueba de SITU"
│   │
│   ├── pasajeros.html                Listado de pasajeros (85 líneas)
│   │   ├── Extiende base.html
│   │   ├── title = "PASAJE"
│   │   ├── Título "SITU" animado
│   │   ├── Modal Bootstrap (#modalAgregarPasajero):
│   │   │   · Form con method=POST, enctype=multipart/form-data
│   │   │   · action = pasajeros (misma vista)
│   │   │   · Campos: {{ form.as_p }} + csrf_token
│   │   │   · Botones: Cancelar, Guardar
│   │   ├── Card con encabezado "Listado de Pasajeros"
│   │   │   · Botón "➕ Agregar" → pasajerosCreate
│   │   ├── Tabla responsiva con columnas:
│   │   │   · # (forloop.counter)
│   │   │   · Cédula, Nombre, Apellido, Teléfono
│   │   │   · Imagen (muestra la foto del pasajero o logo por defecto)
│   │   │   · Correo
│   │   │   · Opciones: Editar (btn-info), Eliminar (btn-danger + confirm)
│   │   │   · Itera sobre {{ pasajeros }}
│   │   └── Nota: la tabla muestra un campo "telefono" que no existe en el
│   │       modelo; aparece vacío.
│   │
│   ├── pasajerosEdit.html            Editar pasajero (28 líneas)
│   │   ├── Extiende base.html
│   │   ├── title = "Pasaje"
│   │   ├── Título "SITU" animado
│   │   ├── Card con form:
│   │   │   · method=POST, enctype=multipart/form-data
│   │   │   · action="" (misma URL)
│   │   │   · {{ form }} (sin .as_p)
│   │   │   · csrf_token
│   │   │   · Botones: Cancelar (→ pasajeros), Editar (submit)
│   │
│   ├── agregar.html                  Crear pasajero (28 líneas)
│   │   ├── Extiende base.html
│   │   ├── title = "Agregar Pasajero"
│   │   ├── Título "SITU" animado
│   │   ├── Card con form:
│   │   │   · method=POST, enctype=multipart/form-data
│   │   │   · action="" (misma URL)
│   │   │   · {{ form.as_p }}
│   │   │   · csrf_token
│   │   │   · Botones: Cancelar (→ pasajeros), Guardar (submit, btn-success)
│   │
│   └── static/
│       ├── login.css                 Estilos personalizados (211 líneas)
│       │   ├── Reset universal (margin:0, padding:0)
│       │   ├── Body con gradient diagonal: #2ebf91 → #8360c3
│       │   ├── .contenido: float right, fondo azul
│       │   ├── .contenedor: width 90%, max-width 1000px, centrado
│       │   ├── .contenedor article: texto blanco, padding 250px, fuente Montserrat
│       │   ├── .btn-abrir-popup: gradient verde, border-radius, hover rainbow
│       │   ├── Overlay/popup:
│       │   │   · overlay: fondo azul semitransparente, flex centrado, hidden
│       │   │   · popup: gradient verde, border-radius 20px, animación scale
│       │   │   · Animaciones keyframes:
│       │   │       · entradaTitulo: desde -25px Y
│       │   │       · entradaSubtitulo: desde +25px Y
│       │   │       · entradaInputs: fade in
│       │   ├── Inputs: border-radius 15px, centrados, ancho 75%
│       │   └── Botón submit: gradient azul, hover rosado
│       │
│       └── logo.png                  Logo del SITU (formato PNG)
│
└── img/                              ★ Archivos multimedia (fotos subidas)
    └── 2026/05/12/
            Almacenamiento organizado automáticamente por fecha
            (upload_to='img/%Y/%m/%d/').
            · N_29.jpg
            · N_31.jpg
            · N_40.jpg
            · N_53.jpg
            · N_58.jpg
```

---

## 3. Modelos de datos — Diagrama relacional

```
┌──────────────────────┐       ┌──────────────────────┐
│      Pasajero        │       │       Tarjeta        │
├──────────────────────┤       ├──────────────────────┤
│ PK id: BigAutoField  │       │ PK id: BigAutoField  │
│    cedula: Char(10)  │◄──┐   │    codigo: Char(6)   │
│    nombre: Char(10)  │   └───┤ FK idPasajero        │
│    apellido: Char(30)│       │    monto: Char(3)    │
│    email: EmailField │       └──────────┬───────────┘
│    imagen: ImageField│                  │
└──────────┬───────────┘                  │
           │                             │
           │  M:N (through Viaje)        │
           │  ┌──────────────┐           │
           │  │    Viaje     │           │
           │  ├──────────────┤           │
           ├──┤ FK pasajero  │           │
           │  │ FK bus       │           │
           │  │ costo: Dec(4)│           │
           │  │ cantidad: Int│           │
           │  │ fecha_viaje  │           │
           │  │ efectivo: Bool           │
           │  │ tipo: Choice │           │
           │  └──────┬───────┘           │
           │         │                  │
┌──────────┴───┐    │   ┌───────────────┴──┐
│     Bus      │    │   │ SimularAccesoPago│
├──────────────┤    │   ├──────────────────┤
│ PK id        │    │   │ PK id            │
│ placa: Char(7│    │   │ numero: Char(7)  │
│ cooperativa  │    │   │ fecha_viaje      │
│ numero: Dec  │    └───┤ FK viaje         │
└──────────────┘        │ FK tarjeta       │
                        └──────────────────┘
```

### Relaciones clave

| Relación | Tipo | Tabla intermedia | On Delete |
|---|---|---|---|
| Pasajero → Tarjeta | 1:N | — | CASCADE |
| Pasajero → Bus | M:N | Viaje | CASCADE |
| Pasajero → Viaje | 1:N | — | CASCADE |
| Bus → Viaje | 1:N | — | CASCADE |
| Viaje → SimularAccesoPago | 1:N | — | CASCADE |
| Tarjeta → SimularAccesoPago | 1:N | — | CASCADE |

---

## 4. Vistas y rutas — Mapeo completo

| Ruta URL | Nombre | Vista | Métodos HTTP | Template | Acción |
|---|---|---|---|---|---|
| `/` | `home` | `home_view` | GET | `index.html` | Muestra página de inicio |
| `/admin/` | — | `admin.site.urls` | GET, POST | Admin Django | Panel de administración |
| `/pasajeros/` | `pasajeros` | `pasajeros` | GET | `pasajeros.html` | Lista todos los pasajeros |
| `/pasajeros/` | `pasajeros` | `pasajeros` | POST | `pasajeros.html` | Crea pasajero (form modal) |
| `/pasajerosCreate/` | `pasajerosCreate` | `pasajerosCreate` | GET | `agregar.html` | Formulario vacío |
| `/pasajerosCreate/` | `pasajerosCreate` | `pasajerosCreate` | POST | redirect → `/pasajeros/` | Guarda nuevo pasajero |
| `/pasajerosEdit/<id>` | `pasajerosEdit` | `pasajerosEdit` | GET | `pasajerosEdit.html` | Form precargado |
| `/pasajerosEdit/<id>` | `pasajerosEdit` | `pasajerosEdit` | POST | redirect → `/pasajeros/` | Actualiza pasajero |
| `/pasajerosDelete/<id>` | `pasajerosDelete` | `pasajerosDelete` | GET | redirect → `/pasajeros/` | Elimina pasajero |

### Características de las vistas

- **Todas son Function-Based Views (FBV)** — no se usan Class-Based Views.
- **No hay autenticación** — no hay decoradores `@login_required` ni sistema de permisos.
- **No hay paginación** — la vista `pasajeros` obtiene todos los registros con `Pasajero.objects.all()`.
- **Subida de imágenes** — las vistas que reciben archivos usan `request.FILES` y los formularios usan `enctype="multipart/form-data"`.
- **Eliminación con confirmación** — el template `pasajeros.html` usa `onclick="return confirm(...)"` del lado del cliente; no hay vista intermedia de confirmación.

---

## 5. Templates y flujo de navegación

```
base.html  (layout con navbar, CDNs, bloques)
│
├── index.html  (Inicio)
│   ├── Título animado "SITU"
│   ├── Logo
│   └── Mensaje de bienvenida
│
├── pasajeros.html  (Listado)
│   ├── Modal "Agregar Pasajero" (form incluido en la misma página)
│   ├── Tabla con todos los pasajeros
│   │   └── Por cada fila: [Editar] → pasajerosEdit/<id>
│   │                         [Eliminar] → pasajerosDelete/<id>
│   └── Botón "➕ Agregar" → pasajerosCreate/
│
├── pasajerosEdit.html  (Edición)
│   ├── Formulario precargado con datos del pasajero
│   └── [Cancelar] → pasajeros/  |  [Editar] → submit
│
└── agregar.html  (Creación)
    ├── Formulario vacío
    └── [Cancelar] → pasajeros/  |  [Guardar] → submit
```

### Dependencias CDN en base.html

```
Bootstrap 4.5.3 CSS  ─────────────────┐
Animate.css 4.1.1    ─────────────────┤
login.css (local)    ─────────────────┤
                                      ├──  <head>
jQuery 3.5.1 slim    ───────────────┐ │
Popper.js 1.16.1     ───────────────┤ │
Bootstrap 4.5.3 JS   ───────────────┤ ├──  </body>
```

---

## 6. Configuración de entorno y despliegue

### Variables de entorno

| Variable | Ejemplo | Efecto si vacía | Helper |
|---|---|---|---|
| `DEBUG` | `False` | True por defecto | `env_bool()` |
| `SECRET_KEY` | `abc123...` | Usa fallback inseguro (solo local) | `os.environ.get()` |
| `ALLOWED_HOSTS` | `midominio.com` | localhost,127.0.0.1 | `env_list()` |
| `CSRF_TRUSTED_ORIGINS` | `https://midominio.com` | Lista vacía | `env_list()` |
| `DATABASE_URL` | `postgres://user:pass@host/db` | Usa SQLite | `dj_database_url.parse()` |

### Pipeline de middlewares (orden de ejecución)

```
Request
  │
  ▼
SecurityMiddleware            (seguridad SSL/HSTS)
WhiteNoiseMiddleware          (sirve archivos estáticos)
SessionMiddleware             (gestión de sesiones)
CommonMiddleware              (redirecciones, APPEND_SLASH)
CsrfViewMiddleware            (protección CSRF)
AuthenticationMiddleware      (asocia usuario con request)
MessageMiddleware             (mensajes flash)
XFrameOptionsMiddleware       (protección clickjacking)
  │
  ▼
URL resolver → View
```

### Flujo de despliegue (Heroku)

```bash
# 1. Instalar dependencias
pip install -r requirements.txt

# 2. Ejecutar migraciones
python manage.py migrate

# 3. Recopilar archivos estáticos
python manage.py collectstatic --noinput

# 4. Iniciar servidor (vía Procfile)
gunicorn ProyectoSITU.wsgi:application
```

WhiteNoise intercepta las peticiones de archivos estáticos en producción, eliminando la necesidad de un servidor web intermedio (nginx, Apache) o servicios externos (S3, CloudFront).

### Almacenamiento

| Tipo | Origen | Destino (producción) | Backend |
|---|---|---|---|
| Estáticos (CSS, JS, imágenes fijas) | `templates/static/` | `staticfiles/` | whitenoise.storage.CompressedManifestStaticFilesStorage |
| Media (fotos de pasajeros) | Subidas por usuarios | `img/%Y/%m/%d/` | django.core.files.storage.FileSystemStorage |

---

## 7. Observaciones y detalle técnico

### Inconsistencias detectadas

| Archivo | Detalle |
|---|---|
| `pasajeros.html:65` | La tabla referencia `{{ e.telefono }}` (teléfono) pero el modelo `Pasajero` no tiene ese campo. Se renderiza vacío. |
| `pasajeros.html:74` | La tabla referencia `{{ e.correo }}` pero el campo en el modelo es `email`. Se renderiza vacío. |
| `settings.py:4` | Comentario generado con Django 4.0.6, pero la versión instalada es 5.2.14. |
| `runtime.txt` | Declara `python-3.13.3`, pero el `.venv` fue creado con Python 3.12. |

### Licencias y atribuciones CDN

Bootstrap, jQuery, Popper.js y Animate.css se cargan desde CDN externos. El proyecto no incluye distribuciones locales de estas bibliotecas.

---

*Documento generado a partir del análisis completo del código fuente del proyecto SITU-main.*
