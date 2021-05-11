## Step 1. Open Terminal

<details>
<summary>Click to expand!</summary>

<p>
    Open the SSH terminal on your machine and run the following command:

    ```bash
    ssh -p 22 <username>@<host_ip_address>
    ```

    Here 22 is a port number, it varies in hosting providers. For example in hostinger.com:

    ```bash
    ssh -p 65002 u443795110@235.24.119.53
    ```

</p>
</details>

## Step 2. Clone git repository outside the public_html directory

<details>
<summary>Click to expand!</summary>

<p>
    First, let say in your hosting server (VPS, or shared hosting ... whatever), you have current `public_html/` directory, which is accessible publicly via web domain, for example:

    ```bash
    /home/<username>/public_html
    ```

    Now, create a new directory, which contains all your application source code, at the same level as `public_html/`, for example:

    ```bash
    /home/<username>/project
    ```

    Use _git_ to transfer your code to this directory:

    ```bash
    git clone https://github.com/username/your-app project
    ```

</p>
</details>

## Step 3. Move the contents of public/ directory to public_html/ and delete public/ directory from project/

<details>
<summary>Click to expand!</summary>

<p>
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

    After public directory has been changed, we need to update the public_path() helper method. Otherwise this error will be shown:

    > The Mix manifest does not exist

    Add following 3 lines to `register()` method in `\App\Providers\AppServiceProvider`:

    ```php
    /**
    * Register any application services.
    *
    * @return void
    */
    public function register()
    {
        // ...

        $this->app->bind('path.public', function () {
            return base_path('../public_html');
        });
    }
    ```

</p>
</details>

## Step 4. Run composer install

By default /vendor folder is not included in laravel package, so we need to run:

```bash
composer install
```

Now laravel and its _dependencies_ are installed via composer.

## Step 5. Create MySQL database

Create a database in cpanel. Keep noted database name, user and password.

## Step 6. Update the Environment Configurations

Copy the example file to new `.env` file using `cp .env.example .env`

Now open the `.env` file and update the necessary configuration changes such as application name and database information.

```dosini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

### IMPORTANT!

Set `APP_DEBUG` to `false` and `APP_ENV` to `production`, and update the `APP_NAME` and `APP_URL` accordingly. If you leave `APP_DEBUG` as true, in the event of errors you’ll be displaying sensitive debug information to the world:

```dosini
APP_NAME="Project name"
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=http://your-app.com
```

### Update default engine to InnoDB

<details>
<summary>Click to expand!</summary>

<p>
    Update following line in `/config/database.php`:

    ```php
    'mysql' => [
        // ...
        'engine' => 'InnoDB ROW_FORMAT=DYNAMIC',
    ],
    ```

    > When a table is created with ROW_FORMAT=DYNAMIC, InnoDB can store long variable-length column values (for VARCHAR, VARBINARY, TEXT and BLOB types). Please visit [this link](https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html) for more details.

</p>
</details>

## Step 7. Application Key

The next thing you should do is set application key to a random string:

```bash
php artisan key:generate
```

## Step 8. Execute Database Migration and Seeder

Now it’s time to migrate our tables and seed database. Run:

```bash
php artisan migrate --seed
```

## Step 9. Create storage directory symlink

Run `pwd` for getting project's BASE_PATH. **In Linux** (Target &#8592; Link):

```bash
ln -s {BASE_PATH}/storage/app {BASE_PATH}/public/storage
```

**In Windows** (Link &#8594; Target):

```bash
mklink /D {BASE_PATH}/public/storage {BASE_PATH}/storage/app
```

or create with _php artisan_ **(This command not exist in Lumen)**:

```bash
php artisan storage:link
```

## Step 10. Give permission to storage and bootstrap directories

We need to set some folder permissions so they are writeable, specifically the `/storage/` and `/bootstrap/cache/` folders.

```bash
chmod -R o+w storage bootstrap/cache
```

## Step 11. Autoloader Optimization

When deploying to production, make sure that you are optimizing Composer's class autoloader map so Composer can quickly find the proper file to load for a given class:

```bash
composer install --optimize-autoloader --no-dev
```

Optimizing configuration, route and view loading:

```bash
php artisan config:cache

php artisan route:cache

php artisan view:cache
```

At last you optimize composer autoload files:

```bash
composer dump-autoload -o
```

## Step 12. Set up Laravel Scheduler

If you have any cron jobs, you should configure them now. In your project directory, run this command:

```bash
crontab -e
```

It will open up `/var/spool/cron/crontabs/` in text editor. Paste the following expression there:

```bash
* * * * * cd /var/www/laravel-app && php artisan schedule:run >> /dev/null 2>&1
```

It means to run `php artisan schedule:run` command every minute. For reference, use following editor for cron schedule expressions: [Crontab guru](https://crontab.guru)

# Laravel Performance Tips

### 1. Remove Unused Service

In the context of Laravel performance tuning, an important tip is not to load all services through the config. While you are there, always remember to disable unused services in the config files. Add comments to these service providers.

### 2. Classmap Optimization

Even a mid-level Laravel app has a number of files because Laravel has the habit of calling including multiple files for include requests. A simple trick is to declare all the files that would be included to include requests and combine them in a single file. Thus, for all include requests, a single file will be called and loaded. For this, use the following command:

```bash
php artisan optimize
```

#### For more details please visit

> [12 Tips for Laravel Performance Optimization](https://www.cloudways.com/blog/laravel-performance-optimization)
