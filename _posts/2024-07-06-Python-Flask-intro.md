---
title: Python Flask intro
date: 2024-07-06 19:41:00 -0700
categories: [PYTHON, FLASK]
tags: [python, python flask]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

## Create venv

Create a new folder

```bash
$ mkdir demo_flask_board
$ cd demo_flask_board
```

Create a venv environment

On Windows

```powershell
python -m venv venv
.\venv\Scripts\activate
```

On Linux

```bash
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $
```

Example:

```bash
gus@ubuntu-vm:~/projects/demo_flask_board$ python3 -m venv venv
gus@ubuntu-vm:~/projects/demo_flask_board$ source venv/bin/activate
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ 
```

Install Python Flask using pip

```bash
python3 -m pip install Flask
```

<details markdown=1>
<summary markdown="span">Output</summary>

```bash
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ python3 -m pip install Flask
Collecting Flask
  Downloading flask-3.0.3-py3-none-any.whl (101 kB)
     |████████████████████████████████| 101 kB 4.9 MB/s 
Collecting itsdangerous>=2.1.2
  Downloading itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Collecting Jinja2>=3.1.2
  Downloading jinja2-3.1.4-py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 17.2 MB/s 
Collecting click>=8.1.3
  Downloading click-8.1.7-py3-none-any.whl (97 kB)
     |████████████████████████████████| 97 kB 9.2 MB/s 
Collecting importlib-metadata>=3.6.0; python_version < "3.10"
  Downloading importlib_metadata-8.0.0-py3-none-any.whl (24 kB)
Collecting blinker>=1.6.2
  Downloading blinker-1.8.2-py3-none-any.whl (9.5 kB)
Collecting Werkzeug>=3.0.0
  Downloading werkzeug-3.0.3-py3-none-any.whl (227 kB)
     |████████████████████████████████| 227 kB 8.1 MB/s 
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.5-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26 kB)
Collecting zipp>=0.5
  Downloading zipp-3.19.2-py3-none-any.whl (9.0 kB)
Installing collected packages: itsdangerous, MarkupSafe, Jinja2, click, zipp, importlib-metadata, blinker, Werkzeug, Flask
Successfully installed Flask-3.0.3 Jinja2-3.1.4 MarkupSafe-2.1.5 Werkzeug-3.0.3 blinker-1.8.2 click-8.1.7 importlib-metadata-8.0.0 itsdangerous-2.2.0 zipp-3.19.2
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$
```
</details><br />

## Run Flask

Create file `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```

Run with:

```bash
(venv) $ python app.py
```

<details markdown=1>
<summary markdown="span">Output</summary>

```bash
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ python app.py
 * Serving Flask app 'app'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://192.168.0.42:8000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 362-633-512
```
</details><br />

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/01-Hello-World.png)

## Run Flask (using Blueprints)

### Transform the Project Into a Package

```bash
(venv) $ mkdir board
(venv) $ mv app.py board/__init__.py
```

The result is that `board/__init__.py` is exactly the same code as `app.py` from the previous section.

Directory structure:

```bash
demo_flask_board/
│
└── board/
    └── __init__.py
```

Run with:

```bash
(venv) $ python -m flask --app board run --port 8000 --debug
```

<details markdown=1>
<summary markdown="span">Output</summary>

```bash
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ python -m flask --app board run --port 8000 --debug
 * Serving Flask app 'board'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:8000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 848-111-764
```
</details><br />

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/01-Hello-World.png)

### Application factory design pattern

With an application factory, the project’s structure becomes more organized. It encourages you to separate different parts of your application, like routes, configurations, and initializations, into different files later on. This encourages a cleaner and more maintainable codebase.

`board/__init__.py`

```python
from flask import Flask

def create_app():
    app = Flask(__name__)

    return app
```

### Using Blueprints

**Blueprints** are modules that contain related views that can be conveniently imported in `__init__.py`. For example, you’ll have a blueprint that stores the main pages of your project.

`board/pages.py`

```python
from flask import Blueprint

bp = Blueprint("pages", __name__)

@bp.route("/")
def home():
    return "Hello, Home!"

@bp.route("/about")
def about():
    return "Hello, About!"
```

`board/__init__.py`

```python
from flask import Flask

from board import pages

def create_app():
    app = Flask(__name__)

    app.register_blueprint(pages.bp)
    return app
```

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/01-Hello-World.png)

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/02-Hello-About.png)

## Templates

Project structure:

```bash
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ tree -I venv
.
└── board
    ├── __init__.py
    ├── pages.py
    ├── static
    │   └── styles.css
    └── templates
        ├── base.html
        ├── _navigation.html
        └── pages
            ├── about.html
            └── home.html

(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ 
```

`board/static/styles.css`

```css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  font-size: 20px;
  margin: 0 auto;
  text-align: center;
}

a,
a:visited {
  color: #007bff;
}

a:hover {
  color: #0056b3;
}

nav ul {
  list-style-type: none;
  padding: 0;
}

nav ul li {
  display: inline;
  margin: 0 5px;
}

main {
  width: 80%;
  margin: 0 auto;
}
```

`board/templates/_navigation.html`

{% raw %}
```html
<nav>
  <ul>
    <li><a href="{{ url_for('pages.home') }}">Home</a></li>
    <li><a href="{{ url_for('pages.about') }}">About</a></li>
  </ul>
</nav>
```
{% endraw %}

`board/templates/base.html`

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Message Board - {% block title %}{% endblock title %}</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
</head>
<body>
<h1>Message Board</h1>
{% include("_navigation.html") %}
<section>
  <header>
    {% block header %}{% endblock header %}
  </header>
  <main>
    {% block content %}<p>No messages.</p>{% endblock content %}
  </main>
</section>
</body>
</html>
```
{% endraw %}

`board/templates/pages/home.html`

{% raw %}
```html
{% extends 'base.html' %}

{% block header %}
  <h2>{% block title %}Home{% endblock title %}</h2>
{% endblock header %}

{% block content %}
  <p>
    Learn more about this project by visiting the <a href="{{ url_for('pages.about') }}">About page</a>.
  </p>
{% endblock content %}
```
{% endraw %}

`board/templates/pages/about.html`

{% raw %}
```html
{% extends 'base.html' %}

{% block header %}
  <h2>{% block title %}About{% endblock title %}</h2>
{% endblock header %}

{% block content %}
  <p>This is a message board for friendly messages.</p>
{% endblock content %}
```
{% endraw %}

`board/pages.py`

```python
from flask import Blueprint, render_template

bp = Blueprint("pages", __name__)

@bp.route("/")
def home():
    return render_template("pages/home.html")

@bp.route("/about")
def about():
    return render_template("pages/about.html")
```

Run with:

```bash
python -m flask --app board run --port 8000 --debug
```

<details markdown=1>
<summary markdown="span">Output</summary>

```bash
(venv) gus@ubuntu-vm:~/projects/demo_flask_board$ python -m flask --app board run --port 8000 --debug
 * Serving Flask app 'board'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:8000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 848-111-764
```
</details><br />

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/03-Templates-Home.png)

![]({{ site.baseurl }}/images/2024/07-06-Python-Flask-intro/04-Templates-About.png)

## References

- https://realpython.com/flask-project/