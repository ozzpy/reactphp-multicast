# clue/reactphp-multicast [![Build Status](https://travis-ci.org/clue/reactphp-multicast.svg?branch=master)](https://travis-ci.org/clue/reactphp-multicast)

Simple multicast UDP message client and server for [ReactPHP](https://reactphp.org/).

Multicast UDP messages are needed for quite a few (low-level) networking protocols.
This library exposes a simple subset of commonly needed functionality for
multicast networking through an easy to use API.

Among others, multicast networking is the basis for:

* MDNS (Multicast DNS)
* HTTPU/HTTPMU (Multicast and Unicast UDP HTTP Messages)
* UPNP/SSDP (Univeral Plug and Play / Simple Service Discovery Protocol).

**Table of Contents**

* [Quickstart example](#quickstart-example)
* [Usage](#usage)
  * [Factory](#factory)
    * [createSender()](#createsender)
    * [createReceiver()](#createreceiver)
  * [SocketInterface](#socketinterface)
* [Install](#install)
* [Tests](#tests)
* [License](#license)

## Quickstart example

Once [installed](#install), you can use the following code to create a simple
echo server that listens for incoming multicast messages:

```php
$loop = React\EventLoop\Factory::create();
$factory = new Factory($loop);
$socket = $factory->createReceiver('224.10.20.30:4050');

$socket->on('message', function ($data, $remote) use ($socket) {
    echo 'Sending back ' . strlen($data) . ' bytes to ' . $remote . PHP_EOL;
    $socket->send($data, $remote);
});

$loop->run();
```

See also the [examples](examples).

## Usage

### Factory

The `Factory` is responsible for creating your [`SocketInterface`](#socketinterface) instances.
It also registers everything with the main [`EventLoop`](https://github.com/reactphp/event-loop#usage).

```php
$loop = React\EventLoop\Factory::create();
$factory = new Factory($loop);
```

#### createSender()

The `createSender(): SocketInterface` method can be used to
create a socket capable of sending outgoing multicast datagrams and receiving
incoming unicast responses. It returns a [`SocketInterface`](#socketinterface) instance.

```php
$socket = $factory->createSender();

// send a multicast message to everybody listening on the given address
$socket->send('hello?', '224.10.20.30:4050');

// report incoming unicast replies
$socket->on('message', function ($data, $address) {
    echo 'received ' . strlen($data) . ' bytes from ' . $address . PHP_EOL;
});
```

This method works on PHP versions as old as PHP 5.3 (and up), as its socket API has always been
[level 1 multicast conformant](https://www.tldp.org/HOWTO/Multicast-HOWTO-2.html#ss2.2).

#### createReceiver()

The `createReceiver(string $address): SocketInterface` method can be used to
create a socket capable of receiving incoming multicast datagrams and sending
outgoing unicast or multicast datagrams. It returns a [`SocketInterface`](#socketinterface) instance.

```php
$socket = $factory->createReceiver('224.10.20.30:4050');

// report incoming multicast messages 
$socket->on('message', function ($data, $remote) use ($socket) {
    echo 'Sending back ' . strlen($data) . ' bytes to ' . $remote . PHP_EOL;
    
    // send a unicast reply to the remote
    $socket->send($data, $remote);
});
```

This method requires PHP 5.4 (or up) and `ext-sockets`.
Otherwise, it will throw a `BadMethodCallException`.
This is a requirement because receiving multicast datagrams requires a
[level 2 multicast conformant](https://www.tldp.org/HOWTO/Multicast-HOWTO-2.html#ss2.2)
socket API.
The required multicast socket options and constants have been added with
[PHP 5.4](http://php.net/manual/en/migration54.global-constants.php) (and up).
These options are only available to the low level socket API (ext-sockets), not
to the newer stream based networking API.

Internally, this library uses a workaround to create stream based sockets
and then sets the required socket options on its underlying low level socket
resource.
This is done because ReactPHP is built around the general purpose stream based API
and has only somewhat limited support for the low level socket API.

### SocketInterface

The [`Factory`](#factory) creates instances of the `React\Datagram\SocketInterface`
from the [react/datagram](https://github.com/reactphp/datagram) package.
This means that you can use all its normal methods like so:

```php
$socket->send($message, $address);

$socket->on('message', function ($message, $address) { });
$socket->on('close', function() { });

$socket->pause();
$socket->resume();

$socket->end();
$socket->close();
```

Please refer to the [datagram documentation](https://github.com/reactphp/datagram#usage) for more details.

## Install

The recommended way to install this library is [through Composer](http://getcomposer.org).
[New to Composer?](http://getcomposer.org/doc/00-intro.md)

```bash
$ composer require clue/multicast-react:~1.0
```

This project aims to run on any platform and thus does not require any PHP
extensions and supports running on legacy PHP 5.3 through current PHP 7+ and
HHVM.
It's *highly recommended to use PHP 7+* for this project.

The [`createSender()`](#createsender) method works on all supported platforms
without any additional requirements. However, the [`createReceiver()`](#createreceiver)
method requires PHP 5.4 (or up) and `ext-sockets`. See above for more details.

## Tests

To run the test suite, you first need to clone this repo and then install all
dependencies [through Composer](https://getcomposer.org):

```bash
$ composer install
```

To run the test suite, go to the project root and run:

```bash
$ php vendor/bin/phpunit
```

## License

MIT
