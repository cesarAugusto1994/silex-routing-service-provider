# RoutingServiceProvider #

-----

[![Build Status](https://travis-ci.org/marcojanssen/silex-routing-service-provider.png?branch=master)](https://travis-ci.org/marcojanssen/silex-routing-service-provider)
[![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/marcojanssen/silex-routing-service-provider/badges/quality-score.png?s=ee8a98ec16a263e96f27ccf6be68db3d434d1156)](https://scrutinizer-ci.com/g/marcojanssen/silex-routing-service-provider/)
[![Code Coverage](https://scrutinizer-ci.com/g/marcojanssen/silex-routing-service-provider/badges/coverage.png?s=c0ad7b2616ce7c0b5e472457d7ec49063f86f527)](https://scrutinizer-ci.com/g/marcojanssen/silex-routing-service-provider/)

**RoutingServiceProvider** is a silex provider for easily adding routes

## Features ##

- Register providers through configuration
- Register multiple providers with the provider
- Register a single provider with the provider

## Installing

- Install [Composer](http://getcomposer.org)

- Add `marcojanssen/silex-routing-service-provider` to your `composer.json`:

```json
{
    "require": {
        "marcojanssen/silex-routing-service-provider": "1.2.*"
    }
}
```

- Install/update your dependencies

## Options

Each route is required to have the following parameters:
* pattern (string) 
* controller (string)
* method - get, put, post, delete, options, head (array)

Optionally, you can set a route name (for [named routes](http://silex.sensiolabs.org/doc/usage.html#named-routes)). The key of the $route-array will be used as the route name or you can set it like this:

```php
$routes = array(
    'foo' => array(
        //'routeName' => 'foo', --> you can omit the routeName if a key is set
        'pattern' => '/foo',
        'controller' => 'Foo\Controller\FooController::fooAction',
        'method' => array('get', 'post', 'put', 'delete', 'options', 'head')
    )
);

```

Optionally the following parameters can also be added:

* value (array)

``` php
$value = array('name' => 'value')
```

* assert (array)

``` php
$assert = array('id' => '^[\d]+$')
```

* before (array)

``` php
$before = array('before' => function() {})
```

* after (array)

``` php
$after = array('after' => function() {})
```

## Usage

### Adding a single route

`index.php`
```php

use Silex\Application;
use MJanssen\Provider\RoutingServiceProvider;

$app = new Application();
$routingServiceProvider = new RoutingServiceProvider();

$route = array(
    'routeName' => 'foo',
    'pattern' => '/foo',
    'controller' => 'Foo\Controller\FooController::fooAction',
    'method' => array('get', 'post', 'put', 'delete', 'options', 'head')
);

$routingServiceProvider->addRoute($app, $route);

```

### Adding multiple routes

`index.php`
```php

use Silex\Application;
use MJanssen\Provider\RoutingServiceProvider;

$app = new Application();
$routingServiceProvider = new RoutingServiceProvider();

$routes = array(
    'foo' => array(
        //'routeName' => 'foo', --> you can omit the routeName if a key is set
        'pattern' => '/foo',
        'controller' => 'Foo\Controller\FooController::fooAction',
        'method' => array('get', 'post', 'put', 'delete', 'options', 'head')
    ),
    'baz' => array(
        //'routeName' => 'baz', --> you can omit the routeName if a key is set
        'pattern' => '/baz',
        'controller' => 'Baz\Controller\BazController::bazAction',
        'method' => array('get', 'post', 'put', 'delete', 'options', 'head')
    )
);

$routingServiceProvider->addRoutes($app, $route);

```
### Adding before/after middleware
To add controller middleware you can use the 'after' and 'before' key of the route configuration. The 'before' key is used to run the middleware code before the controller logic is executed, after execution of the controller logic.
Below is an example using a middleware class and how to configure this in the route config. Instead of using a middleware class you can also use a regular callback.

**Note** Be aware that currently there is only support for php.

#### Example middleware class:

```php
class MiddleWare {

    public function __invoke(Request $request, Application $app)
    {
        //do stuff
        $x = 1;
    }
}
```

#### Using the middleware class in the route configuration


`index.php`
```php
use Silex\Application;
use MJanssen\Provider\RoutingServiceProvider;

$app = new Application();
$routingServiceProvider = new RoutingServiceProvider();

$routes = array(
    'foo' => array(
        //'routeName' => 'foo', --> you can omit the routeName if a key is set
        'pattern' => '/foo',
        'controller' => 'Foo\Controller\FooController::fooAction',
        'method' => array('get'),
        // this is where it all happens!
        'before' => array(new MiddleWare())
    )
);
$routingServiceProvider->addRoutes($app, $route);
```
### Registering providers with configuration

For this example the [ConfigServiceProvider](https://github.com/igorw/ConfigServiceProvider) is used to read the yml file. The RoutingServiceProvider picks the stored configuration through the node `config.routing` as in `$app['config.routing']` by default. If you want to set a different key, add it as parameter when instantiating the RoutingServiceProvider
***Note: The key of the array will also be used as the routeName, so you can omit routeName.

`routes.yaml`

```yaml
config.routes:
    home:
        routeName: 'home'
        pattern: /
        method: [ 'get', 'post' ]
        controller: 'Foo\Controllers\FooController::getAction'
```

`routes.php`

```php

return array(
    'custom.routing.key' => array(
        array(
            'pattern' => '/foo/{id}',
            'controller' => 'Foo\Controllers\FooController::getAction',
            'method' => array(
                'get'
            ),
            'assert' => array(
                'id' => '^[\d]+$'
            ),
            'value' => array(
                'value1' => 'foo',
                'value2' => 'baz'
            )
        )
    )
);

```

`index.php`
```php

use Silex\Application;
use Igorw\Silex\ConfigServiceProvider;
use MJanssen\Provider\RoutingServiceProvider;

$app = new Application();

//Set all routes
$app->register(
    new RoutingServiceProvider(__DIR__."/../app/config/routes.php")
);

//Add all routes
$app->register(new RoutingServiceProvider('custom.routing.key'));

```

**Note**: It's recommended to use php instead of yml/xml/etc, because it can be opcached. Otherwise you have to implement a caching mechanism yourself.

## Todo

convert, there is no option set this per route at the moment
before & after middleware still need to be implemented using xml and yml. (if possible)
