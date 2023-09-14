# StringTemplate 
[![Build Status](https://github.com/youniwemi/StringTemplate/actions/workflows/php.yml/badge.svg?branch=master)](https://github.com/youniwemi/StringTemplate/actions/workflows/php.yml)
[![Packagist](https://img.shields.io/packagist/dt/youniwemi/string-template.svg)](https://packagist.org/packages/youniwemi/string-template/stats)
[![Packagist](https://img.shields.io/packagist/dm/youniwemi/string-template.svg)](https://packagist.org/packages/youniwemi/string-template/stats)


StringTemplate is a very simple string template engine for php. 

It allows named and nested substutions as well as conditionnals et custom filters (originaly a fork of nicmart/StringTemplate )"

For installing instructions, go to the end of this README.

## Why

I have often struggled against sprintf's lack of a named placeholders feature, 
so I have decided to write once and for all a simple component that allows you to render a template string in which
placeholders are named.

Furthermore, its placeholders can be nested as much as you want (multidimensional arrays allowed).

## Usage
Simply create an instance of `Youniwemi\StringTemplate\Engine`, and use its `render` method. 

Placeholders are delimited by default by `{` and `}`, but you can specify others through
the class constructor.

```php
$engine = new Youniwemi\StringTemplate\Engine;

//Scalar value: returns "This is my value: nic"
$engine->render("This is my value: {}", 'nic');

```

You can also provide an array value:

```php
//Array value: returns "My name is Nicolò Martini"
$engine->render("My name is {name} {surname}", ['name' => 'Nicolò', 'surname' => 'Martini']);

```

Nested array values are allowed too! Example:

```php
//Nested array value: returns "My name is Nicolò and her name is Gabriella"
$engine->render(
    "My name is {me.name} and her name is {her.name}",
    [
        'me' => ['name' => 'Nicolò'],
        'her' => ['name' => 'Gabriella']
    ]);
```

Object values will be converted to strings:
```php
class Foo { function __toString() { return 'foo'; }

//Returns "foo: bar"
$engine->render(
    "{val}: bar",
    ['val' => new Foo]);
```

You can change the delimiters as you want:
```php
$engine = new Youniwemi\StringTemplate\Engine(':', '');

//Returns I am Nicolò Martini
$engine->render(
    "I am :name :surname",
    [
        'name' => 'Nicolò',
        'surname' => 'Martini'
    ]);

```


You can use a simple condition:
```php
$engine = new Youniwemi\StringTemplate\Engine();

//Returns Oh! You
$engine->render(
    'Oh! {#name}{test}{/name}',
    [
        'name' => true,
        'test' => 'You'
    ]);

```


You can use a simple condition with else:
```php
$engine = new Youniwemi\StringTemplate\Engine();

//Returns Oh! My
$engine->render(
    'Oh! {#name}{test}{#else}My{/name}',
    [
        'name' => false,
        'test' => 'You'
    ]);

```



You can use a simple filters ( lower|upper|esc_html ):
```php
$engine = new Youniwemi\StringTemplate\Engine();

//Returns Oh! JOHN
$engine->render(
    'Oh! {name|upper}',
    [
        'name' => 'John'
    ]);

```

You can add you own filters:
```php
$engine = new Youniwemi\StringTemplate\Engine('{','}', [
    'ucfist' => 'ucfirst',
    'esc_html' =>  function($string){ return htmlentities($string, ENT_NOQUOTES); } // override a default filter
]);

//Returns Oh! &lt;script&gt;John&lt;/script&gt;'
$engine->render(
    'Oh! {name|esc_html}',
    [
        'name' => '<script>John</script>'
    ]);

```


You can use closures a values
```php
$engine = new Youniwemi\StringTemplate\Engine();

//Returns Oh! John
$engine->render(
    'Oh! {name|upper}',
    [
        'name' => function() {
            return 'John';
        }
    ]);

```

You can use closures can use the variables
```php
$engine = new Youniwemi\StringTemplate\Engine();

//Returns Oh! John Doe
$engine->render(
    'Oh! {name}',
    [
        'first' => 'John',
        'last' => 'Doe',
        'name' => function($values) {
            return $values['first'].' '.$values['last'];
        }
    ]);

```

And lastly, you can use sprintf formats:
 ```php
$engine = new Youniwemi\StringTemplate\SprintfEngine;

//Returns I have 1.2 (1.230000E+0) apples.
    $engine->render(
        "I have {num%.1f} ({num%.6E}) {fruit}.",
        [
            'num' => 1.23,
            'fruit' => 'apples'
        ]
    )

```


### SprintfEngine (Deprecated)
You can use a more powerful version of the engine if you want to specify [convertion specifications](http://php.net/manual/en/function.sprintf.php) for placeholders. The conversion syntax is identical to `sprintf` one, you need only to specify the optional parameter after the placeholder name.

Example:
 ```php
$engine = new Youniwemi\StringTemplate\SprintfEngine;

//Returns I have 1.2 (1.230000E+0) apples.
    $engine->render(
        "I have {num%.1f} ({num%.6E}) {fruit}.",
        [
            'num' => 1.23,
            'fruit' => 'apples'
        ]
    )

```
Keep in mind that power comes at a cost: `SprintfEngine` is 3 times slower than `Engine` 
(although if there are no '%' in the template string then performance is almost the same).

## NestedKeyIterator and NestedKeyArray
Internally the engine iterates through the value array with the `NestedKeyIterator`. `NestedKeyIterator`
iterates through multi-dimensional arrays giving as key the imploded keys stack.

It can be useful even if you don't need the Engine. Keep in mind that it is an `RecursiveIteratorIterator`,
and so you have to pass  a `RecursiveIterator` to its constructor (or, better, a `Youniwemi\StringTemplate\RecursiveArrayOnlyIterator` if you do not want to iterate through objects).

Example:
```php
use Youniwemi\StringTemplate\NestedKeyIterator;
use Youniwemi\StringTemplate\RecursiveArrayOnlyIterator;

$ary = [
    '1' => 'foo',
    '2' => [
        '1' => 'bar',
        '2' => ['1' => 'fog']
    ],
    '3' => [1, 2, 3]
];

$iterator = new NestedKeyIterator(new RecursiveArrayIterator($ary));

foreach ($iterator as $key => $value)
    echo "$key: $value\n";

// Prints
// 1: foo
// 2.1: bar
// 2.2.1: fog
// 3.0: 1
// 3.1: 2
// 3.2: 3

```
### NestedKeyArray
In addition to iteration with nested keys, the library offers a class that allows you to access 
a multidimensional array with flatten nested keys as the ones seen above. It's called `NestedKeyArray`.

Example:
```php
use Youniwemi\StringTemplate\NestedKeyArray;

$ary = [
    '1' => 'foo',
    '2' => [
        '1' => 'bar',
        '2' => ['1' => 'fog']
    ],
    '3' => [1, 2, 3]
];

$nestedKeyArray = new NestedKeyArray($ary);

echo $nestedKeyArray['2.1']; //Prints 'bar'
$nestedKeyArray['2.1'] = 'new bar';
unset($nestedKeyArray['2.2']);
isset($nestedKeyArray['2.1']); //Returns true

foreach ($iterator as $key => $value)
    echo "$key: $value\n";

// Prints
// 1: foo
// 2.1: new bar
// 3.0: 1
// 3.1: 2
// 3.2: 3

```

## Where is it used
I use StringTemplate in [Instareza](https://www.instareza.com), a booking system for activities, as well as in [Mail Control](https://www.wpmailcontrol.com) for its newsletter upcoming feature.

## Install

The best way to install StringTemplate is [through composer](http://getcomposer.org).

Just create a composer.json file for your project:

```JSON
{
    "require": {
        "youniwemi/string-template": "~0.2"
    }
}
```

Then you can run these two commands to install it:

    $ curl -s http://getcomposer.org/installer | php
    $ php composer.phar install

or simply run `composer install` if you have have already [installed the composer globally](http://getcomposer.org/doc/00-intro.md#globally).

Then you can include the autoloader, and you will have access to the library classes:

```php
<?php
require 'vendor/autoload.php';
```
