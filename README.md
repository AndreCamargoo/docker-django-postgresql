# 👋 Hello, this is a django docker repository with Postgresql

<p>I will explain the step by step you must follow to have a Docker environment with Django and Postgresql</p>

📢Alert: **You need to have Python and Docker installed on your machine.**

## Starting the project

### 1. You copy the entire project or follow the steps below

~~~bash
git clone git@github.com:AndreCamargoo/docker-django-postgresql.git
~~~

### 2. Create a folder and let's create a virtual environment

<p>
Just so the editor doesn't complain that nothing is installed, the virtual environment itself will be inside the container. <br>
This environment will not be used, but always leave it active in a development environment, to execute necessary Python commands.
</p>

~~~bash
python -m venv venv
~~~

<p>Or</p>

~~~bash
python3 -m venv venv
~~~

<p>Windows</p>

~~~bash
venv/Scripts/activate
~~~

<p>Linux</p>

~~~bash
source venv/bin/activate
~~~

### 3. Installing Django

<p>Before installing, let's update pip! Typically virtual environments come with outdated pip</p>

~~~bash
pip install pip --upgrade
~~~

<p>Now let's install Django</p>

~~~bash
pip install django
~~~

### 4. Starting a Django project

<p>Create djangoapp folder in the root of your project</p>

~~~bash
mkdir djangoapp
~~~

<p>Navigate into the djangoapp folder</p>

<p>This folder is where our Docker will monitor all changes to the Django project (WORKDIR)</p>

~~~bash
cd djangoapp
~~~

<p>
Inside the djangoapp folder. <b>make sure you are in the folder</b><br>
<b>📢 Alert:</b> Enter the command below without the quotation marks <b>"</b> and replace with your project name! make sure you put the point<b>.</b> 
at the end to create the project in the root of the folder
</p>

~~~bash
django-admin startproject "name of your project" .
~~~

<p>After creating the project, exit the djangoapp folder!</p>

~~~bash
cd ..
~~~

### 5. Requiments.txt

<p>Inside the <b>djangoapp</b> folder, you must create a requirements.txt file with your dependencies to be used by Docker.</p>

~~~bash
cd djangoapp
~~~

<p>Command <b>"cat > requirements.txt"</b>, From there, you can write in the terminal until you press “Ctrl” + “d”, and you can use the “Enter” key to skip lines normally.</p>

<p>Psycopg binary is the most popular PostgreSQL database adapter for the Python programming language.</p>

<p>You can find it at:</p>

