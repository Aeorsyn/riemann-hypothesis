---
title: "Laravel Filters: Route access depending on the user group"
url: "https://bosnadev.com/2014/09/18/laravel-filters-route-access-depending-on-the-user-group"
author: "Mirza Pašić"
date: "2014-09-18"
topic: "Laravel"
tags: ["routing", "authorization", "middleware"]
summary: "A Laravel 4 route filter, used as inGroup:Editor;Admin, that lets only users in the listed Sentry groups reach a route or route group."
---

# Laravel Filters: Route access depending on the user group

> Outdated: Written for Laravel 4 in 2014. Route filters are gone; current Laravel uses middleware.

Filters are one of the Larevel’s features that I like the most. It’s pretty straightforward – you need to run some chunk of code before or after some action. Recently I had to implement restrictions on route resources depending on user group. I use Sentry for user authentication and role management and this Laravel filter should check if user belongs to specified group and allow him access to route resource if he does.

One solution is to simply create a new filter for every group. But, I don’t like repetition. I wanted simple and elegant solution that meets my requirements, without unnecessary repetition of code. Here is solution I came up with:

```php
Route::filter('inGroup', function($route, $request, $value) {
    $values = explode( ';' , $value );
    $user = Sentry::getUser();
    $status = false;
    foreach ( $values as $val ) {
        $group = Sentry::findGroupByName($val);
        if ( $user->inGroup($group) ) {
            $status = true;
        }
    }
    if ( $status == false ) {
        return Redirect::route('index')->with('error' , 'You have no access to this resource!');
    }
});
```

Now, you just need to add filter ‘inGroup:Editor;Admin’ to route or route group you want and only users with specified Sentry groups can access to these resources. In this case we allow access only to users from Editor and Admin groups:

```php
Route::group([ 'before' => 'inGroup:Editor;Admin' ] , function() {
        Route::resource('posts' , 'PostController' );
        Route::resource('agencies' , 'AgencyController');
        Route::resource('hotels' , 'HotelController');
});
```

Or if you want to use only on one route and allow access to users from Admin and SuperAdmin groups:

```php
Route::get('users', ['before' => 'inGroup:Admin;SuperAdmin',
  'uses' => '[email protected]',
  'as' => 'users.list'
]);
```

For more information about laravel filters, you can check [official documentation](http://laravel.com/docs/routing#route-filters "Laravel Filters").
