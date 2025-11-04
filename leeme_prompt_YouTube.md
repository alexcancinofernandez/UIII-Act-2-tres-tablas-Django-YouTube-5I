
Perfecto — te doy TODO paso a paso (comandos, archivos y código listo para pegar) para crear el proyecto **UIII_YouTube_0777** con Django + VS Code y dejar funcionando el CRUD de **Canal**. He incluido instrucciones para Windows y Linux/Mac, cómo crear/activar `.venv`, seleccionar intérprete en VS Code, creación de proyecto sin duplicar carpeta, migraciones y ejecución del servidor en **puerto 8157** (como pediste) y también la forma de ejecutarlo en **8517** (tu paso final). No uso `forms.py` (tal como solicitaste).

---

# 1. Estructura inicial (qué crear)

`UIII_YouTube_0777/`

```
UIII_YouTube_0777/
├─ .venv/            # entorno virtual (oculto)
├─ backend_YouTube/  # proyecto Django
│  ├─ backend_YouTube/  # settings, urls, wsgi, asgi
│  └─ manage.py
└─ app_YouTube/
   ├─ migrations/
   ├─ templates/
   │  ├─ base.html
   │  ├─ header.html
   │  ├─ navbar.html
   │  ├─ footer.html
   │  ├─ inicio.html
   │  └─ canal/
   │     ├─ agregar_canal.html
   │     ├─ ver_canal.html
   │     ├─ actualizar_canal.html
   │     └─ borrar_canal.html
   ├─ admin.py
   ├─ apps.py
   ├─ models.py
   ├─ views.py
   └─ urls.py
```

---

# 2. Comandos paso-a-paso

> Asume que estás en el directorio donde quieres crear la carpeta del proyecto (por ejemplo `C:\Users\Alex\Proyectos` o `~/Proyectos`).

## 1) Crear carpeta del proyecto `UIII_YouTube_0777`

```bash
# Windows PowerShell / Linux / macOS
mkdir UIII_YouTube_0777
cd UIII_YouTube_0777
```

## 2) Abrir VS Code en la carpeta

```bash
# Desde terminal (si tienes `code` en PATH)
code .
# O abre VS Code y usa "File → Open Folder..." → selecciona UIII_YouTube_0777
```

## 3) Abrir terminal en VS Code

* Menú: `Terminal → New Terminal`
  La terminal se abrirá en la raíz del proyecto `UIII_YouTube_0777`.

## 4) Crear carpeta entorno virtual `.venv` desde terminal

```bash
# Windows (PowerShell)
python -m venv .venv

# Linux / macOS
python3 -m venv .venv
```

Esto crea la carpeta `.venv` dentro de `UIII_YouTube_0777`.

## 5) Activar el entorno virtual

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# Windows CMD
.venv\Scripts\activate.bat

# Linux / macOS (bash/zsh)
source .venv/bin/activate
```

Después verás `(.venv)` al inicio de tu prompt.

## 6) Activar intérprete de Python en VS Code

* Abre Command Palette (`Ctrl+Shift+P`) → escribe `Python: Select Interpreter` → elige el Python dentro de `UIII_YouTube_0777/.venv/...` (aparecerá como `.venv` o la ruta completa).
  Esto hace que VS Code use el entorno virtual como intérprete.

## 7) Instalar Django

```bash
pip install django
# Si piensas usar PostgreSQL más adelante (opcional):
# pip install psycopg2-binary
```

## 8) Crear proyecto `backend_YouTube` sin duplicar carpeta

Importante: para evitar una carpeta extra dentro de la carpeta actual, usa el punto `.` al final *si ya estás en la carpeta* donde quieres que esté `manage.py`.

```bash
# Asegúrate de estar en UIII_YouTube_0777/
django-admin startproject backend_YouTube .
# Esto crea backend_YouTube/ (la app del proyecto) y manage.py en la carpeta actual
```

> Si accidentalmente ejecutaste `django-admin startproject backend_YouTube` sin `.` se creará `UIII_YouTube_0777/backend_YouTube/backend_YouTube` (duplicada). Si eso pasara, muévete y reorganiza; pero mejor usar el comando con `.` como arriba.

## 9) Ejecutar servidor en el puerto **8157**

```bash
python manage.py runserver 0.0.0.0:8157
# o localmente solamente:
python manage.py runserver 127.0.0.1:8157
```

## 10) Copiar y pegar el link en el navegador

* Si usaste `127.0.0.1:8157` → abre `http://127.0.0.1:8157/`
* Si usaste `0.0.0.0:8157` y necesitas acceder desde la misma máquina usa `http://127.0.0.1:8157/`.