[Pypi.org](https://pypi.org/)

~~~bash
cat > requirements.txt 
Django 
psycopg2-binary
~~~

<p>Go back to the project root again</p>

~~~bash
cd ..
~~~

### 6. Environment variables

<p>Let's create a folder in the root of the project called dotenv_files to be used by <b>settings.py</b></p>

~~~bash
mkdir dotenv_files
~~~

<p>Navigate to the folder and we will create a .env-example file</p>
<p>This file will serve as a base to start the project, you can share this project with your team or friends!</p>

~~~bash
cd dotenv_files
~~~

```
cat > .env-example
SECRET_KEY="CHANGE-ME"

# 0 False, 1 True
DEBUG="1"

# Comma Separated values
ALLOWED_HOSTS="127.0.0.1, localhost"

DB_ENGINE="django.db.backends.postgresql"
POSTGRES_DB="CHANGE-ME"
POSTGRES_USER="CHANGE-ME"
POSTGRES_PASSWORD="CHANGE-ME"
POSTGRES_HOST="psql"
POSTGRES_PORT="5432"

LANGUAGE_CODE="CHANGE-ME"
TIME_ZONE="CHANGE-ME"
```

<p>Creating a copy of <b>.env-example</b> and renaming it to <b>.env</b></p>

~~~bash
cp .env-example .env
~~~

<p>Now you must change the credentials of the <b>.env</b> file, this can be through the editor or through the terminal itself using nano or another one of your choice.</p>

~~~bash
nano .env
~~~

<p>How to generate a secret key</p>

~~~~bash
python3 -c "import string as s; from secrets import SystemRandom as SR; print(''.join(SR().choices(s.ascii_letters+s.digits+s.punctuation, k=64)))"
~~~~

<p><b>📢 Alert:</b> If the secret key contains any type of single or double quotes, remove it as it may cause problems in the future.</p>

<p><b>📢 Alert:</b> POSTGRES_HOST is a service in your container, so it should be "psql" or whatever name you choose.</p>

### 7. Editing the SETTINGS

<p>At this point you already have a Django project created, let's edit settings.py to use our environment variables. Through the editor or terminal</p>

~~~bash
cd djangoapp/seuprojeto
~~~

~~~bash
nano settings.py
~~~

<p>Steps to be edited or added:<br> 
<b>📢 Alert:</b> when there is a <b style="color:green">></b> character, it must be replaced</p>

- import os
- SECRET_KEY = 'django-insecure...' <b style="color:green">></b> SECRET_KEY = os.getenv('SECRET_KEY', 'change-me')
- DEBUG = TRUE <b style="color:green">></b> DEBUG = bool(int(os.getenv('DEBUG', 0)))
- ALLOWED_HOSTS = [] <b style="color:green">></b> ALLOWED_HOSTS = [h.strip() for h in os.getenv('ALLOWED_HOSTS', '').split(',') if h.strip()]
- LANGUAGE_CODE = 'en-us' <b style="color:green">></b> os.getenv('LANGUAGE_CODE', 'en-us')
- TIME_ZONE = 'UTC' <b style="color:green">></b> os.getenv('TIME_ZONE', 'UTC')
- MEDIA_URL = 'media/'
- STATIC_ROOT = os.path.join(BASE_DIR.parent, 'data/web/staticfiles')
- MEDIA_ROOT = os.path.join(BASE_DIR.parent, 'data/web/media')

<p>DataBases</p>

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

<p>Replace with:</p>

```
DATABASES = {
    'default': {
        'ENGINE': os.getenv('DB_ENGINE', 'change-me'),
        'NAME': os.getenv('POSTGRES_DB', 'change-me'),
        'USER': os.getenv('POSTGRES_USER', 'change-me'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', 'change-me'),
        'HOST': os.getenv('POSTGRES_HOST', 'change-me'),
        'PORT': os.getenv('POSTGRES_PORT', 'change-me'),
    }
}
```

### 8. Editing the URLS

- from django.conf import settings
- from django.conf.urls.static import static

<p>Below the urlpatterns:</p>

```
if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT
    )
```

### 9. Create DockerFile, docker-compose and commands sh

<p>Let's create a Dockerfile at the root of the project</p>

```
cat > Dockerfile 
FROM python:3.11.3-alpine3.18
LABEL mantainer="andre.camargo@msn.com"

# Essa variável de ambiente é usada para controlar se o Python deve 
# gravar arquivos de bytecode (.pyc) no disco. 1 = Não, 0 = Sim
ENV PYTHONDONTWRITEBYTECODE 1

# Define que a saída do Python será exibida imediatamente no console ou em 
# outros dispositivos de saída, sem ser armazenada em buffer.
# Em resumo, você verá os outputs do Python em tempo real.
ENV PYTHONUNBUFFERED 1

# Copia a pasta "djangoapp" e "scripts" para dentro do container.
COPY djangoapp /djangoapp
COPY scripts /scripts

# Entra na pasta djangoapp no container (trabalhar com essa pasta)
WORKDIR /djangoapp

# A porta 8000 estará disponível para conexões externas ao container
# É a porta que vamos usar para o Django.
EXPOSE 8000

# RUN executa comandos em um shell dentro do container para construir a imagem. 
# O resultado da execução do comando é armazenado no sistema de arquivos da 
# imagem como uma nova camada.
# Agrupar os comandos em um único RUN pode reduzir a quantidade de camadas da 
# imagem e torná-la mais eficiente.
RUN python -m venv /venv && \
  /venv/bin/pip install --upgrade pip && \
  /venv/bin/pip install -r /djangoapp/requirements.txt && \
  adduser --disabled-password --no-create-home duser && \
  mkdir -p /data/web/static && \
  mkdir -p /data/web/staticfiles && \
  mkdir -p /data/web/media && \
  chown -R duser:duser /venv && \
  chown -R duser:duser /data/web/static && \
  chown -R duser:duser /data/web/staticfiles && \
  chown -R duser:duser /data/web/media && \
  chmod -R 755 /data/web/static && \
  chmod -R 755 /data/web/staticfiles && \
  chmod -R 755 /data/web/media && \
  chmod -R +x /scripts

# Adiciona a pasta scripts e venv/bin 
# no $PATH do container.
ENV PATH="/scripts:/venv/bin:$PATH"

# Muda o usuário para duser
USER duser

# Executa o arquivo scripts/commands.sh
CMD ["commands.sh"]
```

<p>We need to create a scripts folder in the project root and a file inside the commands.sh folder that will be used by the Dockerfile to execute sh commands</p>

~~~bash
mkdir scripts && cd scripts
~~~

```
cat > commands.sh
#!/bin/sh

# O shell irá encerrar a execução do script quando um comando falhar
set -e

while ! nc -z $POSTGRES_HOST $POSTGRES_PORT; do
  echo "🟡 Waiting for Postgres Database Startup ($POSTGRES_HOST $POSTGRES_PORT) ..."
  sleep 2
done

echo "✅ Postgres Database Started Successfully ($POSTGRES_HOST:$POSTGRES_PORT)"

python manage.py collectstatic --noinput
python manage.py makemigrations --noinput
python manage.py migrate --noinput
python manage.py runserver 0.0.0.0:8000 
```

<p>Now let's create docker-compose.yml in the project root</p>

~~~bash
cd ..
~~~

```
cat > docker-compose.yml
version: '3.9'

services:
  djangoapp:
    container_name: djangoapp
    build:
      context: .
      dockerfile: ./Dockerfile
    ports:
      - 8000:8000
    volumes:
      - ./djangoapp:/djangoapp
      - ./data/web/static:/data/web/static/
      - ./data/web/media:/data/web/media/
    env_file:
      - ./dotenv_files/.env
    depends_on:
      - psql
  psql:
    container_name: psql
    image: postgres:13-alpine
    ports:
      - "5432:5432"
    volumes:
      - ./data/postgres/data:/var/lib/postgresql/data/
    env_file:
      - ./dotenv_files/.env
volumes:
  psql:
    driver: local
```

### 10. Build Image

<p>Make sure docker is active <br>
Let's build the image for the first time!
</p>

~~~bash
docker-compose up --build
~~~

<p>If you make changes to the DockerFile, docker-compose.yml and .env, you will need to rebuild the image</p>

~~~bash
docker-compose up --build --force-recreate
~~~

<p>Start the container</p>

~~~bash
docker-compose up
~~~

<p>Or</p>

~~~bash
docker-compose up -d
~~~

<p>Wax the container</p>

~~~bash
docker-compose down
~~~

<p>Project URL when container is active:</p>

```
http://localhost:8000/
http://127.0.0.1:8000/
```

<p><b>Alert:</b> If you ran the application and it generated an error like:</p>

```
Page not found (404)
    Request Method:	GET
        Request URL:	http://localhost:8000/

Using the URLconf defined in project.urls, Django tried these URL patterns, in this order:

    1.admin/
    2.^media/(?P<path>.*)$
The empty path didn’t match any of these.

You’re seeing this error because you have DEBUG = True in your Django settings file. Change that to False, and Django will display a standard 404 page.
```

<p>Just comment out the lines in your urls file</p>

```
"""
if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT
    )
"""
```

<p>After you create an app and include it in your routes, you can return the lines to use averages. ex:</p>

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('chat.urls')),
]

