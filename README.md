# 👋 Hello, this is a django docker repository with Postgresql

<p>I will explain the step by step you must follow to have a Docker environment with Django and Postgresql</p>

📢Alert: **You need to have Python and Docker installed on your machine.**

## Starting the project

### 1. Clone the project

~~~bash
git clone git@github.com:AndreCamargoo/docker-django-postgresql.git
~~~

### 2. Virtual environment

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

### 6. Environment variables

<p>Note that there is a folder called dotenv_files, you must create a <b>.env</b> to be used by <b>settings.py</b><br> 
There is an example to be copied and then modified with your credentials.</p>

~~~bash
cd dotenv_files
~~~

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

### 9. Build Image

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

<p>Ou</p>

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

### 10. Useful commands

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