## 11) Crear aplicación `app_YouTube`

```bash
python manage.py startapp app_YouTube
```

---

# 3. Código esencial de archivos

A continuación los contenidos listos para pegar. Mantengo el modelo **Canal** que pediste, pero cambio la importación del `JSONField` a la versión estándar `models.JSONField` (funciona en Django >= 3.1 con cualquier DB; si quieres usar la versión de `contrib.postgres` avísame y lo ajusto y te indico `psycopg2`). Si quieres usar exactamente `from django.contrib.postgres.fields import JSONField` deberás usar PostgreSQL y `psycopg2-binary`.

## `app_YouTube/models.py`

```python
from django.db import models

class Canal(models.Model):
    id = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()
    fecha_creacion = models.DateField()
    suscriptores = models.IntegerField(default=0)
    url_personalizada = models.CharField(max_length=100, unique=True)
    pais = models.CharField(max_length=50)
    categoria_principal = models.CharField(max_length=50)

    def __str__(self):
        return self.nombre


class Video(models.Model):
    id = models.AutoField(primary_key=True)
    canal = models.ForeignKey(Canal, on_delete=models.CASCADE, related_name='videos')
    titulo = models.CharField(max_length=200)
    descripcion = models.TextField()
    duracion = models.DurationField()
    vistas = models.IntegerField(default=0)
    publico = models.BooleanField(default=True)
    url_miniatura = models.CharField(max_length=200)
    likes = models.CharField(max_length=200)

    def __str__(self):
        return f"{self.titulo} ({self.canal.nombre})"


class Playlist(models.Model):
    id = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=150)
    descripcion = models.TextField()
    fecha_creacion = models.DateField()
    publica = models.BooleanField(default=True)
    canal_creador = models.ForeignKey(Canal, on_delete=models.CASCADE, related_name='playlists')
    videos = models.ManyToManyField(Video, related_name='playlists')
    orden_videos = models.JSONField()  # usa JSONField estándar
    n_videos = models.DecimalField(max_digits=10, decimal_places=0)

    def __str__(self):
        return f"{self.nombre} - {self.canal_creador.nombre}"
```

---

## `app_YouTube/views.py`

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Canal
from django.urls import reverse
from django.http import HttpResponse

def inicio_YouTube(request):
    # página principal del sistema
    return render(request, 'inicio.html', {'titulo': 'Sistema de Administración YouTube'})

def agregar_canal(request):
    if request.method == 'POST':
        # sin validación (según indicación)
        nombre = request.POST.get('nombre')
        descripcion = request.POST.get('descripcion')
        fecha_creacion = request.POST.get('fecha_creacion')  # YYYY-MM-DD
        suscriptores = request.POST.get('suscriptores') or 0
        url_personalizada = request.POST.get('url_personalizada')
        pais = request.POST.get('pais')
        categoria_principal = request.POST.get('categoria_principal')

        Canal.objects.create(
            nombre=nombre,
            descripcion=descripcion,
            fecha_creacion=fecha_creacion,
            suscriptores=int(suscriptores),
            url_personalizada=url_personalizada,
            pais=pais,
            categoria_principal=categoria_principal
        )
        return redirect('ver_canal')
    return render(request, 'canal/agregar_canal.html')


def ver_canal(request):
    canales = Canal.objects.all().order_by('id')
    return render(request, 'canal/ver_canal.html', {'canales': canales})


