### Check the python version
```
python -V
```

### install django env
```
conda create --name MyDjangoEnv python=3.7.6
```

### To activate this environment, use
```
conda activate MyDjangoEnv
```

### To deactivate an active environment, use
```
conda deactivate
```

### to get the information about the env
```
conda info --envs
```
### get into the virtual environment and install django
```
1) conda activate MyDjangoEnv

2) conda install django
```

### Create a folder
```
mkdir My_Django_Stuff
cd My_Django_Stuff 

conda activate MyDjangoEnv
django-admin startproject frontend_bite
```

### Go to the frontend_bite folder 
```
conda activate MyDjangoEnv (if you are not in virtual env)
python manage.py runserver
```

### make a new project
```
python manage.py startapp first_app
```

### add a new folder route in settings.py
```
add 'first_app' in INSTALLED_APPS

ex)

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'first_app'
]
```

### run server

```
python manage.py runserver
```

### update view.py
```
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.

def index(request):
    returnr HttpResponse("Hello World")

```



