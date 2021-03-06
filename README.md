Geocoder
========

**Geocoder** is a library which helps you build geo-aware applications. It provides an abstraction layer for geocoding manipulations.

The library is splitted in two parts: `HttpAdapter` and `Provider`:

**HttpAdapters** are responsible to get data from remote APIs.
Currently, there is one adapter for [Buzz](https://github.com/kriswallsmith/Buzz), a lightweight PHP 5.3 library for issuing HTTP requests.

**Providers** contain the logic to extract useful information.
Currently, there are three providers for the following APIs:

* [IpInfoDB](http://www.ipinfodb.com/) as IP-Based geocoding provider;
* [Yahoo! PlaceFinder](http://developer.yahoo.com/geo/placefinder/) as Address-Based geocoding provider;
* [HostIp](http://www.hostip.info/) as IP-Based geocoding provider.

Installation
------------

If you don't use a ClassLoader in your application, just require the provided autoloader:

``` php
<?php

require_once 'src/autoload.php';
```

You're done.


Usage
-----

First, you need an `adapter`:

``` php
<?php

$adapter  = new \Geocoder\HttpAdapter\BuzzHttpAdapter();
```

The `BuzzHttpAdapter` is tweakable, actually you can pass a `Browser` object to this adapter:

``` php
<?php

$buzz    = new \Buzz\Browser(new \Buzz\Client\Curl());
$adapter = new \Geocoder\HttpAdapter\BuzzHttpAdapter($buzz);
```

Now, you have to choose a `provider`.

The `YahooProvider` is able to geocode both **IP addresses** and **street addresses**.
The `IpInfoDbProvider` and `HostIpProvider` are able to geocode **IP addresses** only.

You can use one of them or write your own provider. You can also register all providers and decide later.
That's we'll do:

``` php
<?php

$geocoder = new \Geocoder\Geocoder();
$geocoder->registerProviders(array(
    new \Geocoder\Provider\YahooProvider(
        $adapter, '<YAHOO_API_KEY>', $locale
    ),
    new \Geocoder\Provider\IpInfoDbProvider(
        $adapter, '<IPINFODB_API_KEY>'
    ),
    new \Geocoder\Provider\HostIpProvider($adapter)
));
```

The `$locale` parameter is available and optional for the `YahooProvider`.

Everything is ok, enjoy!

API
---

The main method is called `geocode()` which receives a value to geocode. It can be an IP address or a street address (partial or not).

``` php
<?php

$geocoder->geocode('88.188.221.14');
// Result is:
// "latitude"    => string(9) "47.901428"
// "longitude"  => string(8) "1.904960"
// "city"       => string(7) "Orleans"
// "zipcode"    => string(0) ""
// "region"     => string(6) "Centre"
// "country"    => string(6) "France"

$geocoder->geocode('10 rue Gambetta, Paris, France');
// Result is:
// "latitude"   => string(9) "48.863217"
// "longitude"  => string(8) "2.388821"
// "city"       => string(5) "Paris"
// "zipcode"    => string(5) "75020"
// "region"     => string(14) "Ile-de-France"
// "country"    => string(6) "France"
```

Once you've called this method, the `geocoder` contains information that you can query with the following getters:

* `getCoordinates()` will return an array with `latitude` and `longitude` values;
* `getLatitude()` will return the `latitude` value;
* `getLongitude()` will return the `longitude` value;
* `getCity()` will return the `city`;
* `getZipcode()` will return the `zipcode`;
* `getRegion()` will return the `region`;
* `getCountry()` will return te `country`.

The Geocoder's API is fluent, you can write:

``` php
<?php

$geocoder
    ->registerProvider(new \My\Provider\Custom($adapter))
    ->using('custom')
    ->geocode('68.145.37.34')
    ;
```

The `using()` method allows you to choose the `adapter` to use. When you deal with multiple adapters, you may want to
choose one of them. The default behavior is to use the first one but it can be annoying.


Reverse Geocoding
-----------------

This library provides a `reverse()` method to retrieve information from coordinates:

``` php
$geocoder->reverse($latitude, $longitude);
```

**Note:** the `IpInfoDbProvider` doesn't provide this feature.


Extending Things
----------------

You can provide your own `adapter`, you just need to create a new class which implements `HttpAdapterInterface`.

You can also write your own `provider` by implementing the `ProviderInterface`. Note, the `AbstractProvider` class can help you by
providing useful features.


Unit Tests
----------

To run unit tests, you'll need two dependencies:

```
git clone git://github.com/kriswallsmith/Buzz.git vendor/Buzz
git clone git://github.com/symfony/ClassLoader.git vendor/Symfony/Component/ClassLoader
```

Once installed, just launch the following command:

```
phpunit
```


Credits
-------

* William Durand


License
-------

Geocoder is released under the MIT License. See the bundled LICENSE file for details.
