<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

## Features

This marketplace is a full-featured e-commerce package:

* Multi vendor, multi channel and multi warehouse
* From one to 1,000,000,000+ items
* Extremly fast down to 20ms
* For multi-tentant e-commerce SaaS solutions with unlimited vendors
* Bundles, vouchers, virtual, configurable, custom and event products
* Subscriptions with recurring payments
* 100+ payment gateways
* Full RTL support (frontend and backend)
* Block/tier pricing out of the box
* Extension for customer/group based prices
* Discount and voucher support
* Flexible basket rule system
* Full-featured admin backend
* Beautiful admin dashboard
* Configurable product data sets
* JSON REST API based on jsonapi.org
* GraphQL API for administration
* Completly modular structure
* Extremely configurable and extensible
* Extension for market places with millions of vendors
* Fully SEO optimized including rich snippets
* Translated to 30+ languages
* AI-based text translation
* Optimized for smart phones and tablets
* Secure and reviewed implementation
* High quality source code

Check out the demos:

* [Frontend demo](https://laravel.demo.aimeos.org)
* [Admin demo](https://admin.demo.aimeos.org)


## Installation

The Aimeos Laravel online shop package is a composer based library. It can be
installed easiest by using [Composer 2.1+](https://getcomposer.org) in the root
directory of your existing Laravel application:

```
wget https://getcomposer.org/download/latest-stable/composer.phar -O composer
php composer require aimeos/aimeos-laravel:~2023.04
```

Then, add these lines to the composer.json of the **Laravel skeleton application**:

```json
    "prefer-stable": true,
    "minimum-stability": "dev",
    "require": {
        "aimeos/aimeos-laravel": "~2023.04",
        ...
    },
    "scripts": {
        "post-update-cmd": [
            "@php artisan migrate",
            "@php artisan vendor:publish --tag=public --force",
            "\\Aimeos\\Shop\\Composer::join"
        ],
        ...
    }
```

Afterwards, install the Aimeos shop package using

`composer update`

In the last step you must now execute these artisan commands to get a working
or updated Aimeos installation:

```bash
php artisan vendor:publish --provider="Aimeos\Shop\ShopServiceProvider"
php artisan migrate
php artisan aimeos:setup --option=setup/default/demo:1
```

In a production environment or if you don't want that the demo data gets
installed, leave out the `--option=setup/default/demo:1` option.

## Authentication

You have to set up one of Laravel's authentication starter kits. Laravel Breeze
is the easiest one but you can also use Jetstream.

### Laravel 9 & 10

```bash
composer require laravel/breeze
php artisan breeze:install
npm install && npm run build
```

Laravel Breeze adds a route for `/profile` to `./routes/web.php` which may overwrite the
`aimeos_shop_account` route. To avoid an exception about a missing `aimeos_shop_account`
route, change the URL for these lines from `./routes/web.php` file from `/profile` to
`/profile/me`:

```php
Route::middleware('auth')->group(function () {
    Route::get('/profile/me', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile/me', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile/me', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

For more information, please follow the Laravel documentation:
* [Laravel 10.x](https://laravel.com/docs/10.x/authentication)
* [Laravel 9.x](https://laravel.com/docs/9.x/authentication)

### Configure authentication

As a last step, you need to extend the `boot()` method of your
`App\Providers\AuthServiceProvider` class and add the lines to define how
authorization for "admin" is checked in `app/Providers/AuthServiceProvider.php`:

```php
    public function boot()
    {
        // Keep the lines before

        \Illuminate\Support\Facades\Gate::define('admin', function($user, $class, $roles) {
            if( isset( $user->superuser ) && $user->superuser ) {
                return true;
            }
            return app( '\Aimeos\Shop\Base\Support' )->checkUserGroup( $user, $roles );
        });
    }
```

### Create account

Test if your authentication setup works before you continue. Create an admin account
for your Laravel application so you will be able to log into the Aimeos admin interface:

```bash
php artisan aimeos:account --super <email>
```

The e-mail address is the user name for login and the account will work for the
frontend too. To protect the new account, the command will ask you for a password.
The same command can create limited accounts by using `--admin`, `--editor` or `--api`
instead of `--super` (access to everything).

## Setup

To reference images correctly, you have to adapt your `.env` file and set the `APP_URL`
to your real URL, e.g.

```
APP_URL=http://127.0.0.1:8000
```

**Caution:** Make sure, Laravel uses the `file` session driver in your `.env` file!
Otherwise, the shopping basket content won't get stored correctly!

```
SESSION_DRIVER=file
```

If your `./public` directory isn't writable by your web server, you have to create these
directories:

```
mkdir public/aimeos public/vendor
chmod 777 public/aimeos public/vendor
```

In a production environment, you should be more specific about the granted permissions!

## Test

Then, you should be able to call the catalog list page in your browser. For a
quick start, you can use the integrated web server. Simply execute this command
in the base directory of your application:

```
php artisan serve
```

### Frontend

Point your browser to the list page of the shop using:

http://127.0.0.1:8000/shop

**Note:** Integrating the Aimeos package adds some routes like `/shop` or `/admin` to your
Laravel installation but the **home page stays untouched!** If you want to add Aimeos to
the home page as well, replace the route for "/" in `./routes/web.php` by this line:

```php
Route::group(['middleware' => ['web']], function () {
    Route::get('/', '\Aimeos\Shop\Controller\CatalogController@homeAction')->name('aimeos_home');
});
```

For multi-vendor setups, read the article about [multiple shops](https://aimeos.org/docs/latest/laravel/customize/#multiple-shops).

This will display the Aimeos catalog home component on the home page you you get a
nice looking shop home page which will look like this:

[![Aimeos frontend](https://aimeos.org/fileadmin/aimeos.org/images/aimeos-frontend.jpg?2021.07)](http://127.0.0.1:8000/shop)

### Backend

If you've still started the internal PHP web server (`php artisan serve`)
you should now open this URL in your browser:

http://127.0.0.1:8000/admin

Enter the e-mail address and the password of the newly created user and press "Login".
If you don't get redirected to the admin interface (that depends on the authentication
code you've created according to the Laravel documentation), point your browser to the
`/admin` URL again.

**Caution:** Make sure that you aren't already logged in as a non-admin user! In this
case, login won't work because Laravel requires you to log out first.

[![Aimeos backend](https://aimeos.org/fileadmin/aimeos.org/images/aimeos-backend.png)](http://127.0.0.1:8000/admin)

## Hints

To simplify development, you should configure to use no content cache. You can
do this in the `config/shop.php` file of your Laravel application by adding
these lines at the bottom:

```php
    'madmin' => [
        'cache' => [
            'manager' => [
                'name' => 'None',
            ],
        ],
    ],
```

