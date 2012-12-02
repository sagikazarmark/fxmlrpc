# FXMLRPC: really fast XML/RPC for PHP [![Build Status](https://secure.travis-ci.org/lstrojny/fxmlrpc.png)](http://travis-ci.org/lstrojny/fxmlrpc)

 - A convenient, object oriented API (similar to the XML/RPC client in Zend Framework)
 - Very fast serializing and parsing of the XML payloads involved
 - Stick to the HTTP client you already use: Buzz, ZF1/ZF2 HTTP client, Guzzle, PECL HTTP
 - Licensed under the terms of the liberal MIT license
 - Supports modern standards: easy installation via composer, fully PSR-0, PSR-1 and PSR-2 compatible
 - Relentlessly unit- and integration tested

## Latest improvements

 - `[IMPROVEMENT]` Small memory and performance improvements for serializers and parsers
 - `[BC]` Deprecated constructor of `FXMLRPC\Value\Base64` and introduced `::serialize()` an `::deserialize()` instead.
 - `[FEATURE]` Adding `FXMLRPC\Client::prependParams()` and `FXMLRPC\Client::appendParams()` to set default params. This helps e.g. when you need to add authorization information for every call
 - `[FEATURE]` Timing Loggers now support threshold based logging to ease controlling your servers responds in a certain time
 - `[TESTING]` Travis now runs the test suite against various versions of supported HTTP clients and logging components.

### How fast is it really?

IO performance is out of reach from a userspace perspective, but parsing and
serialization speed is what matters. How fast can we generate the XML payload
from PHP data structures and how fast can we parse the servers response? FXMLRPC
uses stream based XML writers/readers to achieve it’s performance and heavily
optimizes (read uglifies) for it. As as result the userland version is only
around 2x slower than the native C implementation (ext/xmlrpc).


#### Parser
```
Zend\XmlRpc\Value (ZF2): 249.02972793579 sec
Zend_XmlRpc_Value (ZF1): 253.88145494461 sec
FXMLRPC\Parser\XMLReaderParser: 36.274516105652 sec
FXMLRPC\Parser\NativeParser: 18.652323007584 sec
```

#### Serializer
```
Zend\XmlRpc\Request (ZF2): 52.004573106766 sec
Zend_XmlRpc_Request (ZF1): 65.042532920837 sec
FXMLRPC\Serializer\XMLWriterSerializer: 23.652673006058 sec
FXMLRPC\Serializer\NativeSerializer: 9.0790779590607 sec
```


### Usage

#### Basic Usage
```php
<?php
$client = new FXMLRPC\Client('http://endpoint.com');
$client->call('remoteMethod', array('arg1', true));
```

#### Using native (ext/xmlrpc based) serializer/parser (for even better performance)
```php
<?php
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    null,
    new FXMLRPC\Parser\NativeParser(),
    new FXMLRPC\Serializer\NativeSerializer()
);
$client->call('remoteMethod', array('arg1', true));
```

#### Integration for various HTTP clients
```php
<?php
/** Buzz (https://github.com/kriswallsmith/Buzz) */
$browser = new Buzz\Browser();
$browser->...();
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    new FXMLRPC\Transport\BuzzBrowserBridge($browser)
);

/** Zend Framework 1 (http://framework.zend.com/) */
$httpClient = new Zend_Http_Client();
$httpClient->...();
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    new FXMLRPC\Transport\ZF1HttpClientBridge($httpClient)
);

/** Zend Framework 2 (http://framework.zend.com/zf2) */
$httpClient = new Zend\Http\Client();
$httpClient->...();
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    new FXMLRPC\Transport\ZF2HttpClientBridge($httpClient)
);

/** Guzzle (http://guzzlephp.org/) */
$httpClient = new Guzzle\Http\Client();
$httpClient->...();
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    new FXMLRPC\Transport\GuzzleBridge($httpClient)
);

/** PECL HTTP (http://pecl.php.net/pecl_http) */
$request = new HttpRequest();
$request->...();
$client = new FXMLRPC\Client(
    'http://endpoint.com',
    new FXMLRPC\Transport\PeclHttpBridge($request)
);
```

#### Timing XML/RPC requests to find problematic calls
FXMLRPC allows you to time your XML/RPC request, to find out which took how long. It provides a
`FXMLRPC\Timing\TimingDecorator` which can be used with various timers implementing
`FXMLRPC\Timing\TimerInterface`. Currently implemented are bridges for Monolog, Zend Framework 1
`Zend_Log` and Zend Framework 2 `Zend\Log`.

Usage:
```php
<?php
$client = new FXMLRPC\Timing\TimingDecorator(
    new FXMLRPC\Client(...),
    new FXMLRPC\Timing\MonologTimerBridge(
        $monolog,
        Monolog\Logger::ALERT,
        'My custom log message template %F'
    )
);
```
