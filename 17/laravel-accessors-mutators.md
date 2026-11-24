---
title: "Laravel Accessors and Mutators"
url: "https://bosnadev.com/2015/12/17/laravel-accessors-mutators"
author: "Mirza Pašić"
date: "2015-12-17"
topic: "Laravel"
tags: ["eloquent", "json"]
summary: "Accessors and mutators on Eloquent models in Laravel 5.1, used to capitalize names, format dates, combine attributes, hash passwords and store JSON."
---

# Laravel Accessors and Mutators

> Outdated: Written for Laravel 5.1 in 2015. Laravel 9 added a new Attribute syntax for accessors and mutators.

Laravel [accessors and mutators](http://laravel.com/docs/5.1/eloquent-mutators#accessors-and-mutators) are custom, user defined methods that allow you to format Eloquent attributes.  _**Accessors** are used to format attributes when you retrieve them from the database, while **mutators** format the attributes before saving them to the database._

Contents

- 1 Defining an accessor
- 2 Defining an mutator
- 3 More examples
- 4 Full example
- 5 Conclusion

## Defining an accessor

Syntax for defining an accessor is getFooAttribute() where Foo is capitalized attribute you want to access. If your attribute is _first_name_ , and you want to be sure that first name will always be camel cased when you fetch it, you’ll need to define an accessor function as follows:

```php
public function getFirstNameAttribute($value)
{
    return ucfirst($value);
}
```

For sure, that’s one of the most commonly used examples when you read about accessors, but you can use it to format dates, prices etc.

Let’s say you have a _Member_ model with following fields:

- first_name
- last_name
- email
- password
- last_visit
- settings
- created_at
- updated_at

We have few fields where we can take an advantage of using accessors. For example, users are often careless and you want to make sure that first name and last name are always capitalized.

To create a _Member_ , you would probably use code similar to this (and yes, we’ll simulate user input here):

```php
$user = App\Member::create([
    'first_name'  => 'mirza',
    'last_name'   => 'pasic',
    'email'       => '[email protected]',
    'password'    => '!supersecretpassword!',
    'last_login'  => Carbon\Carbon::now(),
    'settings'    => ['two_factor_aut' => false, 'session_time' => 1200],
    'created_at'  => Carbon\Carbon::now(),
    'updated_at'  => Carbon\Carbon::now()
]);
```

Even if I was lazy member and didn’t care to properly capitalize my own name, you would probably still want to display my name correctly. Right? Accessors do just that:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Class Member
 * @package App
 */
class Member extends Model
{
    /**
     * Make sure that first name is always capitalized when retrieved from the database
     *
     * @param $value
     * @return string
     */
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }

    /**
     * Make sure that last name is always capitalized when retrieved from the database
     *
     * @param $value
     * @return string
     */
    public function getLastNameAttribute($value)
    {
        return ucfirst($value);
    }
}
```

You can also use accessors to combine two real attributes to create just one. In this example, it’s easy to imagine what I’m referring to – full name. You can create accessor method like this:

```php
    /**
     * Get members full name
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return ucfirst($this->first_name) . ' ' . ucfirst($this->last_name);
    }
```

Then you would use it as if it really exists in the database:

```php
$member = App\Member::find(1);

echo $member->full_name;
```

You can play it with the dates to. With accessors you can chose the how the dates should be displayed. In most european countries date format is different than in the US. With accessors you can show the date in the format that is familiar to your users. For example:

```php
    /**
     * Custom format for the last login date
     *
     * @param $value
     * @return string
     */
    public function getLastLoginAttribute($value)
    {
        return \Carbon\Carbon::parse($value)->format('d.m.Y.');
    }
```

When you call last_login attribure, you’ll have following output:

```php
$member = App\Member::find(1);
echo $member->last_login;

// 15.12.2015.
```

## Defining an mutator

Syntax for defining a mutator function is setFooAttribute() where Foo is a camel cased column you want to access. So, once again, let’s use our first_name and last_name columns, but this time we want to make change before saving it to the database:

```php
    /**
     * Make sure that first name is capitalized BEFORE saving it to the database
     *
     * @param $value
     * @return string
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = ucfirst($value);
    }

    /**
     * Make sure that last name is capitalized BEFORE saving it to the database
     *
     * @param $value
     * @return string
     */
    public function setLastNameAttribute($value)
    {
        $this->attributes['last_name'] = ucfirst($value);
    }