if settings.DEBUG:
    urlpatterns += static(
        settings.MEDIA_URL,
        document_root=settings.MEDIA_ROOT
    )
```    

### 11. Useful commands

~~~bash
docker-compose run djangoapp python manage.py startproject project .
~~~

~~~bash
docker-compose run djangoapp python manage.py startapp chat
~~~

~~~bash
docker-compose run djangoapp python manage.py createsuperuser
~~~

~~~bash
docker-compose run djangoapp python manage.py makemigrations
~~~

~~~bash
docker-compose run djangoapp python manage.py migrate
~~~

~~~bash
docker-compose run djangoapp python manage.py collectstatic
~~~

~~~bash
docker-compose exec -it djangoapp sh
~~~

<p>Observe the virtual environment of the container after entering the shell</p>

~~~bash
cd ..
ls
python -V
which python
~~~

### 12. Files to ignore if using git

<p>At the root of the project we will create some files that are useful in everyday life</p>

- .vscode
- .dockerignore
- .gitignore

#### .VSCODE

~~~bash
mkdir .vscode && cd .vscode
~~~

<p>Create file settings.json</p>

```
cat > settings.json 
{
    "window.zoomLevel": 2,
    "editor.fontSize": 24,
    "editor.hover.enabled": true,
    "workbench.startupEditor": "none",
    "explorer.compactFolders": false,
    "terminal.integrated.fontSize": 24,
    "editor.rulers": [
        80,
        120
    ],
    "workbench.colorTheme": "OM Theme (Default Dracula Italic)",
    "workbench.iconTheme": "material-icon-theme",
    "code-runner.executorMap": {
        "python": "clear ; python -u"
    },
    "code-runner.runInTerminal": true,
    "code-runner.ignoreSelection": true,
    "editor.fontFamily": "'Fira Code', Consolas, 'Dank Mono', 'Source Code Pro', 'Fira Code', Menlo, 'Inconsolata', 'Droid Sans Mono', 'DejaVu Sans Mono', 'Ubuntu Mono', 'Courier New', Courier, Monaco, monospace",
    "terminal.integrated.fontFamily": "",
    "editor.fontLigatures": false,
    "[python]": {
        "editor.defaultFormatter": "ms-python.autopep8",
        "editor.tabSize": 4,
        "editor.insertSpaces": true,
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll": true,
            "source.fixAll.unusedImports": true,
            "source.organizeImports": true
        }
    },
    "python.languageServer": "Pylance",
    "python.formatting.autopep8Args": [
        "--indent-size=4",
        "--max-line-length=80"
        // "--ignore=E111"
    ],
    "python.linting.flake8Args": [
        // "--ignore=E111",
    ],
    "python.linting.flake8Enabled": true,
    "python.linting.mypyEnabled": true,
    "python.testing.unittestEnabled": false,
    "python.testing.pytestEnabled": true,
    "python.analysis.diagnosticSeverityOverrides": {
        "reportUnknownMemberType": "none",
        "reportUnknownArgumentType": "none",
        "reportUnknownVariableType": "none",
        "reportUnknownLambdaType": "none",
        "reportUnknownParameterType": "none"
    },
    "python.defaultInterpreterPath": "./venv/bin/python",
    "python.analysis.typeCheckingMode": "basic",
    "cSpell.enabled": true,
    "python.formatting.provider": "none"
}
```

#### .DOCKERIGNORE

~~~bash
cd ..
~~~

<p>Create file .dockerignore</p>

```
cat > .dockerignore
*.pyc
*.pyo
*.mo
*.db
*.css.map
*.egg-info
*.sql.gz
.cache
.project
.idea
.pydevproject
.idea/workspace.xml
.DS_Store
.git/
.sass-cache
.vagrant/
__pycache__
dist
docs
env
logs
stats
Dockerfile
__localcode/
``` 

#### .GITIGNORE

<p>Create file .gitignore</p>

```
cat > .gitignore
/djangoapp/static/
/djangoapp/media/
/djangoapp/project/local_settings.py
gunicorn-error-log
__localcode
/data/

