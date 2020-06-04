Check the python version
```
python -V
```

install django env
```
conda create --name MyDjangoEnv python=3.7.6
```

# To activate this environment, use
#
#     $ conda activate MyDjangoEnv
#
# To deactivate an active environment, use
#
#     $ conda deactivate

# to get the information about the env
```
conda info --envs
```

# get into the virtual environment and install django
```
1) conda activate MyDjangoEnv

2) conda install django
```

# Create a folder
```
mkdir My_Django_Stuff
cd My_Django_Stuff 

conda activate MyDjangoEnv
django-admin startproject frontend_bite

```

# Go to the frontend_bite folder 
```
conda activate MyDjangoEnv (if you are not in virtual env)
python manage.py runserver
```