```

You probably notice that here we don’t return anything. We directly access the attribute and set a new value. With this approach we can be sure that first and last name will always be capitalized.

## More examples

I’ve shown you just a few very basic examples of accessor and mutator functions. But you can do a lot more and I’ll give you a few other situations where you can use them.

##### Hashing passwords

I’m sure you want to encrypt passwords when you save them into database, right? Mutators are really good way to ensure this:

```php
public function setPasswordAttribute($value) {
    $this->attributes['password'] = Hash::make($value);
}
```

This way you know that password attribute will be always hashed before it’s inserted into DB. Here I used [Laravel’s Hash](http://laravel.com/docs/5.0/hashing#basic-usage) facade to create password hash, but you could also use bcrypt() helper method.

##### Json Encoding

If you look at our _Member_ model once again, you’ll notice settings attribute which is used to store users personal preferences and options you provide in your app. Of course, you would use an array to manage those options and one of the best options to store an array to the database is to encode it as an JSON string. Some of the databases have long history of JSON data types, such as PostgreSQL, first with [json](https://www.postgresql.org/docs/current/static/datatype-json.html), and with latest relase we got new superfast [jsonb](https://www.postgresql.org/docs/9.4/static/datatype-json.html) datatype. As of MySQL 5.7.8, MySQL introduced [json](https://dev.mysql.com/doc/refman/5.7/en/json.html) data type so (even) if you use MySQL you can still store JSON objects into it.

For the settings attribute you should have both accessor to decode JSON string to an array and mutator to encode an array to JSON object:

```php
/**
 * Make sure that we get an array from JSON string
 *
 * @param $value
 * @return array
 */
public function getSettingsAttribute($value) {
    return json_decode($value, true);
}

/**
 * Encode an array to a JSON string
 *
 * @param $value
 */
public function setSettingsAttribute($value)
{
    $this->attributes['settings'] = json_encode($value);
}
```

## Full example

At the end, here is a complete Member model I’ve used in this article:

```php
<?php  namespace App;

use Illuminate\Support\Facades\Hash;
use Illuminate\Database\Eloquent\Model;

/**
 * Class Member
 * @package App
 */
class Member extends Model
{
    /**
     * @var array
     */
    protected $fillable = ['first_name', 'last_name', 'email', 'password', 'last_login', 'settings', 'created_at', 'updated_at'];

    /**
     * Make sure that first name is always capitalized when retrieved from the database
     *
     * @param $value
     * @return string
     */
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }

    /**
     * Make sure that last name is always capitalized when retrieved from the database
     *
     * @param $value
     * @return string
     */
    public function getLastNameAttribute($value)
    {
        return ucfirst($value);
    }

    /**
     * Get members full name
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return ucfirst($this->first_name) . ' ' . ucfirst($this->last_name);
    }

    /**
     * Custom format for the last login date
     *
     * @param $value
     * @return string
     */
    public function getLastLoginAttribute($value)
    {
        return \Carbon\Carbon::parse($value)->format('d.m.Y.');
    }

    /**
     * Make sure that first name is capitalized BEFORE saving it to the database
     *
     * @param $value
     * @return string
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = ucfirst($value);
    }

    /**
     * Make sure that last name is capitalized BEFORE saving it to the database
     *
     * @param $value
     * @return string
     */
    public function setLastNameAttribute($value)
    {
        $this->attributes['last_name'] = ucfirst($value);
    }

    /**
     * Make sure that password is encrypted
     *
     * @param $value
     */
    public function setPasswordAttribute($value) {
        $this->attributes['password'] = Hash::make($value);
    }

    /**
     * Make sure that we get an array from JSON string
     *
     * @param $value
     * @return array
     */
    public function getSettingsAttribute($value) {
        return json_decode($value, true);
    }

    /**
     * Encode an array to a JSON string
     *
     * @param $value
     */
    public function setSettingsAttribute($value)
    {
        $this->attributes['settings'] = json_encode($value);
    }

}
```

and how to use it:

```php
Route::get('members', function() {
    $user = App\Member::create([
        'first_name'  => 'mirza',
        'last_name'   => 'pasic',
        'email'       => '[email protected]',
        'password'    => '!supersecretpassword!',
        'last_login'  => Carbon\Carbon::now(),
        'settings'    => ['two_factor_aut' => true, 'session_time' => 1200],
        'created_at'  => Carbon\Carbon::now(),
        'updated_at'  => Carbon\Carbon::now()
    ]);

    $member = App\Member::find(1);
    echo $member->last_login . '<br>';
    echo $member->full_name . '<br>';
});
```

## Conclusion

I really hope that I helped you understand what accessors and mutattors are, and if you don’t already use them I hope you now have an idea for what you can use them in your projects.
