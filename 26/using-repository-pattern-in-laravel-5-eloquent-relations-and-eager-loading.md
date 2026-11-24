---
title: "Using Repository Pattern In Laravel 5: Eloquent Relations And Eager Loading"
url: "https://bosnadev.com/2015/03/26/using-repository-pattern-in-laravel-5-eloquent-relations-and-eager-loading"
author: "Mirza Pašić"
date: "2015-03-26"
topic: "Laravel"
tags: ["repository-pattern", "eloquent", "performance", "design-patterns"]
summary: "The N+1 query problem, how eager loading in Eloquent avoids it, and a with() method that adds eager loading to the Laravel 5 repository classes."
---

# Using Repository Pattern In Laravel 5: Eloquent Relations And Eager Loading

> Outdated: Written for Laravel 5.0 in 2015.

Contents

- 1 Preface
- 2 What Is Eager Loading?
- 3 Extending Repository Class
- 4 Conclusion

## Preface

Before we go to the main topic of the article, I’ll give you a short heads up for some design problems you may face. Recently one of my clients complained that some pages open very slowly. When I say very, I mean incredibly slow. So I’ve decided to debug that page and what I saw shocked me. Query section was showing that on that page was executed an staggering **16500+ queries** !!

I found troubling part of the code responsible for that. It was 3 foreach loops, querying an attribute and it’s related attributes. It worked fine, until there was about 5.5k items in the database. Basically, here is what’s happening:

```php
$main_object = MainObject::all();

foreach($main_object as $object) {
    echo $object->some_property;

    foreach($object->related_object as $related) {
        echo $related->some_property;
        echo $related->another_property;
    }

    foreach($object->another_related as $another) {
        echo $another->some_property;
        echo $another->another_property;
    }
}
```

So if $main_object = MainObject::all(); returns 5.5k results, first foreach will run 5.5k times, second also 5.5k and third same as much. By using ORM developers very often make the mistake of writing very inefficient database queries, and ORM makes them even more difficult to spot. This problem is known as **N+1 problem**. And I guess previous developer was unaware of that. To avoid this problem we use eager loading.

## What Is Eager Loading?

