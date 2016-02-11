# Instalación de Django para Python 3 en Raspbian

### Instalar Django

- Instalar pip3 (gestor de paquetes de Python 3)
```bash
$ sudo apt-get install python3-pip
```
- Instalar Django para Python 3
```bash
$ sudo pip3 install Django==1.9.2 # Ultima version estable (11-02-2016)
```
- Probar que Django esta instalado correctamente
```bash
$ python3 -c "import django; print(django.get_version())"
```

### Crear proyecto

- Crear un nuevo proyecto en el directorio actual
```bash
django-admin startproject [NOMBRE-PROYECTO]
```
Entonces creara una carpeta con lo siguiente
```
[NOMBRE-PROYECTO]/
    manage.py
    [NOMBRE-PROYECTO]/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```
- Para lanzar el servidor hacemos lo siguiente
```bash
$ python3 manage.py runserver # Por defecto localhost:8000
$ python3 manage.py runserver 8080
$ python3 manage.py runserver 0.0.0.0:8080
```
- Para crear una App en Django haremos lo siguiente
```bash
$ python manage.py startapp [NOMBRE-APP]
```
Nos creara una carpeta dentro de la carpeta de nuestro proyecto con lo siguiente
```
[NOMBRE-APP]/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
Quedando hasta el momento la estructura del proyecto de la siguiente manera
```
[NOMBRE-PROYECTO]/
    manage.py
    [NOMBRE-PROYECTO]/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    [NOMBRE-APP]/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
        models.py
        tests.py
        views.py
```

### Ejemplo sencillo

Para probar que todo funciona correctamente creamos un ejemplo muy sencillo. Para ello seguimos los sigueitnes pasos:

- Añadimos en el archivo ```views.py``` de nuestra app lo siguiente:
```python
from django.shortcuts import render
from django.http import HttpResponse
def index(request):
    return HttpResponse("Hello, world! You're at the DjangoAPP index.\n")
```
- Creamos dentro de uestra app un archivo llamado ```urls.py```. Mapeamos la URL añadiendo lo siguiente al archivo ```urls.py``` de nuestra app:
```python
from django.conf.urls import url
from . import views
urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```
- Modificamso el archivo ```urls.py``` **del proyecto** para que quede de la siguiente manera:
```python
from django.conf.urls import include, url
from django.contrib import admin
urlpatterns = [
    url(r'^AppPRUEBA/', include('AppPRUEBA.urls')),
    url(r'^admin/', admin.site.urls),
]
```
- Lanzamos el servidor:
```bash
$ python3 manage.py runserver
```
- Desde una terminal del mismo dispositivo podemos comprobar que funciona usando ```curl``` de la siguiente manera:
```bash
$ curl -L http://localhost:8000/AppPRUEBA
Hello, world! You're at the DjangoAPP index.
```
