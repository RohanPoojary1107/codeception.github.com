---
layout: doc
title: FunctionalTests - Codeception 4 Documentation
---

<div class="alert alert-warning">👴 <b>You are reading docs for Codeception 4</b>. Current version is 5.x <a href="/docs/FunctionalTests">Read for latest version</a></div>

# Functional Tests

Now that we've written some acceptance tests, functional tests are almost the same, with one major difference:
Functional tests don't require a web server.

Under the hood, Codeception uses Symfony's [BrowserKit](https://symfony.com/doc/current/components/browser_kit.html)
to "send" requests to your app. So there's no real HTTP request made, but rather a BrowserKit
[Request object](https://github.com/symfony/browser-kit/blob/master/Request.php) with the required properties is
passed to your framework's (front-)controller.

As a first step, you need to enable Codeception's module for your framework in `functional.suite.yml` (see below).

All of Codeception's framework modules share the same interface, and thus your tests are not bound to any one of them.
This is a sample functional test:

{% highlight php %}

<?php
// LoginCest.php

class LoginCest
{
    public function tryLogin(FunctionalTester $I)
    {
        $I->amOnPage('/');
        $I->click('Login');
        $I->fillField('Username', 'Miles');
        $I->fillField('Password', 'Davis');
        $I->click('Enter');
        $I->see('Hello, Miles', 'h1');
        // $I->seeEmailIsSent(); // only for Symfony
    }
}

{% endhighlight %}

As you see, the syntax is the same for functional and acceptance tests.

## Limitations

Functional tests are usually much faster than acceptance tests. But functional tests are less stable as they run Codeception
and the application in one environment. If your application was not designed to run in long lived processes (e.g.
if you use the `exit` operator or global variables), then functional tests are probably not for you.

### Headers, Cookies, Sessions

One of the common issues with functional tests is the use of PHP functions that deal with headers, sessions and cookies.
As you may already know, the `header` function triggers an error if it is executed after PHP has already output something.
In functional tests we run the application multiple times, thus we will get lots of irrelevant errors in the result.

### External URLs

Functional tests cannot access external URLs, just URLs within your project. You can use PhpBrowser to open external URLs.

### Shared Memory

In functional testing, unlike running the application the traditional way, the PHP application does not stop
after it has finished processing a request. Since all requests are run in one memory container, they are not isolated.
So **if you see that your tests are mysteriously failing when they shouldn't - try to execute a single test.**
This will show if the tests were failing because they weren't isolated during the run.
Keep your memory clean, avoid memory leaks and clean global and static variables.

## Enabling Framework Modules

You have a functional testing suite in the `tests/functional` directory.
To start, you need to include one of the framework modules in the suite configuration file: `tests/functional.suite.yml`.

### Symfony

To perform Symfony integration you just need to include the Symfony module into your test suite. If you also use Doctrine2,
don't forget to include it too. To make the Doctrine2 module connect using the `doctrine` service from Symfony,
you should specify the Symfony module as a dependency for Doctrine2:

{% highlight yaml %}

# functional.suite.yml

actor: FunctionalTester
modules:
    enabled:
        - Symfony
        - Doctrine2:
            depends: Symfony # connect to Symfony

{% endhighlight %}

By default this module will search for AppKernel in the `app` directory.

The module uses the Symfony Profiler to provide additional information and assertions.

[See the full reference](https://codeception.com/docs/modules/Symfony)

### Laravel5

The [Laravel5](https://codeception.com/docs/modules/Laravel5) module is included and requires no configuration:

{% highlight yaml %}

# functional.suite.yml

actor: FunctionalTester
modules:
    enabled:
        - Laravel5

{% endhighlight %}

### Yii2

Yii2 tests are included in [Basic](https://github.com/yiisoft/yii2-app-basic)
and [Advanced](https://github.com/yiisoft/yii2-app-advanced) application templates. Follow the Yii2 guides to start.

### Zend Framework 2

Use [the ZF2 module](https://codeception.com/docs/modules/ZF2) to run functional tests inside Zend Framework 2:

{% highlight yaml %}

# functional.suite.yml

actor: FunctionalTester
modules:
    enabled:
        - ZF2

{% endhighlight %}

### Zend Expressive

[Zend Expressive](https://codeception.com/docs/modules/ZendExpressive) tests can be executed with enabling a corresponding module.

{% highlight yaml %}

# functional.suite.yml

actor: FunctionalTester
modules:
    enabled:
        - ZendExpressive

{% endhighlight %}

> See module reference to more configuration options

### Phalcon 4

The `Phalcon4` module requires creating a bootstrap file which returns an instance of `\Phalcon\Mvc\Application`.
To start writing functional tests with Phalcon support you should enable the `Phalcon4` module
and provide the path to this bootstrap file:

{% highlight yaml %}

# functional.suite.yml

actor: FunctionalTester
modules:
    enabled:
        - Phalcon4:
            bootstrap: 'app/config/bootstrap.php'
             cleanup: true
             savepoints: true

{% endhighlight %}

[See the full reference](https://codeception.com/docs/modules/Phalcon4)

## Writing Functional Tests

Functional tests are written in the same manner as [Acceptance Tests](https://codeception.com/docs/03-AcceptanceTests)
with the `PhpBrowser` module enabled. All framework modules and the `PhpBrowser` module share the same methods
and the same engine.

Therefore we can open a web page with `amOnPage` method:

{% highlight php %}

<?php
$I->amOnPage('/login');

{% endhighlight %}

We can click links to open web pages:

{% highlight php %}

<?php
$I->click('Logout');
// click link inside .nav element
$I->click('Logout', '.nav');
// click by CSS
$I->click('a.logout');
// click with strict locator
$I->click(['class' => 'logout']);

{% endhighlight %}

We can submit forms as well:

{% highlight php %}

<?php
$I->submitForm('form#login', ['name' => 'john', 'password' => '123456']);
// alternatively
$I->fillField('#login input[name=name]', 'john');
$I->fillField('#login input[name=password]', '123456');
$I->click('Submit', '#login');

{% endhighlight %}

And do assertions:

{% highlight php %}

<?php
$I->see('Welcome, john');
$I->see('Logged in successfully', '.notice');
$I->seeCurrentUrlEquals('/profile/john');

{% endhighlight %}

Framework modules also contain additional methods to access framework internals. For instance, Laravel5, Phalcon,
and Yii2 modules have a `seeRecord` method which uses the ActiveRecord layer to check that a record exists in the database.

Take a look at the complete reference for the module you are using. Most of its methods are common to all modules
but some of them are unique.

You can also access framework globals inside a test or access the dependency injection container
inside the `Helper\Functional` class:

{% highlight php %}

<?php
namespace Helper;

class Functional extends \Codeception\Module
{
    function doSomethingWithMyService()
    {
        $service = $this->getModule('Symfony')->grabServiceFromContainer('myservice');
        $service->doSomething();
    }
}

{% endhighlight %}

Also check all available *Public Properties* of the used modules to get full access to their data.

## Error Reporting

By default Codeception uses the `E_ALL & ~E_STRICT & ~E_DEPRECATED` error reporting level.
In functional tests you might want to change this level depending on your framework's error policy.
The error reporting level can be set in the suite configuration file:

{% highlight yaml %}

actor: FunctionalTester
...
error_level: E_ALL & ~E_STRICT & ~E_DEPRECATED

{% endhighlight %}

`error_level` can also be set globally in `codeception.yml` file. In order to do that, you need to specify `error_level` as a part of `settings`. For more information, see [Global Configuration](https://codeception.com/docs/reference/Configuration). Note that suite specific `error_level` value will override global value.

## Conclusion

Functional tests are great if you are using powerful frameworks. By using functional tests you can access
and manipulate their internal state. This makes your tests shorter and faster. In other cases,
if you don't use frameworks there is no practical reason to write functional tests.
If you are using a framework other than the ones listed here, create a module for it and share it with the community.




* **Next Chapter: [UnitTests >](/docs/05-UnitTests)**
* **Previous Chapter: [< AcceptanceTests](/docs/03-AcceptanceTests)**

<div class="alert alert-warning"><a href="https://github.com/Codeception/codeception.github.com/edit/master/docs/4.x/FunctionalTests.md"><strong>Edit</strong> this page on GitHub</a></div>