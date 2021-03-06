# Woodling

Easy to use fixtures for your models. Requires no configuration on your side, leverages _your code_ to do all the work.

### Installation

You can install Woodling via [Composer](http://getcomposer.org/). Put this in your `composer.json` file and run `$ composer update --dev`:

```
"require-dev":
{
	"summerstreet/woodling": "0.1.*"
}
```

You don't need to do anything else. Woodling will be automatically loaded for you when you first use it.

### Defining blueprints

Blueprints are used to construct an instance of your specified model. Woodling guesses which model class you want to use from the name you give your blueprint.

The following code will return an instance of _User_ class with `name` attribute set to "Mindaugas Bujanauskas" and `hobbies` attribute set to "Skateboarding".

```
Woodling::seed('User', function($blueprint)
{
	$blueprint->name = 'Mindaugas Bujanauskas';
	$blueprint->hobbies = 'Skateboarding';
});
```

You can also specify a different class name by passing it in with an array of arguments. This will create a blueprint called "Admin" and use class "User" when creating an object:

```
Woodling::seed('Admin', array('class' => 'User', 'do' => function($blueprint)
{
	$blueprint->name = 'Mindaugas Bujanauskas';
	$blueprint->admin = 1;
}));
```

By the way, since Woodling utilises _your_ code, you can take advantage of everything that your classes do under the hood. For example, if you were using Laravel's Eloquent, and it would have a `set_password()` method defined, this method would be called when setting your password.

### Autoloading blueprints

Woodling will automagically load your blueprints defined in following locations:

* application/tests/blueprints/*.php
* application/tests/blueprints.php
* app/tests/blueprints/*.php
* app/tests/blueprints.php
* tests/blueprints/*.php
* tests/blueprints.php

You can also add additional search paths. In your bootstrap file add the following lines:

```
use Woodling/Woodling;

// Single path
Woodling::getCore()->finder->addPaths('src/Acme/DemoBundle/Tests/blueprints');

// Several paths
Woodling::getCore()->finder->addPaths(array(
	'/absolute/path/to/blueprints',
	'relative/path/to/blueprints'
));

// Search in new paths
Woodling::getCore()->finder->findBlueprints();
```

When adding paths, keep in mind, that the last segment of the path will be used as both file name and dir name:

```
// Looks in these destinations:
// - tests/blueprints/*.php
// - tests/blueprints.php
Woodling::getCore()->finder->addPaths('tests/blueprints');
```

### Retrieving blueprints

To retrieve instances of your models, you can use the following methods:

```
$user = Woodling::retrieve('User');
$user = Woodling::saved('User');
```

The `retrieve()` method will return an instance of your class. The `saved()` method will create an instance of your class and call the `save()` method on it before returning it.

### Retrieving blueprints with attribute overrides

Sometimes you need to retrieve your models with slightly different attributes than the ones you specified in your blueprint. You can do so by passing an array of `key => value` pairs where `key` is the name of the attribute you want to override.

```
$user = Woodling::retrieve('User', array(
	'hobbies' => 'Skateboarding, cooking',
	'occupation' => 'Developer'
));
```

### Retrieving lists

You can also retrieve an array of several instances instead of a single result. Woodling provides two build strategies for you: `retrieveList()` and `savedList()`. Here's how to use them (attribute overrides are optional):

```
$usersArray = Woodling::retrieveList('User', 50);
$savedUsers = Woodling::savedList('User', 50, array('name' => 'Test User'));
```

### Lazy attributes

If you want to calculate your attribute values during instantiation time, you can do so with _lazy attributes_. Just assign a closure to your attribute. The value returned will be the value used when setting this attribute on your model.

```
Woodling::seed('User', function($blueprint)
{
	$blueprint->created = function() { return time(); };
});
```

### Sequences

Very often you have validation rules on your models or in the database that require you to use unique values. If that is the case, you can use _sequences_. A sequence takes a closure, which receives a counter value. You can then use this counter to construct a unique value for your model.

Each time you retrieve an object, it's counter will be incremented. The following code will set an `email` attribute on your model. The first time you retrieve this object, it will receive a counter value of `1`, the second time your retrieve it, counter will be `2` and so on.

```
Woodling::seed('User', function($blueprint)
{
	$blueprint->sequence('email', function($i)
	{
		return "user{$i}@hostname.com";
	});
});
```

### Faux associations

It is possible to create model associations by returning them from lazy attributes. This is how you'd retrieve an instance with one-to-one relationship from Woodling:

```
Woodling::seed('Weakness', function($weakness)
{
    $weakness->type = 'Fruit';
    $weakness->name = 'Apple';
});

Woodling::seed('Person', function($person)
{
    $person->name = 'Eve';
    $person->weakness = function() { return Woodling::retrieve('Weakness'); };
});

$eve = Woodling::retrieve('Person');
```

And this is how you'd do the same for one-to-many:


```
Woodling::seed('Atom', function($atom)
{
    $atom->element = 'H';
});

Woodling::seed('Molecule', function($molecule)
{
    $molecule->name = 'Water';
    $molecule->atoms = function()
    {
        $h2 = Woodling::retrieve('Atom', array('element' => 'H2'));
        $o = Woodling::retrieve('Atom', array('element' => 'O'));
        return array($h2, $o);
    };
});

$H2O = Woodling::retrieve('Molecule');
```

### Retrieving blueprints with advanced overrides

You can override lazy attributes by passing them in as a regular callback function that returns something. To override sequences, you can put them under `:sequences` array:

```
$user = Woodling::retrieve('User', array(
	'occupation' => 'Developer',
	'created' => function() { return time(); },
	':sequences' => array(
		'email' => function($i) { return "name{$i}@hostname.com"; }
	)
));
```

### More awesome usage

Here's an example that shows how to use sequences and lazy attributes to create more realistic model instances.

```
Woodling::seed('User', function($blueprint)
{
	$blueprint->name = 'Mindaugas';
	$blueprint->surname = 'Bujanauskas';
	$blueprint->birthday = '1989.03.03';
	$blueprint->sequence('email', function($i) use($blueprint)
	{
		$email = "{$blueprint->name}{$i}@hostname.com";
		return strtolower($email);
	});
	$blueprint->created = function() use($blueprint)
	{
		$date = date('Y.m.d', strtotime("{$blueprint->birthday} + 20 years"));
		return $date;
	};
});
```

### Free tip

Don't forget to put `use Woodling\Woodling;` at the top of your file. This will let you use short class syntax instead of having to prepend it with a namespace.

### License

For license information, check the LICENSE file.
