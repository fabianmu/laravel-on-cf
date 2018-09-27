# Laravel on CF

A skeleton Laravel app that stages and runs on Cloud Foundry. Replicate the steps below or fork this repo to start your Laravel project.

## Initialize a Laravel project

After installing PHP and Composer, follow the [Laravel installation documentation](https://laravel.com/docs/5.7) and commit the results:

```
composer create-project --prefer-dist laravel/laravel laravel-on-cf
cd laravel-on-cf
git init
git add .
git commit -m "Initialized laravel with composer"
```

## Customize for Cloud Foundry

### PHP Buildpack configuration

Set the webdir, the composer vendor directory, and prefer nginx:

```
mkdir -p ./.bp-config
echo '{
  "WEB_SERVER": "nginx",
  "WEBDIR": "public",
  "COMPOSER_VENDOR_DIR": "vendor"
}' > ./.bp-config/options.json
```

### NGINX configuration
Override the PHP Buildpack's nginx server-locations configuration:

```
mkdir -p ./.bp-config/nginx
echo '       # Try the URI, else index.php
       location / {
           try_files $uri $uri/ /index.php?$query_string;
       }

       # index.php is the only PHP file that will be processed
       location ~ ^/index\.php(/|$) {
           include         fastcgi_params;
           fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
           fastcgi_pass    php_fpm;
       }

       # Some basic cache-control for static files to be sent to the browser
       location ~* \.(?:ico|css|js|gif|jpeg|jpg|png)$ {
           expires         max;
           add_header      Pragma public;
           add_header      Cache-Control "public, must-revalidate, proxy-revalidate";
       }

       # Deny hidden files (.htaccess, .htpasswd, .DS_Store).
       location ~ /\. {
           deny            all;
           access_log      off;
           log_not_found   off;
       }' > ./.bp-config/nginx/server-locations.conf
```

### Cloud Foundry Manifest

Finally, a __very__ basic Cloud Foundry manifest.

**IMPORTANT**: Every environment should utilize a unique `APP_KEY` for encryption. Generate your own using `./artisan key:generate --show`.

```
echo "---
applications:
  - name: laravel-on-cf
    buildpacks:
      - php_buildpack
    memory: 128M
    env:
      APP_ENV: production
      APP_DEBUG: false
      APP_KEY: $(./artisan key:generate --show)" > manifest.yml
```

### Commit and Push!

```
git add .
git commit -m "Customizations for CF"
cf push
```

## What next?
* Laravel likely requires a relational database
* Laravel seems to have some sort of integration with Node.js given there is a package.json

## Credits
* [Laravel](./LARAVEL.md)
* [laravel-on-bluemix](https://github.com/tarikgan/laravel-on-bluemix)