# Django #
*.log
*.pot
*.pyc
__pycache__
db.sqlite3

# Backup files #
*.bak

# If you are using PyCharm #
# User-specific stuff
.idea/**/workspace.xml
.idea/**/tasks.xml
.idea/**/usage.statistics.xml
.idea/**/dictionaries
.idea/**/shelf

# AWS User-specific
.idea/**/aws.xml

# Generated files
.idea/**/contentModel.xml

# Sensitive or high-churn files
.idea/**/dataSources/
.idea/**/dataSources.ids
.idea/**/dataSources.local.xml
.idea/**/sqlDataSources.xml
.idea/**/dynamic.xml
.idea/**/uiDesigner.xml
.idea/**/dbnavigator.xml

# Gradle
.idea/**/gradle.xml
.idea/**/libraries

# File-based project format
*.iws

# IntelliJ
out/

# JIRA plugin
atlassian-ide-plugin.xml

# Python #
*.py[cod]
*$py.class

# Distribution / packaging
.Python build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.coverage
.coverage.*
.cache
.pytest_cache/
nosetests.xml
coverage.xml
*.cover
.hypothesis/

# Jupyter Notebook
.ipynb_checkpoints

# pyenv
.python-version

# celery
celerybeat-schedule.*

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# mkdocs documentation
/site

# mypy
.mypy_cache/

# Sublime Text #
*.tmlanguage.cache
*.tmPreferences.cache
*.stTheme.cache
*.sublime-workspace
*.sublime-project

# sftp configuration file
sftp-config.json

# Package control specific files Package
Control.last-run
Control.ca-list
Control.ca-bundle
Control.system-ca-bundle
GitHub.sublime-settings

# Visual Studio Code #
.vscode/*
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
.history
```
