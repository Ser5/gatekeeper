# Installation & Configuration

## Installation

The best way to install Gatekeeper is through Composer (though I suppose you could clone the repo if you want to do it the hard way). Use the following command to add it to your current project:

```
composer require psecio/gatekeeper
```

or, if you want to edit the `composer.json` yourself and get specific with the version:

```
{
    "require": {
        "psecio/gatekeeper": "1.0"
    }
}
```

## Dependencies

*Gatekeeper* makes use of several other PHP dependencies to help reduce code duplication:

- [Modler](http://github.com/enygma/modler)
- [Phinx](http://github.com/robmorgan/phinx)
- [password_compat](http://github.com/ircmaxell/password-compat)
- [phpdotenv](http://github.com/vlucas/phpdotenv)

## Setup

There's a few things you'll need to do before using the system. First off, it uses [Phinx](https://phinx.org) for working
with the database tables (creation and all). Second it uses the handy [phpdotenv](https://github.com/vlucas/phpdotenv) library
to handle the loading of environment values. This helps you understand the installation and usage a bit better.

1. Create a database named `gatekeeper` and create a user for it (this is configured later)

```
create database gatekeeper;
grant all on gatekeeper.* to 'gk-user'@'localhost' identified by 'some-password-here';
flush privileges;
```

1. Copy the `.env.dist` file from the *Gatekeeper* vendor directory to the (non-`DOCUMENT_ROOT`) place of your choosing and rename it to `.env`.
2. Copy the `phinx.dist.xml` file to the place of your choosing (again, non-`DOCUMENT_ROOT`) and rename it to `phinx.yml`.

```
cp vendor/psecio/gatekeeper/.env.dist /tmp/someplace/.env
cp vendor/psecio/gatekeeper/phinx.dist.yml /tmp/someplace/phinx.yml
```

3. Open both of these files and update the **connection information** to match the database you created earlier.
4. Now we run the migrations:

```
vendor/bin/phinx migrate -c /tmp/somplace/phinx.yml
```

Where the `/tmp/someplace/phinx.yml` is the path to where you put your `phinx.yml` file (again, not in the document root **please!**).

If you get errors here, be sure the connection information is correct.

5. You're ready to go!

You can now start using the *Gatekeeper* functionality in your application. You only need to call the `init` function to set
up the connection and get the instance configured:

```php
<?php
require_once 'vendor/autoload.php';
Gatekeeper::init();
?>
```

## Configuration options

You can pass in options to the `init` call if you don't necesarily want to use the `.env` configuration file handling. There's a few options:

```php
<?php
// You can specify your own .env path
Gatekeeper::init('/path/to/directory');

// You can also force the use of your own database configuration
$config = array(
    'type' => 'mysql',
    'username' => 'gatekeeper-user',
    'password' => 'gatekeeper-pass',
    'name' => 'gatekeeper',
    'host' => 'gatekeeper-db.localhost'
);
Gatekeeper::init(null, $config);
?>
```

## Throttling

By default Gatekeeper will have throttling enabled. This means that, on incorrect login by a user, the information will be logged. When they hit a threshold (defaults to 5) in a certain amount of time (default of 1 minute) they'll be marked as blocked and will not be allowed to log in.

You can disable this feature in one of two ways:

```php
<?php
// Either throught the init call
Gatekeeper::init(null, array('throttle' => false));

// Or through a method call
Gatekeeper::disableThrottle();

// And to reenable
Gatekeeper::enableThrottle();
?>
```

You can also check the status of the throttling:

```php
<?php
if (Gatekeeper::throttleStatus() === true) {
    echo 'Throttling is enabled!';
}
?>
```

