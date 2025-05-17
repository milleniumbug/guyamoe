# Guya.moe
Generalized manga reading framework. Adapted for Kaguya-sama manga, but can be used generically for any and all manga.

Testing Supported By<br/>
<img width="160" src="http://foundation.zurb.com/sites/docs/assets/img/logos/browser-stack.svg" alt="BrowserStack"/>

⚠ **Note:** The install instructions below will not result in a general purpose CMS due to the amount of hardcoded assets in Guyamoe.

## Prerequisites 
- git
- python 3.6.5+
- pip
- virtualenv

## Install
1. Create a venv for Guyamoe in your home directory.
```
virtualenv -p python3 ~/guyamoe
```

2. Clone Guyamoe's source code into the venv.
```
git clone https://github.com/appu1232/guyamoe ~/guyamoe/app
```

3. Activate the venv.
```
cd ~/guyamoe/app && source ../bin/activate
```

4. Install Guyamoe's dependencies.
```
pip3 install -r requirements.txt
```

At this point you may get an error that pg_config is missing. [Install development version of PostgreSQL.](https://stackoverflow.com/questions/11618898/pg-config-executable-not-found)

5. Change the value of the `SECRET_KEY` variable to a randomly generated string.
```
sed -i "s|\"o kawaii koto\"|\"$(openssl rand -base64 32)\"|" guyamoe/settings/base.py
```

6. Upstream repo don't have migrations set up because something. It's possible it will be fixed later, in the meantime do this.
```
mkdir misc/migrations && touch misc/migrations/__init__.py
```

7. Generate the default assets for Guyamoe.
```
python3 init.py
```

8. Create an admin user for Guyamoe.
```
python3 manage.py createsuperuser
```

Before starting the server, create a `media` folder in the base directory. Add manga with the corresponding chapters and page images. Structure it like so:
```
media
└───manga
    └───<series-slug-name>
        └───001
            ├───001.jpg
            ├───002.jpg
            └───...
```
E.g. `Kaguya-Wants-To-Be-Confessed-To` for `<series-slug-name>`. 

**Note:** Zero pad chapter folder numbers like so: `001` for the Kaguya series (this is how the fixtures data for the series has it). It doesn't matter for pages though nor does it have to be .jpg. Only thing required for pages is that the ordering can be known from a simple numerical/alphabetical sort on the directory.

## Start the server
-  `python3 manage.py runserver` - keep this console active

Now the site should be accessible on localhost:8000

Django docs say this: [DO NOT USE THIS SERVER IN A PRODUCTION SETTING. It has not gone through security audits or performance tests.](https://docs.djangoproject.com/en/3.1/ref/django-admin/). Below is the section on my attempt on making it work good enough for production, on Debian.

## Other info
Relevant URLs (as of now): 

- `/` - home page
- `/about` - about page
- `/admin` - admin view (login with created user above)
- `/admin_home` - admin endpoint for clearing the site's cache
- `/reader/series/<series_slug_name>` - series info and all chapter links
- `/reader/series/<series_slug_name>/<chapter_number>/<page_number>` - url scheme for reader opened on specfied page of chapter of series.
- `/api/series/<series_slug_name>` - all series data requested by reader frontend
- `/media/manga/<series_slug_name>/<chapter_number>/<page_file_name>` - url scheme to used by reader to actual page as an image.

# Debian setup

The `nginx` directory contains systemd module, the script and nginx config. Nginx is a web server that will communicate with guyamoe Django app through uWSGI.
These instructions make a lot of assumptions. Adjust these instructions and configuration to your own needs.

Install uWSGI

```
pip3 install uwsgi
```

Add the user that will be running the app to www-data

```
sudo usermod -a -G www-data milleniumbug
```

Set the correct permissions

```
chmod +x nginx/start.sh
sudo chown milleniumbug:www-data nginx/socket
sudo chmod g+sw nginx/socket
sudo chown milleniumbug:www-data -R media/manga
sudo chmod g+s -R media/manga
```

Copy the config to appropriate places

```
sudo cp nginx/guyamoe.service /etc/systemd/system
sudo cp nginx/guya-site-nginx /etc/nginx/sites-available/guyamoe
sudo ln -s /etc/nginx/sites-available/guyamoe /etc/nginx/sites-enabled/guyamoe
```

Use certbot to set up TLS certificate for your own domain