To put it simple, **eager loading** is a method of doing everything when asked. It’s also exactly opposite from[**lazy loading**](https://en.wikipedia.org/wiki/Lazy_loading "Lazy  Loading") when we execute tasks when needed. Eager loading helps us to avoid common performance pitfalls, as you saw in my example above. You’ll understand it better through an example so let’s imagine the following situation:

![Stores Model](https://media.bosnadev.com/hub/0a50d4eda729f675/2015-03-stores-model-650x371.webp)

we have an [Enhanced Entity Relationship](https://en.wikipedia.org/wiki/Enhanced_entity%E2%80%93relationship_model "Enhanced Entity Relationship Model") model (EER) with three entities each related to another. We can read the EER as follows: _Each member can have**many** stores, but one store belongs to **only one** member. Each store can have **many** products, but one product belongs to **only one** store_.

Next step is to create an Eloquent models for these entities.

**Member** :

```php
<?php namespace App;

use Illuminate\Database\Eloquent\Model;

class Member extends Model {

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['username', 'email', 'first_name', 'last_name'];

    public function stores() {
        return $this->hasMany('App\\Store');
    }
}
```

**Store** :

```php
<?php namespace App;

use Illuminate\Database\Eloquent\Model;

class Store extends Model {

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'slug', 'site', 'member_id'];

    public function  member() {
        return $this->belongsTo('App\\Member');
    }

    public function products() {
        return $this->hasMany('App\\Product');
    }
}
```

and **Product** :

```php
<?php namespace App;

use Illuminate\Database\Eloquent\Model;

class Product extends Model {

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name', 'short_desc', 'long_desc', 'price', 'store_id', 'member_id'];

    public function  store() {
        return $this->belongsTo('App\\Store');
    }
}
```

Imagine that you are building an application that will allow your users to create their own store. Of course, like in every other store, users are able to create multiple products. Also, we want to create one page that will display all stores and top products from each store. Something like this:

![Store List Mockup](https://media.bosnadev.com/hub/9b977c5a7dbd0077/2015-03-storelist-mockup-726x781.webp)

You may end up with something like this in your controller:

```php
<?php namespace App\Http\Controllers;

use App\Repositories\StoreRepository;

class StoresController extends Controller {

    protected $stores;

    function __construct(StoreRepository $stores) {
        $this->stores = $stores;
    }

    public function index() {
        $stores = $this->stores->all();

        return \View::make('stores.index')->with('stores', $stores);
    }
}
```

in the View you want to display that data:

```blade
@foreach($stores as $store)
    <h1>{{ $store->name }}</h1>
    <span>Owner: {{ $store->member->first_name . ' ' . $store->member->last_name }}</span><br>

    <h2>Products:</h2>
    @foreach($store->products as $product)
        <h3>{{$product->name}}</h3>
        <span>{{$product->short_desc}}</span><br/><br/>
        <span>Price: {{$product->price}}</span>
        <br/>
        <?php Debugbar::info('Product displayed'); ?>
    @endforeach
    <br/>
    ========================
    <br/>
@endforeach
```

And the result is: ![N Plus One Problem](https://media.bosnadev.com/hub/d1f39134a46c4dce/2015-03-n-plus-one-problem.webp)

For this example, I’ve seeded the database with 5 members, 3 stores and 4 products. First query is to get all stores from the database and that is **+1** part of [**N+1 problem**](https://web.archive.org/web/20160913084230/https://secure.phabricator.com/book/phabcontrib/article/n_plus_one/ "N+1 Problem"). In this specific case, **N** represents a number of stores returned from the first query, since that many times we’ll execute select \*from query on the _**products**_ and _**members**_ table. Since we have 3 stores, that means we’ll make 3 queries on the members table and 3 additional queries on the products table. In total we executed 3+3+1 queries.

Now imagine what would happen if we had 5000 or 10000 stores? You would have 10-20k queries executed every time when some user visits that page. And what if you have 10k or 100k visits in 24h? Nightmare! It’s clear now that this approach is a **performance killer**. No matter what DB you use, how powerful your server is it will always reach the tipping point where your powerful hardware will no longer be your ally. You can try to improve performance by caching these queries, using Redis for example. And it will work just fine, but just temporarily. That way you only delay the inevitable end, which will cost you a lot of money, time and probably you’ll lose some client(s) or your userbase will be reduced significantly.

Here eager loading come to rescue. Using [eager loading](http://laravel.com/docs/4.2/eloquent#eager-loading "Eager Loading") in Laravel is pretty simple. The relations you want to be eager loaded you specify in the with method:

```php
$stores = Store::with('member','products')->get();
```

Now, instead executing 7 queries, by using eager loading we reduced number of queries to just 3:

![N Plus One Problem](https://media.bosnadev.com/hub/1ae8293f9196f309/2015-03-n-plus-one-problem-eager-736x128.webp)

And it’ll be just three queries even if there is a 10k store entries. As you can see, wise use of eager loading may drastically increase the performance of your application. To actually have an improvement, we also need to have an index on the id fields in the _**members**_ and _**products**_ table. With a ton of records, executing in( '1', '2', ... ) on non-indexed fields can take a while.

After this quick introduction into eager loading, let’s see how we can use the relations with repositories.

## Extending Repository Class

I’ll show you one way how you can use relations in the concrete repository classes. Here is an example of end result:

```php
function __construct(StoreRepository $stores) {
    $this->stores = $stores;
}

public function index() {
    $stores = $this->stores->with('member', 'products')->all();

    ....
}
```

As you can see, we have a with method where we can chain our model relations. This method will be similar to the [Laravel’s Query Builder](https://github.com/laravel/framework/blob/5.0/src/Illuminate/Database/Eloquent/Builder.php#L734 "Laravel Query Builder") with method.

```php
public function with($relations) {
    if (is_string($relations)) $relations = func_get_args();

    $this->with = $relations;

    return $this;
}
```

Now we need to attach each provided relation to the model:

```php
protected function eagerLoadRelations() {
    if(!is_null($this->with)) {
        foreach ($this->with as $relation) {
            $this->model->with($relation);
        }
    }

    return $this;
}
```

And that’s it, only thing left is to update our all() repository method (and any other you want) to use eager loading:

```php
public function all($columns = array('*')) {
    $this->applyCriteria();
    $this->newQuery()->eagerLoadRelations();
    return $this->model->get($columns);
}
```

As I have already mentioned, you can add multiple relations within the with() method. Here is an example of StoresController :

```php diff
 <?php namespace App\Http\Controllers;

 use App\Repositories\StoreRepository;

 class StoresController extends Controller {

     protected $stores;

     function __construct(StoreRepository $stores) {
         $this->stores = $stores;
     }

     public function index() {
-        $stores = $this->stores->all();
+        $stores = $this->stores->with('member', 'products')->all();

         return \View::make('stores.index')->with('stores', $stores);
     }
 }
```

In the **View** you can display data however you want, for testing purposes this is just enough:

```blade
@foreach($stores as $store)
    <h1>{{ $store->name }}</h1>
    <span>Owner: {{ $store->member->first_name . ' ' . $store->member->last_name }}</span><br>

    <h2>Products:</h2>
    @foreach($store->products as $product)
        <h3>{{$product->name}}</h3>
        <span>{{$product->short_desc}}</span><br/><br/>
        <span>Price: {{$product->price}}</span>
        <br/>
        <?php Debugbar::info('Product displayed'); ?>
    @endforeach
    <br/>
    ========================
    <br/>
@endforeach
```

As expected, we now have only three queries:

![Eager Loading](https://media.bosnadev.com/hub/946086add87d9735/2015-03-eager-loading-699x143.webp)

## Conclusion

By using eager loading you can improve your application performance. Sometimes as your application grows, even eager loading is not enough to keep up top performance. In the next tutorial I’ll show you how you can decorate repositories to cache the queries for better performance.
