# Cookie - Laravel file generator

Cookie is a Laravel package which helps you automate creation of files.

[![Build Status](https://travis-ci.org/skyronic/laravel-cookie.svg?branch=master)](https://travis-ci.org/skyronic/laravel-cookie)

### Benefits

* If you create a type of file frequently, you can automate it and improve productivity. 
* Prevent "context switching" where you lose focus for 30 seconds while you create new files, directories and populate it with boilerplate.
* Comes with several built-in boilerplates. Easy to share your own as github gists.
* All boilerplates and generators are part of your repository, letting you share standard templates with your team.
* Find things like `artisan make:model` and `artisan make:controller` useful? You can make your own.
* All boilerplates can be written as blade templates.

## Quick Start

**Step 1**: Install the package

```bash
$ composer require skyronic/laravel-cookie
```

**Step 2**: Add `CookieServiceProvider` to your `config/app.php`

```php
'providers' => [
    // ... other providers ...
    
    \Skyronic\Cookie\CookieServiceProvider::class,
]
```

**Step 3**: Publish the "goodies" - an included set of useful boilerplates like PHP Classes, Vue Components, etc.

```bash
$ php artisan vendor:publish --tag='goodies'
```

**Step 4**: You can list all the installed boilerplates

```bash
$ php artisan bake:list

+---------------+------------------------------+
| Type          | Name                         |
+---------------+------------------------------+
| js            | JavaScript File              |
| php:class     | PHP Class in 'app' Directory |
| php:trait     | PHP Trait in 'app' Directory |
| vue:component | Vue Component as a .vue file |
+---------------+------------------------------+


Use `artisan bake <type>` to create a new file!
```

**Step 5**: You can create a php class now:

```bash
$ php artisan bake php:class "Support/Helpers/AwesomeHelper" --extends "BaseHelper" --constructor

Created file [ app/Support/Helpers/AwesomeHelper.php ]
```

The generator `php:class` creates one by default in. You can now open app/Support/Helpers/AwesomeHelper.php

```php
<?php

namespace App\Support\Helpers\AwesomeHelper;

class AwesomeHelper extends BaseHelper  {
    public function __construct () {

    }
}
```

**Step 6**: Create your own template:

```bash
$ php artisan bake:new mytemplate --description "My New Template"

Created new boilerplate at [ resources/boilerplates/mytemplate.boilerplate.txt ]
```

## Understanding Boilerplate Template

Open the file created by Step 6. You will see something like this:

```
{
   "name": "My Template",
   "out": "edit/me/{{ $name }}.txt",
   "params": {
        "myParam": "optional"
   }
}
---

Template goes here. Blade syntax works. You can use a parameter like {{ $myParam }}

```

There's two parts to the file, separated by `---`. 

* The top part is the configuration of how the template should behave, and also specifying parameters
* The bottom part is the actual template which will be.

### The configuration object

Let's take a closer look at this config object:

```
{
   "name": "My Template",
   "out": "edit/me/{{ $name }}.txt",
   "params": {
        "myParam": "optional"
   }
}
```

This should be valid JSON. The key `name` is the name of the template used for `bake:list`. Not to be confusued with the `$name` variable.

#### Setting Output Path

If you try to run the template with something like:

```
$ php artisan bake mytemplate foo/bar
```

The output path here will be: `edit/me/foo/bar.txt`. `$name` contains the second parameter, and even the strings can use blade so `{{ $name }}` will produce the path.

### Parameters

Parameters allow you to customize and change the content of the file. For example, here we have `myParam`. So running this boilerplate with

```
$ php artisan bake mytemplate foo/bar --myParam "Hello"
```

Will result in the text file:

```
Template goes here. Blade syntax works. You can use a parameter like Hello
```

#### Flag Parameters

Here's a simple template (some elements omitted for brevity).
```
"params": {
    "someFlag": "optional"
}
---
This is always visible.

@if($someFlag)
This is only visible when the flag is set
@endif
```

Now we can run it like:

```
# $someFlag will be set to false
$ php artisan bake mytemplate foo/bar 

# $someFlag will be set to true
$ php artisan bake mytemplate foo/bar --someFlag
```

#### Required Parameters

```
"params": {
    "className": "required"
}
---
class {{ $className }} {
   
}
```

```
$ php artisan bake mytemplate foo/bar

  [Skyronic\Cookie\CookieException]
  Needs argument [ className ]
```

#### Optional Parameters

```
"params": {
    "authorName": "optional"
}
---
@if($authorName)
/* Author: {{ $authorName }} */
@endif
class MyClass {
   
}
```

You can recognize the `if` and `endif` as blade conditional structures. If `authorName` is set like:

```
$ php artisan bake mytemplate foo/bar --authorName John
```

Then the value is set to "John". Else it's null.

#### Default Values

If you set the parameter to anything except `flag` or `optional` or `required` it's considered a default value.
```
"params": {
    "copyrightYear": "2017"
}
---

/* Copyright (c) {{ $copyrightYear }} */
```

The value is going to be set to 2017 unless specified otherwise.

```
# Set to default value of 2017
$ php artisan bake mytemplate foo/bar

# Override the value to 2016
$ php artisan bake mytemplate foo/bar --copyrightYear 2016
```

## Tips for writing boilerplates

* A template like `vue__component.boilerplate.txt` will become `vue:component` for cleaner organization. You can use `__` in your own templates.
* **Important:** Pass a `--dry-run` flag like `php artisan bake --dry-run mytemplate foo/bar --myParam "paramvalue"` to display the output in console. This lets you iterate and fix any potential issues without creating files.
* You can use most of laravel's helper functions and even some other PHP classes with some advanced blade and the `@php` directive
* You can use paths like `foo/bar/{{ $name }}` and Cookie will automatically adjust directory separators on windows.

## Formatter

Sometimes you might need to do some string manipulation. Later versions of Cookie will contain more comprehensive string manipulation later. 

#### Camel-case, Snake-case, etc

You can use Laravel's built in helpers for things like [camel_case](https://laravel.com/docs/5.4/helpers#method-camel-case) and others.
 
 
#### Basename from path

If you've got something like `app/Support/MyHelper.php` and want to extract `MyHelper` you can use `Format::baseName ($path)` which extracts a classname like entity, ignoring any file extension.

#### Getting a namespace from path

Namespaces are a bit tricky, since they need to render forward-slashes. Cookie contains a simple format helper which can generate a namespace from a given file path. It uses the laravel `app` directory and `App` namespace by default.

```
// $path = "app/Support/Helpers/AwesomeHelper.php"
Format::getNamespace ($path)
// -> "App\Support\Helpers"

// For non `app` directories, you need to manually specify namespace routes
// $path = "tests/Unit/HelperTests/AwesomeHelperTest.php"
Format::getNamespace ($path, 'tests', "Tests")
// -> "Tests\Unit\HelperTests"
```

## Example: PHP Class generator

First, be sure that you've run `php artisan vendor:publish --tag='goodies'` and check `app/resources/boilerplates/php__class.boilerplate.txt`

```
{
   "name": "PHP Class in 'app' Directory",
   "out": "app/{{ $name }}.php",
   "params": {
       "extends": "optional",
       "constructor": "flag"
   }
}
---
<?php

namespace {{ Format::getNamespace($path) }};

class {{ Format::baseName($name) }} @if($extends)extends {{ $extends }}@endif  {
@if($constructor)
    public function __construct () {

    }
@endif
}

```

The example should be pretty self explanatory. But can illustrate that even a little blade templating can go a long way.


### What's with the name?

You can use a [Cookie Cutter](https://en.wikipedia.org/wiki/Cookie_cutter) to quickly and reliably create similar (delicious) cookies from a template. 

Also, Cookie is the name of a really cute Beagle I know.