def actualizar_canal(request, canal_id):
    canal = get_object_or_404(Canal, pk=canal_id)
    if request.method == 'POST':
        # aquí también podrías redirigir a una función separada, pero usaremos POST directo
        canal.nombre = request.POST.get('nombre')
        canal.descripcion = request.POST.get('descripcion')
        canal.fecha_creacion = request.POST.get('fecha_creacion')
        canal.suscriptores = int(request.POST.get('suscriptores') or 0)
        canal.url_personalizada = request.POST.get('url_personalizada')
        canal.pais = request.POST.get('pais')
        canal.categoria_principal = request.POST.get('categoria_principal')
        canal.save()
        return redirect('ver_canal')
    return render(request, 'canal/actualizar_canal.html', {'canal': canal})


def borrar_canal(request, canal_id):
    canal = get_object_or_404(Canal, pk=canal_id)
    if request.method == 'POST':
        canal.delete()
        return redirect('ver_canal')
    return render(request, 'canal/borrar_canal.html', {'canal': canal})
```

> Observación: tus pasos pedían `actualizar_canal` y `realizar_actualizacion_canal` por separado; aquí combino en una sola vista `actualizar_canal` que atiende GET (mostrar form) y POST (guardar cambios). Si quieres estrictamente separadas, dime y lo adapto.

---

## `app_YouTube/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_YouTube, name='inicio_YouTube'),
    path('canal/agregar/', views.agregar_canal, name='agregar_canal'),
    path('canal/ver/', views.ver_canal, name='ver_canal'),
    path('canal/actualizar/<int:canal_id>/', views.actualizar_canal, name='actualizar_canal'),
    path('canal/borrar/<int:canal_id>/', views.borrar_canal, name='borrar_canal'),
]
```

---

## Agregar `app_YouTube` a `INSTALLED_APPS` (settings.py)

Abre `backend_YouTube/settings.py` y en `INSTALLED_APPS` añade:

```python
INSTALLED_APPS = [
    # ... apps por defecto de Django
    'app_YouTube',
]
```

(No hace falta tocar `TEMPLATES` si `APP_DIRS=True` (por defecto) y las plantillas están en `app_YouTube/templates/`).

---

## `backend_YouTube/urls.py` (configurar para enlazar app_YouTube)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_YouTube.urls')),  # ruta raíz enlaza app_YouTube
]
```

---

## `app_YouTube/admin.py` (registrar modelos)

```python
from django.contrib import admin
from .models import Canal, Video, Playlist

@admin.register(Canal)
class CanalAdmin(admin.ModelAdmin):
    list_display = ('id', 'nombre', 'pais', 'suscriptores', 'url_personalizada')
    search_fields = ('nombre', 'pais')

@admin.register(Video)
class VideoAdmin(admin.ModelAdmin):
    list_display = ('id', 'titulo', 'canal', 'vistas')

@admin.register(Playlist)
class PlaylistAdmin(admin.ModelAdmin):
    list_display = ('id', 'nombre', 'canal_creador', 'n_videos')
```

---

# 4. Templates mínimos (HTML)

> Uso Bootstrap CDN. Paleta suave: puedes ajustar con clases `style` si quieres; dejo diseño sencillo y moderno.

