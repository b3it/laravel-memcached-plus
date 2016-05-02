# laravel-memcached-plus
[![StyleCI](https://styleci.io/repos/31662516/shield?style=flat)](https://styleci.io/repos/31662516)
[![Build Status](https://travis-ci.org/b3it/laravel-memcached-plus.svg?branch=master)](https://travis-ci.org/b3it/laravel-memcached-plus)
[![Total Downloads](https://poser.pugx.org/b3it/laravel-memcached-plus/downloads)](https://packagist.org/packages/b3it/laravel-memcached-plus)
[![Latest Stable Version](https://poser.pugx.org/b3it/laravel-memcached-plus/v/stable)](https://packagist.org/packages/b3it/laravel-memcached-plus)
[![License](https://poser.pugx.org/b3it/laravel-memcached-plus/license)](https://packagist.org/packages/b3it/laravel-memcached-plus)

## Summary

_This package is useful for Laravel 5.0 - 5.2. From 5.3 onwards two PRs I submitted to `laravel/framework` have been merged, therefore the features of this package are available out-the-box with Laravel 5.3 onwards._

Integrating with cloud memcached services such as [MemCachier](https://www.memcachier.com/) and
[memcached cloud](https://redislabs.com/memcached-cloud) can require memcached features not available
with the built-in [Laravel Cache](http://laravel.com/docs/5.2/cache) memcached driver from version 5.0 to 5.2.

These include:

* SASL authentication 
* custom options
* persistent connections

Adding 3 new configuration items, this package _enhances_ the built-in Laravel 5 Cache memcached driver.
Optionally, this package also allows these extra configuration items to be used for memcached
Sessions.

Read on for detailed instructions - you may find it useful to reference the
[demo app](https://github.com/b3it/laravel-memcached-plus-demo) at the same time.

## Requirements

* >= PHP 5.4 with [ext-memcached](http://php.net/manual/en/book.memcached.php)
* To use [SASL](http://docs.php.net/manual/en/memcached.setsaslauthdata.php) it must be compiled with
SASL support. This is the default on [Heroku](https://devcenter.heroku.com/articles/php-support)

## Installation

Available to install as a Composer package on
[Packagist](https://packagist.org/packages/b3it/laravel-memcached-plus), all you need to do is:

`composer require b3it/laravel-memcached-plus`

If your local environment does not meet the requirements you may need to append the
`ignore-platform-reqs` option:

`composer require b3it/laravel-memcached-plus --ignore-platform-reqs`

## Configuration

Once installed you can use this package to enhance the Laravel
[Cache](http://laravel.com/docs/5.0/cache) and [Session](http://laravel.com/docs/5.0/session)
services.

### Providers

This section discusses the Laravel application configuration file `app/config.php`.

In the `providers` array you need to replace following built-in Service Providers:

 * `Illuminate\Cache\CacheServiceProvider` and
 * `Illuminate\Session\SessionServiceProvider` (optional)

A recommended approach is to comment out the built-in providers and append the
Service Providers from this package:

```
'providers' => [
    ...
    //'Illuminate\Cache\CacheServiceProvider',
    ...
    //'Illuminate\Session\SessionServiceProvider',
    ...

    /*
     * Application Service Providers...
     */
     ...

    'B3IT\MemcachedPlus\CacheServiceProvider',
    'B3IT\MemcachedPlus\SessionServiceProvider', // optional
],
```

The `B3IT\MemcachedPlus\SessionServiceProvider` is optional. You only need to add this if:

* You want to specify a non-default memcached store to use for sessions, default is "memcached", or
* You want to use the memcached features provided by this package for sessions

### Cache

This section discusses the Laravel cache configuration file `config/cache.php`.

This package makes the following extra configuration items available for use with a memcached store:

* `persistent_id` - [`Memcached::__construct`] (http://php.net/manual/en/memcached.construct.php)
explains how this is used
* `sasl` - used by [`Memcached::setSaslAuthData`](http://php.net/manual/en/memcached.setsaslauthdata.php)
* `options` - see [`Memcached::setOptions`](http://php.net/manual/en/memcached.setoptions.php)

These may be used in a store config like so:

```
'stores' => [
    'memcachedstorefoo' => [
        'driver'  => 'memcached',
        'persistent_id' => 'laravel',
        'sasl'       => [
            env('MEMCACHIER_USERNAME'),
            env('MEMCACHIER_PASSWORD')
        ],
        'options'    => [
            Memcached::OPT_NO_BLOCK         => true,
            Memcached::OPT_AUTO_EJECT_HOSTS => true,
            Memcached::OPT_CONNECT_TIMEOUT  => 2000,
            Memcached::OPT_POLL_TIMEOUT     => 2000,
            Memcached::OPT_RETRY_TIMEOUT    => 2,
        ],
        'servers' => [
            [
                'host' => '127.0.0.1', 'port' => 11211, 'weight' => 100
            ],
        ],
    ],
],
```

Note that as this package _enhances_ the built-in Laravel 5 memcached Cache driver, the driver string
remains `memcached`.

In case you are unfamiliar with how to use multiple cache stores in Laravel, you would access
this store from your application code like so:

```
$value = Cache::store('memcachedstorefoo')->get('key');
```

### Session

This section discusses the Laravel session configuration file `config/session.php`.

If you are using memcached sessions you will have set the `driver` configuration item to 'memcached'.

If you have added the `B3IT\MemcachedPlus\SessionServiceProvider` as discussed above, the
`store` configuration item is available. This is explained in the following new snippet
you can paste into your session configuration file:

```
    /*
    |--------------------------------------------------------------------------
    | Session Cache Store
    |--------------------------------------------------------------------------
    |
    | When using the "apc" or "memcached" session drivers, you may specify
    | a cache store that should be used for these sessions. This should
    | correspond to a store in your cache configuration. By default,
    | the driver name will be used as the store.
    |
    */

    'store' => null,

```

## laravel-memcached-plus in action

I created a [demo app](https://github.com/b3it/laravel-memcached-plus-demo) for you to see
how this package integrates with Laravel 5 and how you could run it on Heroku.


## Support

Please do let me know if you have any comments or queries.

Thanks!
