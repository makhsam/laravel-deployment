## Step 1. Open Terminal

```bash
ssh -p 22 bprouz@188.130.162.10
```

## Step 2. Clone git repository outside the public_html directory

First, let say in your hosting server (VPS, or shared hosting ... whatever), you have current public_html/ directory, which is accessible publicly via web domain, for example:

```
/home/<username>/public_html
```

Now, create a new directory, which contains all your application source code, at the same level as public_html/, for example:

```
/home/<username>/project
```

Use *git* to transfer your code to this directory:

```
git clone https://github.com/username/your-app project
```

## Step 3. Move the contents of public/ directory to public_html/ and delete public/ directory from project/

Then, navigate to the public_html/ folder and locate the index.php file. Find the following lines:

```php
require __DIR__.'/../vendor/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
```

And update them to the correct paths as following:

```php
require __DIR__.'/../project/vendor/autoload.php';
$app = require_once __DIR__.'/../project/bootstrap/app.php';
```

## Step 4. Run composer install

By default /vendor folder is not included in laravel package, so we need to run:

```
composer install
```

Now laravel and its *dependencies* are installed via composer.

## Step 5. Create MySQL database

Create a database in cpanel. Keep noted database name, user and password.

## Step 6. Update the Environment Configurations

Copy the example file to new `.env` file using `cp .env.example .env`

Now open the `.env` file and update the necessary configuration changes such as application name and database information.

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

### IMPORTANT!

Set `APP_DEBUG` to `false` and `APP_ENV` to `production`, and update the `APP_NAME` and `APP_URL` accordingly. If you leave `APP_DEBUG` as true, in the event of errors you’ll be displaying sensitive debug information to the world:

```
APP_NAME="Project name"
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://your-app.com
```

## Step 7. Application Key

The next thing you should do is set application key to a random string:

```
php artisan key:generate
```


## Step 8. Execute Database Migration and Seeder

Now it’s time to migrate our tables and seed database. Run:

```
php artisan migrate --seed
```

## Step 9. Create storage directory symlink

```
ln -s /home/<username>/project/storage/app/public/ /home/<username>/public_html/storage
```

or create with *php artisan* (this will not work as public/ folder outside of project directory):

```
php artisan storage:link
```

## Step 10. Give permission to storage and bootstrap directories

We need to set some folder permissions so they are writeable, specifically the `/storage/` and `/bootstrap/cache/` folders.

```
chmod -R o+w storage bootstrap/cache
```

## Step 11. Autoloader Optimization

When deploying to production, make sure that you are optimizing Composer's class autoloader map so Composer can quickly find the proper file to load for a given class:

```
composer install --optimize-autoloader --no-dev
```

### Optimizing Configuration Loading

```
php artisan config:cache
```

### Optimizing Route Loading

```
php artisan route:cache
```

### Optimizing View Loading

```
php artisan view:cache
```

At last you optimize composer autoload files:

```
composer dump-autoload -o
```

## Step 12. Set up Laravel Scheduler

If you have any cron jobs, you should configure them now. Use following editor for cron schedule expressions: [Crontab guru](https://crontab.guru)


# Laravel Performance Tips

### 1. Remove Unused Service

In the context of Laravel performance tuning, an important tip is not to load all services through the config. While you are there, always remember to disable unused services in the config files. Add comments to these service providers.

### 2. Classmap Optimization

Even a mid-level Laravel app has a number of files because Laravel has the habit of calling including multiple files for include requests. A simple trick is to declare all the files that would be included to include requests and combine them in a single file. Thus, for all include requests, a single file will be called and loaded. For this, use the following command:

```
php artisan optimize --force
```

> For more details please visit [12 Tips for Laravel Performance Optimization](https://www.cloudways.com/blog/laravel-performance-optimization)