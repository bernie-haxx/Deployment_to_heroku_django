How to Deploy Django Applications on Heroku
===========================================

![heroku-django](https://simpleisbetterthancomplex.com/media/2016-08-09-how-to-deploy-django-applications-on-heroku/featured.jpg)
# Install heroku CLI
[Sign up](https://signup.heroku.com/) to Heroku.

Then install the [Heroku Toolbelt](https://toolbelt.heroku.com/). It is a command line tool to manage your Heroku apps

After installing the Heroku Toolbelt, open a terminal and login to your account:
```bash
$ heroku login
Enter your Heroku credentials.
Email: newtonkiragu@herokudeploying.com
Password (typing will be hidden):
Authentication successful.
```

# Preparing the Application
In this tutorial I will be using a repo which is in development which is [ArtExtractKe](https://github.com/Benard18/ArtExtractKe)
It's an very simple  Django project, that shows various categories and their companies.
Its available on [github](https://github.com/Benard18/ArtExtractKe) so you can actually clone the repository and follow along or try it on your own existing django project.

## Assumptions
* Your familiar with the basics of django e.g concept of apps, settings, urls, basics of databases 
* You have django application that you want to deploy to heroku
* You are familiar with virtual environments - not a must but the knowledge would be a plus
* Your deployment db is postgres

We need to add the following to our project, we will cover each of them in detail in the below section

* Add a `Procfile` in the project root;
* Add `requirements.txt` file with all the requirements in the project root;
* Add `Gunicorn` to `requirements.txt`;
* A `runtime.txt` to specify the correct Python version in the project root;
* Configure `whitenoise` to serve static files.

## Procfile
Heroku apps include a `Procfile` that specifies the commands that are executed by the app’s dynos. 

For more information read on the [heroku documentation](https://devcenter.heroku.com/articles/procfile).

Create a file named `Procfile` in the project root with the following content:
```
web: gunicorn your_project_name.wsgi
```

## Runtime.txt
This file contains the python version you are using for heroku to use, create `runtime.txt` in your project root and add your python version in the following format
```
python-3.6.6
```
## Django-Heroku
We then install Django-Heroku which will come with the necessary requirements that will help us in deployment such as whitenoise,decouple and such but this will not satisfy all instalations.

```bash
pip install django-heroku && pip freeze > requirements.txt
```
PLEASE after doing this remove the pkg-resources bug so that you may deploy fluidly.

List of [Heroku Python Runtimes](https://devcenter.heroku.com/articles/python-runtimes).

## Whitenoise: Django Static Files settings
Lets first configure static related parameter in `settings.py`
```python 
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.9/howto/static-files/
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = '/static/'

# Extra places for collectstatic to find static files.
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```
Turns out django does not support serving static files in production. However, WhiteNoise project can integrate into your Django application, and was designed with exactly this purpose in mind.

Lets first install Whitenoise   `pip install whitenoise`

It is not a must to edit the wsgi.py file because there is an update available so let's just ignore it.

Next, install `WhiteNoise` into your Django application. This is done in `settings.py’s middleware section` (at the top):
```python 
MIDDLEWARE_CLASSES = (
    # Simplified static file serving.
    # https://warehouse.python.org/project/whitenoise/
    'whitenoise.middleware.WhiteNoiseMiddleware',
    ...
```
Add the following setting to `settings.py` in the static files section to enable gzip functionality.
```python
# Simplified static file serving.
# https://warehouse.python.org/project/whitenoise/

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# configuring the location for media
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Configure Django App for Heroku.
django_heroku.settings(locals())
```
## Requirements.txt
If your under a virtual environment run the command below to generate the requirements.txt file which heroku will use to install python package dependencies 

`pip freeze > requirements.txt`
Make sure you have the following packages if not install the using pip then run the command above again
```
config==0.3.9
dj-database-url==0.5.0
Django==1.11
django-bootstrap3==10.0.1
django-heroku==0.3.1
gunicorn==19.9.0
Pillow==5.2.0
psycopg2==2.7.5
python-decouple==3.1
pytz==2018.5
whitenoise==3.3.1
```
If you are following along with the mtribune app you should use the provided `requirements.txt` as you need to install more python packages, for any app just make sure you have the above packages as a plus.(Do not forget the pkg bug..remove it)

## Optional but very helpfull settings
### python-decouple and dj-database-url
Python Decouple is a must have app if you are developing with Django. It’s important to keep your application credentials like API Keys, Amazon S3, email parameters, database parameters safe, specially if it’s an open source repository. Also no more development_settings.py and production_settings.py, use just one settings.py for your whole project.

Install it via `pip install python-decouple`

dj-database-url is a simple Django utility allows you to utilize the 12factor inspired `DATABASE_URL` environment variable to configure your Django application.

Install it via `pip install dj-database-url`

Firts create a `.env` file and add it to `.gitignore` so you don’t commit any sensitive data to your remote repository.
below is an example of configurations you can add to the `.env` file.

```python
#just an example, dont share your .env settings
SECRET_KEY='342s(s(!hsjd998sde8$=o4$3m!(o+kce2^97kp6#ujhi'
DEBUG=True
DB_NAME='tribune'
DB_USER='user'
DB_PASSWORD='password'
DB_HOST='127.0.0.1'
MODE='dev'
ALLOWED_HOSTS='<app name in heroku>.herokuapp.com'
DISABLE_COLLECTSTATIC=1
```

We then edit `settings.py` to enable decouple to use the `.env` configurations.

 ```python
import os
import django_heroku
import dj_database_url
from decouple import config,Csv

MODE=config("MODE", default="dev")
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
 # development
if config('MODE')=="dev":
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST'),
            'PORT': '',
        }
        
    }
# production
else:
    DATABASES = {
        'default': dj_database_url.config(
            default=config('DATABASE_URL')
        )
    }

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
```

# Lets deploy now
First make sure you are in the root directory of the repository you want to deploy

Next create the heroku app
```bash
heroku create <your-app>
```

Create a postgres addon to your heroku app
```bash
heroku addons:create heroku-postgresql:hobby-dev
```
Next we log in to [Heroku dashboard](https://dashboard.heroku.com) to access our app and configure it

![Imgur](https://i.imgur.com/dbDQxlJ.png)

Click on the Settings menu and then on the button Reveal Config Vars:
Next add all the environment vaiables, by default you should have `DATABASE_URI` configuration created after installing postgres to heroku.

Alternatively you can add all your configurations in `.env` file directly to heroku by running the this command.

```bash
heroku config:set $(cat .env | sed '/^$/d; /#[[:print:]]*$/d')
```
Remember to first set `DEBUG` to false and confirm that you have added all the confuguration variables needed.

![Imgur](https://i.imgur.com/D0s6BkV.png?1)

### pushing to heroku

confirm that your application is running as expected before pushing, runtime errors will cause deployment to fail so make sure you have no bugs, you have all the following `Procfile`, `requirements.txt` with all required packages and  `runtime.txt` .

```bash
$ git push heroku master
```

if you find an error where the heroku git initialization has turned out unsuccessful; run the following command
```
bash
$ heroku git:remote -a <application-name>
```
If you did everything correctly then the deployment should be done after a while with an output like this

```bash
Enumerating objects: 94, done.
Counting objects: 100% (94/94), done.
Delta compression using up to 8 threads.
Compressing objects: 100% (84/84), done.
Writing objects: 100% (94/94), 3.35 MiB | 630.00 KiB/s, done.
Total 94 (delta 24), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Python app detected
remote: -----> Installing python-3.6.6
remote: -----> Installing pip
remote: -----> Installing requirements with pip
remote:        Collecting config==0.3.9 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 1))
remote:          Downloading https://files.pythonhosted.org/packages/0a/46/186ac016f3175211ec9bb4208579bc6dc9dd7dc882790d9f281533b83b0f/config-0.3.9.tar.gz
remote:        Collecting dj-database-url==0.5.0 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 2))
remote:          Downloading https://files.pythonhosted.org/packages/d4/a6/4b8578c1848690d0c307c7c0596af2077536c9ef2a04d42b00fabaa7e49d/dj_database_url-0.5.0-py2.py3-none-any.whl
remote:        Collecting Django==1.11 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 3))
remote:          Downloading https://files.pythonhosted.org/packages/47/a6/078ebcbd49b19e22fd560a2348cfc5cec9e5dcfe3c4fad8e64c9865135bb/Django-1.11-py2.py3-none-any.whl (6.9MB)
remote:        Collecting django-bootstrap3==10.0.1 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 4))
remote:          Downloading https://files.pythonhosted.org/packages/18/a8/f12d8491155c7f237084b883b8600faf722e3a46e54f17a25103b0fb9641/django-bootstrap3-10.0.1.tar.gz (40kB)
remote:        Collecting django-heroku==0.3.1 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 5))
remote:          Downloading https://files.pythonhosted.org/packages/59/af/5475a876c5addd5a3494db47d9f7be93cc14d3a7603542b194572791b6c6/django_heroku-0.3.1-py2.py3-none-any.whl
remote:        Collecting gunicorn==19.9.0 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 6))
remote:          Downloading https://files.pythonhosted.org/packages/8c/da/b8dd8deb741bff556db53902d4706774c8e1e67265f69528c14c003644e6/gunicorn-19.9.0-py2.py3-none-any.whl (112kB)
remote:        Collecting Pillow==5.2.0 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 7))
remote:          Downloading https://files.pythonhosted.org/packages/d1/24/f53ff6b61b3d728b90934bddb4f03f8ab584a7f49299bf3bde56e2952612/Pillow-5.2.0-cp36-cp36m-manylinux1_x86_64.whl (2.0MB)
remote:        Collecting psycopg2==2.7.5 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 8))
remote:          Downloading https://files.pythonhosted.org/packages/5e/d0/9e2b3ed43001ebed45caf56d5bb9d44ed3ebd68e12b87845bfa7bcd46250/psycopg2-2.7.5-cp36-cp36m-manylinux1_x86_64.whl (2.7MB)
remote:        Collecting python-decouple==3.1 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 9))
remote:          Downloading https://files.pythonhosted.org/packages/9b/99/ddfbb6362af4ee239a012716b1371aa6d316ff1b9db705bfb182fbc4780f/python-decouple-3.1.tar.gz
remote:        Collecting pytz==2018.5 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 10))
remote:          Downloading https://files.pythonhosted.org/packages/30/4e/27c34b62430286c6d59177a0842ed90dc789ce5d1ed740887653b898779a/pytz-2018.5-py2.py3-none-any.whl (510kB)
remote:        Collecting whitenoise==3.3.1 (from -r /tmp/build_19aebf8f25d534a39e73b13219af9927/requirements.txt (line 11))
remote:          Downloading https://files.pythonhosted.org/packages/0c/58/0f309a821b9161d0e3a73336a187d1541c2127aff7fdf3bf7293f9979d1d/whitenoise-3.3.1-py2.py3-none-any.whl
remote:        Installing collected packages: config, dj-database-url, pytz, Django, django-bootstrap3, whitenoise, psycopg2, django-heroku, gunicorn, Pillow, python-decouple
remote:          Running setup.py install for config: started
remote:            Running setup.py install for config: finished with status 'done'
remote:          Running setup.py install for django-bootstrap3: started
remote:            Running setup.py install for django-bootstrap3: finished with status 'done'
remote:          Running setup.py install for python-decouple: started
remote:            Running setup.py install for python-decouple: finished with status 'done'
remote:        Successfully installed Django-1.11 Pillow-5.2.0 config-0.3.9 dj-database-url-0.5.0 django-bootstrap3-10.0.1 django-heroku-0.3.1 gunicorn-19.9.0 psycopg2-2.7.5 python-decouple-3.1 pytz-2018.5 whitenoise-3.3.1
remote: 
remote: -----> Discovering process types
remote: 
remote: -----> Compressing...
remote:        Done: 56.3M
remote: -----> Launching...
remote:        Released v6
remote:        https://mtr1bune.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/mtr1bune.git
 * [new branch]      master -> master

 ```

 ### Run migrations
 ```bash
 $ heroku run python manage.py migrate
```

If you instead wish to push your postgres database data to heroku then run
```bash
$ heroku pg:reset
$ heroku pg:push <The name of the db in the local psql> DATABASE_URL --app <heroku-app>
```
You can the open the app in your browse.

# Comment
This process was a lot and you can easily mess up as I did, I suggest analyzing the part where you went wrong and going back to read on what you are supposed to do. I also highly recommend going through official documentations about deploying python projects to heroku as you will get a lot information that can help you debug effectively. I will provide some links in the resources section.

Remember heroku does not offer support for media files in the free tier subscription so find some where else to store those e.g Amazon s3.

# Resources
## heroku Docs
* https://devcenter.heroku.com/articles/heroku-postgresql
* https://devcenter.heroku.com/articles/django-assets
* https://devcenter.heroku.com/articles/python-runtimes
* https://devcenter.heroku.com/articles/procfile
* https://devcenter.heroku.com/articles/getting-started-with-python#introduction
* https://devcenter.heroku.com/articles/deploying-python
* https://devcenter.heroku.com/articles/django-app-configuration
* https://devcenter.heroku.com/articles/python-gunicorn

## very helpfull articles
* https://simpleisbetterthancomplex.com/tutorial/2016/08/09/how-to-deploy-django-applications-on-heroku.html
* https://simpleisbetterthancomplex.com/2015/11/26/package-of-the-week-python-decouple.html



