# Práctica Didáctica: CRUD de Alumnos con Python, FastAPI y SQLite

**Nivel:** 5.º semestre de preparatoria
**Tema:** Desarrollo Web Backend con FastAPI
**Proyecto:** CRUD de Alumnos (Crear, Listar, Actualizar y Eliminar)
**Base de Datos:** SQLite
**IDE:** Visual Studio Code

---

# Objetivo

Desarrollar una aplicación web utilizando **FastAPI** y **SQLite** que permita administrar alumnos mediante las operaciones CRUD:

* Crear alumnos (POST)
* Consultar alumnos (GET)
* Actualizar alumnos (PUT)
* Eliminar alumnos (DELETE)

La aplicación utilizará:

* Python
* FastAPI
* SQLAlchemy
* SQLite
* Jinja2 Templates
* HTML5
* CSS3

---

# PASO 1. Crear la carpeta del proyecto

Crear una carpeta llamada:

```text
crudalumno
```

Estructura inicial:

```text
crudalumno
```

---

# PASO 2. Abrir la carpeta en VS Code

Abrir Visual Studio Code.

Seleccionar:

```text
Archivo → Abrir Carpeta
```

Elegir:

```text
crudalumno
```

---

# PASO 3. Verificar instalación de Python

Abrir la terminal integrada:

```text
Terminal → New Terminal
```

Ejecutar:

```bash
python --version
```

o

```bash
py --version
```

Resultado esperado:

```text
Python 3.12.x
```

---

# PASO 4. Crear el entorno virtual

Desde la terminal ejecutar:

```bash
python -m venv .venv
```

Se creará:

```text
.venv
```

Este entorno permite instalar librerías sin afectar otras aplicaciones.

---

# PASO 5. Activar el entorno virtual (Windows)

Ejecutar:

```bash
.venv\Scripts\activate
```

Resultado:

```text
(.venv) C:\crudalumno>
```

---

# PASO 6. Instalar FastAPI y dependencias

## Instalar FastAPI

```bash
pip install fastapi
```

---

## Instalar servidor Uvicorn

```bash
pip install uvicorn
```

---

## Instalar SQLAlchemy

Permite trabajar con bases de datos.

```bash
pip install sqlalchemy
```

---

## Instalar Jinja2

Permite generar páginas HTML dinámicas.

```bash
pip install jinja2
```

---

## Instalar soporte para formularios

```bash
pip install python-multipart
```

---

## Verificar instalación

```bash
pip list
```

Debe aparecer:

```text
fastapi
uvicorn
sqlalchemy
jinja2
python-multipart
```

---

# PASO 7. Crear la estructura del proyecto

```text
crudalumno
│
├── .venv
│
├── main.py
├── modelo.py
├── conectarBD.py
│
├── templates
│   └── index.html
│
├── static
│   └── style.css
│
└── alumnos.db
```

---

# PASO 8. Crear archivo modelo.py

Archivo:

```python
# modelo.py

from sqlalchemy import Column, Integer, String
from conectarBD import Base


class Alumno(Base):
    __tablename__ = "alumnos"

    id = Column(Integer, primary_key=True, index=True)
    nombre = Column(String)
    edad = Column(Integer)
    sexo = Column(String)
    correo = Column(String)
```

---

## Explicación

```python
from sqlalchemy import Column
```

Importa columnas para la tabla.

---

```python
Integer
```

Tipo entero.

---

```python
String
```

Tipo texto.

---

```python
__tablename__
```

Nombre de la tabla.

---

```python
primary_key=True
```

Indica llave primaria.

---

# PASO 9. Crear conectarBD.py

```python
# conectarBD.py

from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./alumnos.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}
)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

Base = declarative_base()
```

---

## Explicación

```python
DATABASE_URL
```

Ruta de la base de datos SQLite.

---

```python
create_engine()
```

Conecta Python con SQLite.

---

```python
SessionLocal
```

Permite crear sesiones de trabajo.

---

```python
Base
```

Clase base para las tablas.

---

# PASO 10. Crear archivo principal main.py

```python
# main.py

from fastapi import FastAPI
from fastapi import Request
from fastapi import Form
from fastapi.responses import RedirectResponse

from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

from sqlalchemy.orm import Session

from conectarBD import SessionLocal
from conectarBD import engine

import modelo

app = FastAPI()

modelo.Base.metadata.create_all(bind=engine)

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")


def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()


@app.get("/")
def inicio(request: Request):

    db = SessionLocal()

    alumnos = db.query(modelo.Alumno).all()

    return templates.TemplateResponse(
        "index.html",
        {
            "request": request,
            "alumnos": alumnos
        }
    )


@app.post("/guardar")
def guardar(
        nombre: str = Form(...),
        edad: int = Form(...),
        sexo: str = Form(...),
        correo: str = Form(...)
):

    db = SessionLocal()

    nuevo = modelo.Alumno(
        nombre=nombre,
        edad=edad,
        sexo=sexo,
        correo=correo
    )

    db.add(nuevo)
    db.commit()

    return RedirectResponse("/", status_code=303)


@app.post("/eliminar/{id}")
def eliminar(id: int):

    db = SessionLocal()

    alumno = db.query(modelo.Alumno).filter(
        modelo.Alumno.id == id
    ).first()

    if alumno:
        db.delete(alumno)
        db.commit()

    return RedirectResponse("/", status_code=303)


@app.post("/actualizar/{id}")
def actualizar(
        id: int,
        nombre: str = Form(...),
        edad: int = Form(...),
        sexo: str = Form(...),
        correo: str = Form(...)
):

    db = SessionLocal()

    alumno = db.query(modelo.Alumno).filter(
        modelo.Alumno.id == id
    ).first()

    if alumno:
        alumno.nombre = nombre
        alumno.edad = edad
        alumno.sexo = sexo
        alumno.correo = correo

        db.commit()

    return RedirectResponse("/", status_code=303)
```

