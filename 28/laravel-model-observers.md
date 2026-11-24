---
title: "Laravel: Model Observers"
url: "https://bosnadev.com/2014/12/28/laravel-model-observers"
author: "Mirza Pašić"
date: "2014-12-28"
topic: "Laravel"
tags: ["eloquent", "caching", "events"]
summary: "Using Eloquent model observers in Laravel 4 to flush cached queries whenever a model is saved or deleted, following on from the query caching tutorial."
---

# Laravel: Model Observers

> Outdated: Written for Laravel 4 in 2014.

In the last tutorial I showed you how you can speed up a web application by caching database queries that are executed very often. However, there is a problem. Application will cache the resulted dataset, and in the future this dataset will be served. You need to tell your application to flush the cache every time when certain Model is updated. That way every time you edit, delete or add new Model item, old cache will be flushed, and query will be executed again. Afterwards, resulting new dataset will be cached and served until Model is updated once again.

Here Laravel [Model Observers](http://laravel.com/docs/4.2/eloquent#model-observers "Eloquent Model Observers") come in rescue. You can use them for numerous things. One of practical examples for using Model Observers is cache flushing.

### Eloquent model

Let’s start with the model, here is basic Department model:

```php
<?php namespace Bosnadev\Models;

class Department extends \Eloquent {
    protected $table = 'departments';

    protected $primaryKey = 'dept_no';

    public $timestamps = false;
}
```

One of the places where you can put your observers is Eloquent boot method. We need to override the default boot method to add our observers:

```php {3,17-21}
<?php namespace Bosnadev\Models;

use Bosnadev\Observers\DepartmentObserver;

class Department extends \Eloquent {
    protected $table = 'departments';

    protected $primaryKey = 'dept_no';

    public $timestamps = false;

    public static function table() {
        $model = new static;
        return $model->getTable();
    }

    public static function boot() {
        parent::boot();

        Department::observe(new DepartmentObserver());
    }
}
```

### Laravel Model Observers

Some of the code can be abstracted for later implementation, so we create _**AbstractionObserver**_ class first:

```php
<?php namespace Bosnadev\Observers;

use \Cache;

abstract class AbstractObserver {

    protected function clearCacheTags($tags) {
        Cache::tags($tags)->flush();
    }

    protected function clearCacheSections($section) {
        Cache::section($section)->flush();
    }

    abstract public function saved($model);

    abstract public function saving($model);

    abstract public function deleted($model);

    abstract public function deleting($model);
}
```

Now we can implement this class to create any observer we need. For this purpose, let’s create one Department observer which will flush cache every time when **Department** model is updated.

```php
<?php namespace Bosnadev\Observers;

class DepartmentObserver extends AbstractObserver {
    public function saved($model)
    {
        $this->clearCacheSections($model->getTable());
    }
    public function deleted($model) {
        $this->clearCacheSections($model->getTable());
    }
}
```

Only thing left now is to cache some data. Here is a simple controller which handles IT Department resources, employees and everything else related to it:

```php
<?php namespace Bosnadev\Controllers;

use BaseController;
use Bosnadev\Models\Department;
use DB;

class ITDepartmentController extends BaseController {
    public function index() {
        $it_department = \Cache::section(Department::table())->remember("departments-it", 60, function() {
            $sql = "
            SELECT employees.emp_no, employees.first_name, employees.last_name, employees.birth_date, employees.gender, employees.hire_date,
              departments.dept_name, departments.dept_no
            FROM employees
            INNER JOIN dept_emp
              ON employees.emp_no = dept_emp.emp_no
            AND dept_emp.dept_no = 'd005'
            INNER JOIN departments
              ON dept_emp.dept_no = departments.dept_no
            LIMIT 10 OFFSET 0;
            ";
            return DB::select(DB::raw($sql));
        });

        // Here you can do whatever you want with $it_department, but let's check the cache
        var_dump(\Cache::section(Department::table())->get("departments-it"));
    }
}
```

By opening route _departments/it_ we can see that one query is executed:

![Laravel Model Observers - IT Department Cache Complex Queries](https://media.bosnadev.com/hub/69c0a1657c07aa0e/2014-12-itdepartment-query-cache.webp)Refreshing the same page gives the same result, but no query is executed.

![Laravel Model Observer - Cached Query for IT Department](https://media.bosnadev.com/hub/30db4b0c5cf70772/2014-12-itdepartment-query-cached.webp)OK. Now when we know that caching is working, how about making some changes in the Department model. What if we want to rename Development into, let’s say – DevOps.

I created _DepartmentsApiController_ with basic **CRUD** operations, here you can see how update() method looks like, and what is result of **PUT** request:

![Laravel Model Observers - Departments API Controller](https://media.bosnadev.com/hub/b20a0f6724cef967/2014-12-departments-api-controller-update-method-1024x563.webp)Let’s refresh our _departments/it_ route once again:

Query is executed once again and new dataset is cached for later use. This query will not be executed until new change in _Department_ model triggers _DepartmentObserver_ or until cache expires, which is for 1 hour.

I hope you now understand basic principle behind **Model Observers** and how this powerful feature can be used for other things that require notification on Model update.
