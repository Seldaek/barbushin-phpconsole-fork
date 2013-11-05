# PHP Console server library

PHP Console allows you to handle PHP errors & excepions, dump variables, execute PHP code remotely and many other things using [Google Chrome extension PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef/details) and [PhpConsole server library](https://github.com/barbushin/php-console).

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/barbushin/php-console/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

### Overview

* Take a look at PHP Console [presentation video](http://youtu.be/vt5fpE0bzSY).
* Install Google Chrome extension [PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef/details).
* See how it works on PHP Console [live examples](http://php-console.com/instance/examples) server.

### Requirements

* Google Chrome extension [PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef) on client.
* PHP 5.3 (or later) on server.

*For projects with PHP < 5.3 you can try to use old [deprecated version](https://groups.google.com/forum/?hl=ru#!forum/php-console-deprecated-version) of PHP Console. But mention that actual last version is much more functional.*

# Installation

### Composer

	{
		"require": {
			"php-console/php-console": "dev-master"
		}
	}

This is the most recommended way, so PhpConsole will be autoloaded using Composer PSR-0 autoloader. Also it can be easy updated to last version using `composer update`.

### GIT

	git clone https://github.com/barbushin/php-console.git php-console

### SVN

	svn checkout https://github.com/barbushin/php-console/trunk php-console

### Download .zip

Download and extract repository [archive](https://github.com/barbushin/php-console/archive/master.zip). 
Include in your project using:

	require_once('/path/to/php-console/src/PhpConsole/__autoload.php');

### Download .phar

Download [PhpConsole.phar](http://phpc/PhpConsole/examples/utils/build_phar.php?download). 
Include in your project using:

	require_once('phar://path/to/PhpConsole.phar');

### Yii Framework extension
See https://github.com/barbushin/php-console-yii

# Features

You can see how most of this features works on PHP Console [live examples](http://php-console.com/instance/examples) server.

## Connector

There is a [\PhpConsole\Connector](src/PhpConsole/Connector.php) class that initializes connection between PHP server and Google Chrome extension. Connection is initalized when `\PhpConsole\Connector` instance is initialized:

	$connector = \PhpConsole\Connector::getInstance();

`\PhpConsole\Connector` uses headers to communicate with client, so it must be initialized before any output.

### Strip sources base path

If you want to see errors sources and traces paths more short, call:

	$connector->setSourcesBasePath('/path/to/project');

So paths like `/path/to/project/module/file.php` will be displayed on client as `/module/file.php`.

### Work with different server encodings

If you internal server encoding is not UTF-8, so you need to call:

	$connector->setServerEncoding('CP1251');

### Initialization performance

PhpConsole server library is optimized for lazy initialization only for clients that have Google Chrome extension PHP Console installed. There is [example](examples/features/highload_optimization.php) of correct initialization PhpConsole on your production server.

## Protect connection

PHP Console provide different ways to protect

### Protect by password

[![ScreenShot](http://php-console.com/res/screenshot/auth_420.png)](http://php-console.com/instance/examples/#protect_by_password)

	$connector->setPassword('yohoho123', true);

Clients will need to enter password to get access to PHP Console server data. All passwords are stored on client as SHA-256 hashes. Second argument says that PHP Console authorization token will depends on client IP.

### SSL only connection mode

	$connector->enableSslOnlyMode();

So all PHP Console clients will be automatically redirected to HTTPS.

### Protect connection by list of allowed IP masks

	$connector->setAllowedIpMasks(array('192.168.*.*'));

## Handle errors

[![ScreenShot](http://php-console.com/res/screenshot/errors_420.png)](http://php-console.com/instance/examples/#handle_errors)

There is a [\PhpConsole\Handler](src/PhpConsole/Handler.php) class that initializes PHP errors & exceptions handlers and provides the next features:

* Handle PHP errors(+fatal & memory limit errors) and exceptions.
* Ignore repeated errors.
* Call previously defined errors and exceptions handlers.
* Handle caught exceptions using `$handler->handleException($exception)`.
* Debug vars using `$handler->debug($var, 'some.tags')`.

Initialize `\PhpConsole\Handler` in the top of your main project script:

	$handler = PhpConsole\Handler::getInstance();
	/* You can override default Handler behavior:
		$handler->setHandleErrors(false);  // disable errors handling
		$handler->setHandleExceptions(false); // disable exceptions handling
		$handler->setCallOldHandlers(false); // disable passing errors & exceptions to prviously defined handlers
	*/
	$handler->start(); // initialize handlers


## Debug vars

[![ScreenShot](http://php-console.com/res/screenshot/debug_420.png)](http://php-console.com/instance/examples/#debug_vars)

PHP Console has multifunctional and smart vars dumper that allows to

* Dump any type variable.
* Dump protected and private objects properties.
* Limit dump by level, items count, item size and total size(see `$connector->getDumper()`).
* Dump objects class name.
* Smart dump of callbacks and Closure.
* Detect dump call source & trace(call `$connector->getDebugDispatcher()->detectTraceAndSource = true`).


### How to call

**Longest** native debug method call: 

	\PhpConsole\Connector::getInstance()->getDebugDispatcher()->dispatchDebug($var, 'some.tags');`

**Shorter** call debug from Handler: 

	\PhpConsole\Handler::getInstance()->debug($var, 'some.tags');

**Shortest** call debug using global `PC` class

	\PhpConsole\Helper::register(); // it will register global PC class
	// ...
	PC::debug($var, 'tag');
	PC::tag($var);

**Custom** call debug by user defined function

	function d($var, $tags = null) {
		\PhpConsole\Connector::getInstance()->getDebugDispatcher()->debug($var, $tags, 1);
	}
	d($var, 'some.tags');

### Tags

* Debug tags argument is optional.
* Tags is a string with tags separated by dot(e.g. "low.db").
* Tags can be used to identify what exactly var was dumped.
* You can configure client to ignore displaying some tags.

## Remote PHP code execution

[![ScreenShot](http://php-console.com/res/screenshot/eval_terminal_420.png)](http://php-console.com/instance/examples/#eval_terminal)

PHP Console provide a way to execute PHP code on your server remotely, from Google Chrome extension terminal.

* Remote PHP code execution allowed only in password protected mode
* Every eval request is signed with unique SHA-256 token
* Result contains: `output`, `return` and `time` data
* Errors & exception occured during PHP code execution will be handled
 

### Configuration

	$connector = PhpConsole\Connector::getInstance();
	$connector->setPassword($password);
	
	// Configure eval provider
	$evalProvider = $connector->getEvalDispatcher()->getEvalProvider();
	$evalProvider->addSharedVar('post', $_POST); // so "return $post" code will return $_POST
	$evalProvider->setOpenBaseDirs(array(__DIR__)); // see http://php.net/open-basedir
	
	$connector->startEvalRequestsListener(); // must be called in the end of all configurations


## PSR-3 logger implementation

There is PHP Console implementation of [PSR-3](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md) interface. to integrate PHP Console with PSR-3 compitable loggers(e.g. [Monolog](https://github.com/Seldaek/monolog)). See [\PhpConsole\PsrLogger](src/PhpConsole/PsrLogger.php).

## Jump to file

Read [this article](https://github.com/barbushin/php-console/wiki/Jump-to-file) if you want to configure PHP Console extension to open errors/exceptions source file:line right in your IDE, just by click on the button in Notification popup.

## Easy migrate from PhpConsole `v1.x` to `v3.x`

If you have used PhpConsole `v1.x` and want to migrate to `v3.x`  without any code changes, so just use [\PhpConsole\OldVersionAdapter](src/PhpConsole/OldVersionAdapter.php):

	\PhpConsole\OldVersionAdapter::register(); // register PhpConsole v1.x class emulator
	
	// Call old PhpConsole v1 methods as is
	PhpConsole::start(true, true, $_SERVER['DOCUMENT_ROOT']);
	PhpConsole::debug('Debug using old method PhpConsole::debug()', 'some,tags');
	debug('Debug using old function debug()', 'some,tags');
	echo $undefinedVar;
	PhpConsole::getInstance()->handleException(new Exception('test'));
	
	// Call new PhpConsole methods, if you want :)
	\PhpConsole\Connector::getInstance()->setServerEncoding('cp1251');
	\PhpConsole\Helper::register();
	PC::debug('Debug using new methods');

But, anyway, if you can't migrate to new version of PHP Console because of using PHP < 5.3 on your servers, then you can use old [deprecated version](https://groups.google.com/forum/?hl=ru#!forum/php-console-deprecated-version) of PHP Console.
