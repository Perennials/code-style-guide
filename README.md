Code Style Guide
================

* [About](#about)
  * [Contributing](#contributing)
  * [Exceptions from this guide](#exceptions-from-this-guide)
* [Examples](#examples)
  * [Basic sanity](#basic-sanity)
  * [Spacing and new lines](#spacing-and-new-lines)
  * [Naming](#naming)
  * [Global scope rules](#global-scope-rules)
  * [Variable scoping](#variable-scoping)
  * [Comparisons](#comparisons)
  * [Array and property access](#array-and-property-access)
  * [Empty values](#empty-values)
  * [Class member visibility](#class-member-visibility)
  * [Conditions](#conditions)
  * [Error checks, conditions with long if-bodies](#error-checks-conditions-with-long-if-bodies)
  * [Language abuse, performance considerations](#language-abuse-performance-considerations)
  * [Doc comments](#doc-comments)
* [Authors](#authors)


About
-----
Obviously this is my own code style. As much as I don't want to try to force
any style to anyone, the carelessness and lack of love to one's code I meet
sometimes is appalling. Mixed tabs and spaces, one liners that you need to
scroll two screens, whole functions bodies wrapped in a single if, the list
goes on. This list doesn't cover every possible case or stupidity one can come
up with, but I will expand it in time when I meet more examples of bad code.

The guide was written with PHP in mind so some of the rules inside are not
applicable everywhere, but most are pretty general.

### Contributing
If you have a real world use case that requires modification or exception in
this guide, please file an issue. If you don't have a real world use case at
least prepare a very strong argumentation, based on experience, otherwise the
issue is likely to be closed. Suggestions are welcome - if they conform to my
views.

### Exceptions from this guide
The general rule is make things look
[beautiful](https://github.com/Perennials/articles/blob/master/Good-better-best-programming-practices.md)
and consistent. Make things readable, not a brainfuck. Exceptions are allowed
but only with specific and clear intention, not out of laziness. Try not
to conflict with the overall style of the project you are working on.

Examples
--------

### Basic sanity

* One class per file.
* Simple classes and methods for narrow purpose.
* Max 500 lines per file, size around 200 lines is usually much more pleasant to
  manage.
* Avoid many level of indentation, 3-4 levels seems reasonable.
* Avoid long function bodies, consider splitting them if longer than 20-30 lines.
* Don't repeat yourself.

### Spacing and new lines

* Indent with tabs, never mix tabs and spaces.
* Spaces around binary operators, after commas, be generous with spacing.
* Space after statements like `if`, `for`, `while`, after function declarations,
  inside parenthesis.
* No one-liners, always curly brackets.
* Opening brackets on the same line.
* Closing brackets on separate line.

```php
//// BAD

if(!$rh)
{
    if($this instanceof IRequestHandler)
    {
	$this->setRequestHandler($this);
	$rh = $this->getRequestHandler();
    }
    else return null;
}

//// GOOD

if ( $rh === null ) {

	if ( $this instanceof IRequestHandler ) {
		$this->setRequestHandler( $rh = $this );
	}
	else {
		return null;
	}
}
```

### Naming

Exceptions may apply if you are designing something with the purpose of making
the typing of a common repetitive task easier (e.g. you are making jQuery for
PHP) or if it is a specific design choice to make the code more expressive,
(e.g. `$db->INSERT()`, `$db->UPDATE()` which all generate a SQL query and
intend to make the code look more SQLish).

* No underscores in names, exceptions may apply when the intention is to make
  something look ugly and standout on purpose.
* Files containing global scope scripts are in lowercase like `init.php`.
* Files containing classes match the name of the class like `MyClass.php`.
* `ClassNamesLikeThis`, `Not_Like_This`, `NOTLikeThis`; `MySql` not `MySQL`,
  `UserId` not `UserID`.
* `$object->method()`, `$object->methodName()`, `$object->getMyStuff()`,
  `$object->setMyStuff()`.
* `$private->_properties`, `$with->_leadingUnderscore`.
* `$protected->properties`, `$without->leadingUnderscore`.
* `$public->Properties`, `$starting->WithUppercase`.
* `GlobalFunction()`.
* `CONSTANT_NAME`.

### Global scope rules

* No scripting in the global scope unless unavoidable.
* Don't pollute the global variable namespace and unset your stuff.

  ```php
  //// BAD

  $my_temporary_global_stuff = 5;
  DoSomethingWith( $my_temporary_global_stuff );

  //// GOOD
  
  $_my_temporary_global_stuff = 5;
  DoSomethingWith( $_my_temporary_global_stuff );
  unset( $_my_temporary_global_stuff );
  ```

* Don't use global variables, ever. Use static class members if necessary.

### Variable scoping

```php
//// BAD

if ( true ) {
	$var = 5;
}
else {
	$var = 6;
}

$var += 2;

//// GOOD

$var = 0;

if ( true ) {
	$var = 5;
}
else {
	$var = 6;
}

$var += 2;
```

### Comparisons

* Use strong type checks unless the types are known and the same.
  
  ```php
  //// BAD

  if ( $a == 'string' ) {}
  
  //// GOOD
  
  if ( is_string( $a ) && $a == 'string' ) {}
  if ( 'asd' == 'qwe' ) {}
  if ( $a === 'string' ) {}

  ```

* Use `instanceof`.
 
  ```php
  //// BAD

  if ( get_class( (object)null ) == 'stdClass' ) {}
  
  //// GOOD
  
  if ( (object)null instanceof stdClass ) {}
  ```

* No comparisons between different types with implicit conversation.
  
  ```php
  //// BAD

  if ( '1' == 1 ) {}
  if ( 0 == null ) {}

  //// GOOD
  
  if ( (int)'1' == 1 ) {}
  if ( 0 === null ) {}
  ```

### Array and property access

* No undefined array index or property access, but not to imply that every
  array/property access should have a check.
 
  ```php
  //// BAD
  
  $arr = [ 1, 2, 3 ];
  $a = $arr['NO-SUCH-KEY'];

  $obj = new stdClass;
  $a = $obj->noSuchKey;
  
  //// GOOD
  
  $arr = [ 1, 2, 3 ];
  $a = array_key_exists( 'NO-SUCH-KEY', $arr ) ? $arr['NO-SUCH-KEY'] : null;
  
  $obj = new stdClass;
  $a = property_exists( $obj, 'noSuchKey' ) ? $obj->noSuchKey : null;
  ```

### Empty values

* Use `null` to signify empty value, not empty string, not zero.

  ```php
  //// BAD

  if ( $a == '' ) {}

  //// GOOD
  
  if ( $a === null ) {}
  ```

### Class member visibility

* Use protected class members only when the class should be extended by
  design, otherwise use private members and make them protected when/if the
  class needs to be extended.
* No direct access to other classes' variables within the same library. Use
  getters/setters. But for convenience or performance reasons, or with
  specific design intention, direct access to public variables is permissible
  for the user of the library.
* Public visibility is default for methods, so omit the `public` keyword, this
  is PHP, not Java.

  ```php
  //// BAD
  
  class MyClass {

  	public function someMethod () {
  	}

  }
  
  //// GOOD
  
  class MyClass {

  	function someMethod () {
  	}

  }
  ```

### Conditions

* Make conditions expressive, so the intention is clear.
* Avoid implicit conversations to bool and be explicit.

  ```php
  //// BAD

  // not clear intention, "not age" means nothing
  if ( !$age ) {
  }

  // not clear intention, "not length" means nothing
  if ( !$string->length ) {
  }

  // not clear intention; are we interested to check
  // if the value is empty string, or if it is zero,
  // or if it is defined or something else?
  if ( !$object->property ) {
  }

  //// GOOD

  // clear intention
  if ( $age > 0 ) {
  }

  // clear intention
  if ( $string->length > 0 ) {
  }
  
  // clear intention
  if ( property_exists( $object, 'property' ) ) {
  }

  ```

### Error checks, conditions with long if-bodies

* Prefer positive conditions over negative ones.
* Avoid long if-bodies when possible.

```php
//// BAD

function MyFunction () {

	if ( !error ) {
		// ...
		// ...
		// ...
		// ...
		// ...
		// ...
		// ...
		// ...
		// many lines of code
	}

}

//// GOOD

function MyFunction () {

	if ( error ) {
		return;
	}
	
	// ...
	// ...
	// ...
	// ...
	// ...
	// ...
	// ...
	// ...
	// many lines of code

}
```

### Language abuse, performance considerations

Don't too be clever when there is simple way to achieve it. Usualy there is no
added benefit, not for readability neither for performance.

```php
//// BAD

foreach ($ProvCodeList as $ProvCode)
{
	if (in_array($ProvCode->Type, array('City', 'CityArea')))
	{
		$CityCodes[] = $ProvCode->Code;
	}
}

//// GOOD

foreach ( $ProvCodeList as $ProvCode ) {
	$type = $ProvCode->Type;
	if ( $type === 'City' || $type === 'CityArea' ) {
		$CityCodes[] = $ProvCode->Code;
	}
}
```

```js
//// BAD

jQuery.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type[ "[object " + name + "]" ] = name.toLowerCase();
});

//// GOOD

var class2type = {
	'[object Boolean]': 'boolean',
	'[object Number]': 'number',
	'[object String]': 'string',
	'[object Function]': 'function',
	'[object Array]': 'array',
	'[object Date]': 'date',
	'[object RegExp]': 'regexp',
	'[object Object]': 'object',
	'[object Error]': 'error'
};
```

### Doc comments

* In [jsdocgen](https://github.com/Perennials/jsdocgen) format.
* Empty line before and after the description.
* Every sentence ends with a punctuation mark (e.g. dot), including titles,
  function parameters and everything else that is a human text and not email
  for example.
* Consistent line length.
* Always document functions that are meant for reuse, otherwise they should be private.
* Document complex logic with one line tips to describe the intention.

```php
/**
jsdocgen format comments.
  
Not as pretty as when lined up with stars, but quick
to write and stars don't mess things when editing. So
one is more inclined to write docs.
  
@return bool And put dots everywhere, this is not IRC.
*/
```

<!--
```php
//// BAD

/**
 * phpDocumentor style stars on every line.
 * They do look good but are not keyboard friendly and
 * and discourage documentation.
 * 
 * @param $zendStyle Parameter repetition is even less welcome.
 */

//// GOOD

/**
jsdocgen format comments.
  
Not as pretty as when lined up with stars, but quick
to write and stars don't mess things when editing. So
one is more inclined to write docs.
  
@return bool And put dots everywhere, this is not IRC.
*/
```
-->

Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)
