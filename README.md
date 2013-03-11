LexikMonologBrowserBundle
==========================

This Symfony2 bundle integrates a [Doctrine DBAL](https://github.com/doctrine/dbal) handler for [Monolog](https://github.com/Seldaek/monolog) and a web UI to display log entries. You can list, filter and paginate logs as you can see on the screenshot bellow:

![Log entries listing](https://github.com/lexik/LexikMonologBrowserBundle/raw/master/Resources/screen/list.jpg)

As this bundle execute database query on each raise log, it's relevant to small and medium projects, if you have billion of logs consider use a specific log server like [sentry](http://getsentry.com/), [airbrake](https://airbrake.io/), etc.

Requirements:
------------

* Symfony 2.1+
* KnpLabs/KnpPaginatorBundle

Installation
------------

Installation with composer:

``` json
    ...
    "require": {
        ...
        "lexik/monolog-browser-bundle": "dev-master",
        ...
    },
    ...
```

Next, be sure to enable this bundles in your `app/AppKernel.php` file:

``` php
public function registerBundles()
{
    return array(
        // ...
        new Knp\Bundle\PaginatorBundle\KnpPaginatorBundle(),
        new Lexik\Bundle\MonologBrowserBundle\LexikMonologBrowserBundle(),
        // ...
    );
}
```

Configuration
-------------

First of all, you need to configure the Doctrine DBAL connection to use in the handler. You have 2 ways to do that:

**By using an existing Doctrine connection:**

``` yaml
# app/config/config.yml
doctrine:
    dbal:
        connections:
            default:
                ...
            monolog:
                driver:   pdo_sqlite
                dbname:   monolog
                path:     %kernel.root_dir%/cache/monolog2.db
                charset:  UTF8

lexik_monolog_browser:
    doctrine:
        connection_name: monolog
```

**By creating a custom Doctrine connection for the bundle:**

``` yaml
# app/config/config.yml
lexik_monolog_browser:
    doctrine:
        connection:
            driver:      pdo_sqlite
            driverClass: ~
            pdo:         ~
            dbname:      monolog
            host:        localhost
            port:        ~
            user:        root
            password:    ~
            charset:     UTF8
            path:        %kernel.root_dir%/db/monolog.db # The filesystem path to the database file for SQLite
            memory:      ~                               # True if the SQLite database should be in-memory (non-persistent)
            unix_socket: ~                               # The unix socket to use for MySQL
```

Please refer to the [Doctrine DBAL connection configuration](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html#configuration) for more details.

Optionally you can override schema table name (`monolog_entries` by default):

``` yaml
# app/config/config.yml
lexik_monolog_browser:
    doctrine:
        table_name: monolog_entries
```

Now your database is configured, you can generate schema of your log entry table via following command:

```
./app/console lexik:monolog-browser:schema-create
# you should see as result:
# Created table monolog_entries for Doctrine Monolog connection
```

Next, you can configure Monolog to use the Doctrine DBAL handler:

``` yaml
# app/config/config_prod.yml # or any env
monolog:
    handlers:
        main:
            type:         fingers_crossed # or buffer
            level:        error
            handler:      lexik_monolog_browser
        app:
            type:         buffer
            action_level: info
            channels:     app
            handler:      lexik_monolog_browser
        deprecation:
            type:         buffer
            action_level: warning
            channels:     deprecation
            handler:      lexik_monolog_browser
        lexik_monolog_browser:
            type:         service
            id:           lexik_monolog_browser.handler.doctrine_dbal
```

Now you have enabled and configured the handler, you should want to display log entries so just import the routing files:

``` yaml
# app/config/routing.yml
lexik_monolog_browser:
    resource: "@LexikMonologBrowserBundle/Resources/config/routing.xml"
    prefix:   /admin/monolog
```

Translations
------------

If you wish to use default texts provided in this bundle, you have to make sure you have translator enabled in your config:

``` yaml
# app/config/config.yml
framework:
    translator: ~
```

Overriding default layout
-------------------------

You can override default layout of the bundle through configure `base_layout`:

``` yaml
# app/config/config.yml
lexik_monolog_browser:
    base_layout: "::LexikMonologBrowserBundle.html.twig"
```

or quite simply with the Symfony way by create a template on `app/Resources/LexikMonologBrowserBundle/views/layout.html.twig`.

ToDo
----

* configure Processors to push into the Handler
* abstract handler and connector for Doctrine and browse another like Elasticsearh
* write Tests
