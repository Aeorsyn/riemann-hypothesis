---
title: "Laravel: Caching Database Queries"
url: "https://bosnadev.com/2014/12/27/laravel-caching-database-queries"
author: "Mirza Pašić"
date: "2014-12-27"
topic: "Laravel"
tags: ["eloquent", "caching", "performance"]
summary: "Caching database queries in Laravel 4 with remember(), the Cache facade for raw queries, and cache sections and tags that can be flushed on their own."
---

# Laravel: Caching Database Queries

> Outdated: Written for Laravel 4 in 2014. The remember() query method was removed in Laravel 5.

If your application runs a large number of queries very often, with time it will become very, very sluggish. Here Laravel caching comes handy. Laravel provides a simple mechanism for caching these queries using a very simple chained method call. Here is an example using Laravel’s Fluent Query Builder:

```php
$users = DB::table('users')
           ->orderBy('latest_activity', 'desc')
           ->take(10)
           ->remember(60)
           ->get();
```

Of course, we can do the same thing using Eloquent:

```php
$users = User::orderBy('latest_activity', 'desc')
             ->take(10)
             ->remember(60)
             ->get();
```

Behind the scene Laravel executes the query and then stores it along with the query result using the cache adapter, with an expiration time of 60 minutes. Running the same query again will result that cached query will be found, which means it will not be executed again, instead the results will be taken from the cache.

Contents

- 1 What About More Complex Queries?
- 2 Cache Sections
  - 2.1 Accessing Items In A Section Cache
- 3 Cache Tags
  - 3.1 Accessing Items In A Tagged Cache

## What About More Complex Queries?

If you have much more complex queries, and you use raw queries, you can cache them to, using [Cache Facade](http://laravel.com/docs/4.2/facades "Laravel Facades"):

```php
$usersTable = Cache::remember('usersTable', 60, function()
{
    return  DB::table('users')
        ->select(DB::raw(
            "SOME COMPLEX JOINS ETC.."
        ))->get();
});
```

Make sure to check Laravel [official documentation](http://laravel.com/docs/4.2/cache "Laravel Cache"), to see what other methods you have at disposal.

Of course, if you have such a huge application that requires caching, you will probably use this snippet in a multiple places across entire application. That’s why is better to create a generic helper method which will then handle caching:

```php
public function cacheQuery($key, $sql, $timeout = 60) {
    return Cache::remember($key, $timeout, function() use ($sql) {
        return DB::select(DB::raw($sql));
    });
}

$cache = $this->cacheQuery("usersTable", "SOME COMPLEX JOINS ETC..", 30);
```

**UPDATE:** A better solution for handling cache keys is to make MD5 hash from SQL Query, as suggested in this [Stackoverflow answer](http://stackoverflow.com/a/26214974/975850 "Laravel Caching Complex Queries"), therefore previous method will look something like this:

```php {1-2,7}
public function cacheQuery($sql, $timeout = 60) {
    return Cache::remember(md5($sql), $timeout, function() use ($sql) {
        return DB::select(DB::raw($sql));
    });
}

$cache = $this->cacheQuery("SOME COMPLEX JOINS ETC..", 30);
```

**UPDATE 2:** Thank you all for your feedback and for making this article much better

## Cache Sections

**Note:** Cache sections are not supported when using the **file** or **database** cache drivers.

Cache sections were first introduced in Laravel 4.0. With cache sections we got a possibility to bundle a specific dataset in the cache. Flushing all of the cached data is done simply by calling section’s name. On the official documentation page I couldn’t find any reference on how to use cache sections, but fortunately there is an Api Documentation for the [Cache Section](https://web.archive.org/web/20150514131230/http://laravel.com/api/4.2/Illuminate/Cache/Section.html "Cache Section") class.

Here we can see pretty standard set of methods that we can use. Caching queries in the sections can be done in following way:

```php
function cacheQuery($key, $timeout = 60) {
    return Cache::section('departments')->remember($key, $timeout, function() {
        return Department::all()->toArray();
    });
}

cacheQuery("all-departments", $timeout = 60);
```

After you open the page that execute this query, result of the query will be stored in the cache and you’ll get expected result.

### Accessing Items In A Section Cache

To access a section cache use section name used to save it.

```php
$cached_section = Cache::section("departments")->get('all-departments');

var_dump($cached_section);
```

I created simple route, just to demonstrate this:

![Query Caching](https://media.bosnadev.com/hub/d5c7b737e79691a2/2014-12-caching-query-section.webp)Now, if we reload the same page again we see the same result, but no query was executed whatsoever. That’s because this dataset is fetched from the cache.

![Cached Query](https://media.bosnadev.com/hub/72c1ffdaa32ad33c/2014-12-cached-query-section.webp)Now, if you make any change in these Departments, if you delete or add new ones, these changes will not be visible for another 60 minutes (that is expiration time I set in this example). This is not an good option in a real production-ready applications, because you want to flush cache whenever Model is changed.

## Cache Tags

Laravel 4.1 version has brought us a lot of new features. One of these are [**cache tags**](http://laravel.com/docs/4.2/cache#cache-tags "Cache Tags"). Now it is possible to tag individually cached objects and flush these separately from everything else in our application cache.

**Note:** Same as with _**Cache Sections**_ , _**Cache Tags**_ are not supported when using the **file** or **database** cache drivers.

As documentations states, you can store a tagged cache by passing in an ordered list of tag names as arguments, or as an ordered array of tag names:

```php
function cacheTags($tags, $depKey, $sql, $timeout = 60) {
    return Cache::tags("employees", "managers")->remember($depKey, $timeout, function() use($sql) {
        // Some join query that will return all Managers from selected Department
        return DB::select(DB::raw($sql));
    });
}
```

Now you can use this helper method as follows:

```php
$sql = "SOME COMPLEX LEFT JOIN..";
cacheTags(["employees", "managers"], "departments_support", $sql);

$sql = "ANOTHER COMPLEX LEFT JOIN..";
cacheTags(["employees", "developers"], "departments_support", $sql);

$sql = "...";
cacheTags(["employees", "managers"], "departments_it", $sql);

$sql = "...";
cacheTags(["employees", "developers"], "departments_it", $sql);
```

 

### Accessing Items In A Tagged Cache

Similar as we did to access a Section Cache, to access items in a tagged cache we pass the tag or the same ordered list of tags used to save it:

```php
$d = Cache::tags(["employees", "managers"])->get('departments_it');

var_dump($d);
```

Voilà! Here is the result:

![Tagged Cache](https://media.bosnadev.com/hub/50a3e47e7e6f3107/2014-12-tagged-cache-result.webp)I hope that you have learned something new and that now you have a basic idea how to use this concept of caching in your Laravel application. Stay tuned for the next article where I’ll explain how to use Model observer to invoke Cache::flush() when Model is changed.

If you have any questions or suggestions feel free to drop a comment in a section bellow.