## `app_YouTube/templates/base.html`

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}YouTube Admin{% endblock %}</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Bootstrap Icons -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.css">
    <style>
      body { background: #EAE0D5; color: #0A0908; padding-bottom: 70px; }
      .footer-fixed { position: fixed; left:0; right:0; bottom:0; background:#22333B; color:#EAE0D5; padding:10px 0; }
      .nav-bg { background: #22333B; }
      .content { padding: 20px; }
    </style>
  </head>
  <body>
    {% include 'navbar.html' %}
    <div class="container content">
      {% block content %}{% endblock %}
    </div>

    {% include 'footer.html' %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
  </body>
</html>
```

## `app_YouTube/templates/navbar.html`

```html
<nav class="navbar navbar-expand-lg nav-bg navbar-dark">
  <div class="container-fluid">
    <a class="navbar-brand" href="{% url 'inicio_YouTube' %}"><i class="bi bi-youtube"></i> Sistema de Administración YouTube</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio_YouTube' %}"><i class="bi bi-house"></i> Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-channel"></i> Canal</a>
          <ul class="dropdown-menu dropdown-menu-end">
            <li><a class="dropdown-item" href="{% url 'agregar_canal' %}">Agregar Canal</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_canal' %}">Ver Canal</a></li>
            <li><a class="dropdown-item" href="#">Actualizar Canal</a></li>
            <li><a class="dropdown-item" href="#">Borrar Canal</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-camera-video"></i> Video</a>
          <ul class="dropdown-menu dropdown-menu-end">
            <li><a class="dropdown-item" href="#">Agregar Video</a></li>
            <li><a class="dropdown-item" href="#">Ver Video</a></li>
            <li><a class="dropdown-item" href="#">Actualizar Video</a></li>
            <li><a class="dropdown-item" href="#">Borrar Video</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown"><i class="bi bi-list-check"></i> Playlist</a>
          <ul class="dropdown-menu dropdown-menu-end">
            <li><a class="dropdown-item" href="#">Agregar Playlist</a></li>
            <li><a class="dropdown-item" href="#">Ver Playlist</a></li>
            <li><a class="dropdown-item" href="#">Actualizar Playlist</a></li>
            <li><a class="dropdown-item" href="#">Borrar Playlist</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>
```

## `app_YouTube/templates/footer.html`

```html
<footer class="footer-fixed footer-fixed text-center footer-fixed">
  <div class="container">
    <small>© {{ now|date:"Y" }} — Creado por Sr.Alexander Cancino, Cbtis 128. Todos los derechos reservados.</small>
  </div>
</footer>
```

> Nota: en tus settings asegúrate de `TEMPLATES` tiene `'django.template.context_processors.request'` y el `now` puede venir si añades `django.template.context_processors.request` o usas `{% now "Y" %}`. Si `now` no funciona usa `{% now "Y" %}` directamente.

## `app_YouTube/templates/inicio.html`

```html
{% extends "base.html" %}
{% block title %}Inicio - YouTube Admin{% endblock %}
{% block content %}
  <div class="text-center">
    <h1>{{ titulo }}</h1>
    <p class="lead">Sistema sencillo para administrar Canales, Videos y Playlists.</p>
    <img src="https://upload.wikimedia.org/wikipedia/commons/b/b8/YouTube_Logo_2017.svg" alt="YouTube" style="max-width:300px;">
  </div>
{% endblock %}
```

---

## Templates para `canal/`

### `canal/agregar_canal.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Agregar Canal</h2>
<form method="post">
  {% csrf_token %}
  <div class="mb-3"><label>Nombre</label><input class="form-control" name="nombre"></div>
  <div class="mb-3"><label>Descripción</label><textarea class="form-control" name="descripcion"></textarea></div>
  <div class="mb-3"><label>Fecha creación</label><input type="date" class="form-control" name="fecha_creacion"></div>
  <div class="mb-3"><label>Suscriptores</label><input type="number" class="form-control" name="suscriptores"></div>
  <div class="mb-3"><label>URL personalizada</label><input class="form-control" name="url_personalizada"></div>
  <div class="mb-3"><label>País</label><input class="form-control" name="pais"></div>
  <div class="mb-3"><label>Categoría principal</label><input class="form-control" name="categoria_principal"></div>
  <button class="btn btn-primary" type="submit">Agregar</button>
</form>
{% endblock %}
```

### `canal/ver_canal.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Listar Canales</h2>
<table class="table table-striped">
  <thead><tr><th>#</th><th>Nombre</th><th>País</th><th>Suscriptores</th><th>Acciones</th></tr></thead>
  <tbody>
    {% for c in canales %}
    <tr>
      <td>{{ c.id }}</td>
      <td>{{ c.nombre }}</td>
      <td>{{ c.pais }}</td>
      <td>{{ c.suscriptores }}</td>
      <td>
        <a class="btn btn-sm btn-info" href="{% url 'actualizar_canal' c.id %}">Editar</a>
        <a class="btn btn-sm btn-danger" href="{% url 'borrar_canal' c.id %}">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="5">No hay canales</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}
```

### `canal/actualizar_canal.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Actualizar Canal</h2>
<form method="post">
  {% csrf_token %}
  <div class="mb-3"><label>Nombre</label><input class="form-control" name="nombre" value="{{ canal.nombre }}"></div>
  <div class="mb-3"><label>Descripción</label><textarea class="form-control" name="descripcion">{{ canal.descripcion }}</textarea></div>
  <div class="mb-3"><label>Fecha creación</label><input type="date" class="form-control" name="fecha_creacion" value="{{ canal.fecha_creacion }}"></div>
  <div class="mb-3"><label>Suscriptores</label><input type="number" class="form-control" name="suscriptores" value="{{ canal.suscriptores }}"></div>
  <div class="mb-3"><label>URL personalizada</label><input class="form-control" name="url_personalizada" value="{{ canal.url_personalizada }}"></div>
  <div class="mb-3"><label>País</label><input class="form-control" name="pais" value="{{ canal.pais }}"></div>
  <div class="mb-3"><label>Categoría principal</label><input class="form-control" name="categoria_principal" value="{{ canal.categoria_principal }}"></div>
  <button class="btn btn-success" type="submit">Guardar</button>
  <a class="btn btn-secondary" href="{% url 'ver_canal' %}">Cancelar</a>
</form>
{% endblock %}
```

### `canal/borrar_canal.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Confirmar borrado</h2>
<p>¿Deseas eliminar el canal "<strong>{{ canal.nombre }}</strong>"?</p>
<form method="post">
  {% csrf_token %}
  <button class="btn btn-danger" type="submit">Sí, borrar</button>
  <a class="btn btn-secondary" href="{% url 'ver_canal' %}">No, cancelar</a>
</form>
{% endblock %}
```

---

# 5. Migraciones (12.5)

Desde la raíz (`UIII_YouTube_0777`) con `.venv` activado:

```bash
# crear migraciones de la app
python manage.py makemigrations app_YouTube

# aplicar migraciones
python manage.py migrate
```

Si agregaste modelos y luego registraste en admin, vuelve a:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

# 6. Crear superusuario para admin (opcional)

```bash
python manage.py createsuperuser
# sigue prompts (email/username/password)
```

Luego accede a `http://127.0.0.1:8157/admin/`.

---

# 7. Nota sobre la base de datos y JSONField

* En el `models.py` usé `models.JSONField()` (compatible con SQLite y PostgreSQL en Django moderno).
* Si quieres insistir en `from django.contrib.postgres.fields import JSONField` usa PostgreSQL y `pip install psycopg2-binary`, y configura `DATABASES` en `settings.py`.

---

# 8. Ejecutar servidor en **puerto 8517** (paso final)

Cuando quieras ejecutar en 8517:

```bash
python manage.py runserver 0.0.0.0:8517
# o
python manage.py runserver 127.0.0.1:8517
```

Luego abre `http://127.0.0.1:8517/`.

---

# 9. Resumen de pasos a completar ahora (rápido)

1. Crear carpeta `UIII_YouTube_0777` → `cd` dentro.
2. `python -m venv .venv` → activar `.venv`.
3. `code .` → seleccionar intérprete `.venv`.
4. `pip install django`.
5. `django-admin startproject backend_YouTube .`
6. `python manage.py startapp app_YouTube`.
7. Pegar los archivos `models.py`, `views.py`, `urls.py`, `admin.py`, y las plantillas.
8. Añadir `app_YouTube` a `INSTALLED_APPS`.
9. `python manage.py makemigrations` → `python manage.py migrate`.
10. `python manage.py runserver 127.0.0.1:8157` (o 8517).

---

Si quieres, en la siguiente respuesta pego un *zip* del proyecto ya con la estructura y archivos listos (tendría que generar archivos aquí), o te genero los archivos completos uno por uno listos para crear localmente. ¿Qué prefieres ahora — que te genere los archivos listos para descargar o que haga algún ajuste (por ejemplo separar `actualizar_canal` en dos vistas, usar PostgreSQL, agregar validaciones o estilos con tu paleta exacta)?