---

# Métodos HTTP utilizados

## GET

Obtiene información.

```python
@app.get("/")
```

Muestra los alumnos registrados.

---

## POST

Guarda datos.

```python
@app.post("/guardar")
```

Inserta nuevos alumnos.

---

## PUT

Actualiza información.

Ejemplo API:

```python
@app.put("/actualizar/{id}")
```

Actualiza registros.

---

## DELETE

Elimina información.

Ejemplo API:

```python
@app.delete("/eliminar/{id}")
```

Borra registros.

---

# PASO 11. Crear carpeta templates

Crear:

```text
templates
```

Dentro:

```text
index.html
```

---

# Código index.html

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">
    <title>CRUD Alumnos</title>

    <link rel="stylesheet"
          href="/static/style.css">
</head>

<body>

<div class="contenedor">

<h1>📚 CRUD DE ALUMNOS</h1>

<form action="/guardar" method="post">

<input type="text"
       name="nombre"
       placeholder="Nombre"
       required>

<input type="number"
       name="edad"
       placeholder="Edad"
       required>

<input type="text"
       name="sexo"
       placeholder="Sexo"
       required>

<input type="email"
       name="correo"
       placeholder="Correo"
       required>

<button type="submit">
Guardar Alumno
</button>

</form>

<table>

<tr>
<th>ID</th>
<th>Nombre</th>
<th>Edad</th>
<th>Sexo</th>
<th>Correo</th>
<th>Acción</th>
</tr>

{% for alumno in alumnos %}

<tr>

<td>{{ alumno.id }}</td>
<td>{{ alumno.nombre }}</td>
<td>{{ alumno.edad }}</td>
<td>{{ alumno.sexo }}</td>
<td>{{ alumno.correo }}</td>

<td>

<form action="/eliminar/{{ alumno.id }}"
method="post">

<button class="eliminar">
Eliminar
</button>

</form>

</td>

</tr>

{% endfor %}

</table>

</div>

</body>
</html>
```

---

# PASO 12. Crear carpeta static

Crear:

```text
static
```

Dentro:

```text
style.css
```

---

# Código style.css

```css
body{

font-family: Arial, sans-serif;

background:
linear-gradient(
135deg,
#667eea,
#764ba2);

margin:0;
padding:0;

}

.contenedor{

width:90%;

margin:auto;

margin-top:30px;

background:white;

padding:20px;

border-radius:15px;

box-shadow:
0 10px 25px rgba(0,0,0,.2);

}

h1{

text-align:center;

color:#4a148c;

}

form{

display:flex;

gap:10px;

margin-bottom:20px;

flex-wrap:wrap;

}

input{

padding:10px;

border:1px solid #ccc;

border-radius:8px;

}

button{

background:#4caf50;

color:white;

border:none;

padding:10px 20px;

border-radius:8px;

cursor:pointer;

}

button:hover{

background:#388e3c;

}

table{

width:100%;

border-collapse:collapse;

}

th{

background:#673ab7;

color:white;

padding:10px;

}

td{

padding:10px;

border:1px solid #ddd;

}

.eliminar{

background:#e53935;

}

.eliminar:hover{

background:#c62828;

}
```

---

# PASO 13. Ejecutar el servidor

Desde terminal:

```bash
uvicorn main:app --reload
```

---

Resultado:

```text
INFO:
Uvicorn running on

http://127.0.0.1:8000
```

---

# PASO 14. Abrir en navegador

Abrir:

```text
http://127.0.0.1:8000
```

---

# PASO 15. Probar CRUD

## Crear alumno

Capturar:

```text
Juan Pérez
17
Masculino
juan@gmail.com
```

Presionar:

```text
Guardar Alumno
```

---

## Listar alumnos

Se mostrarán en la tabla.

---

## Eliminar alumno

Presionar:

```text
Eliminar
```

---

## Actualizar alumno

Como práctica adicional, agregar formularios de edición o implementar:

```python
@app.put("/actualizar/{id}")
```

para modificar registros mediante formularios dinámicos.

---

# Resultado Final

El estudiante aprenderá:

✅ Crear proyectos FastAPI
✅ Crear entorno virtual
✅ Instalar dependencias con pip
✅ Utilizar SQLite
✅ Crear modelos con SQLAlchemy
✅ Manejar sesiones de base de datos
✅ Utilizar GET, POST, PUT y DELETE
✅ Crear interfaces web con HTML y CSS
✅ Ejecutar aplicaciones con Uvicorn
✅ Implementar un CRUD completamente funcional de alumnos con diseño moderno y atractivo.
