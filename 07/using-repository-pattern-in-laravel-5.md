---
title: "Using Repository Pattern in Laravel 5"
url: "https://bosnadev.com/2015/03/07/using-repository-pattern-in-laravel-5"
author: "Mirza Pašić"
date: "2015-03-07"
topic: "Laravel"
tags: ["repository-pattern", "architecture", "eloquent", "design-patterns"]
summary: "Building a repository pattern package for Laravel 5: a repository interface, an abstract Eloquent repository, and criteria classes for custom queries."
---

# Using Repository Pattern in Laravel 5

> Outdated: Written for Laravel 5.0 in 2015.

These days there is a lot of buzz about software design patterns, and one of the most frequently asked questions is “How can I use _some pattern_ with _some technology_ “. In the case of Laravel and the Repository pattern, I see often questions like _**“ How I can use repository pattern in Laravel 4”**_ or nowadays _\*\* “..in Laravel 5”\*\*_. Important thing you must remember is that **design patterns do not depend on specific technology, framework or programming language**.

Contents

- 1 Introduction
- 2 It’s all about interfaces
- 3 Directory structure
- 4 A Repository Implementation
- 5 Criteria Queries
  - 5.1 Creating A New Criteria
  - 5.2 Using Criteria In The Controller
- 6 Package Installation
- 7 Conclusion
- 8 Credits

## Introduction

If you have really understood Repository Pattern then it does not matter what framework or programming language you are going to use. What is important is that you understand the **principle** behind the **Repository pattern**. Then you can implement it in whatever technology you want. With that in mind, let’s start with the [definition](http://martinfowler.com/eaaCatalog/repository.html "Repository Pattern Definition") of the Repository pattern:

A Repository mediates between the domain and data mapping layers, acting like an in-memory domain object collection. Client objects construct query specifications declaratively and submit them to Repository for satisfaction. Objects can be added to and removed from the Repository, as they can from a simple collection of objects, and the mapping code encapsulated by the Repository will carry out the appropriate operations behind the scenes.

**Repository pattern** separates the data access logic and maps it to the business entities in the business logic. Communication between the data access logic and the business logic is done through interfaces.

![Using Repository Pattern with Laravel 5](https://media.bosnadev.com/hub/a13fc7ed0ee765fe/2015-03-repository-pattern-776x300.webp)

To put it simply, **Repository pattern** is a kind of container where data access logic is stored. It hides the details of data access logic from business logic. In other words, we allow business logic to access the data object without having knowledge of underlying data access architecture.

The separation of data access from business logic have many benefits. Some of them are:

- Centralization of the data access logic makes code easier to maintain
- Business and data access logic can be tested separately
- Reduces duplication of code
- A lower chance for making programming errors

## It’s all about interfaces

**Repository pattern** is all about interfaces. An interface acts like a contract which specify what an concrete class must implement. Let’s think a little bit. If we have two data objects _**Actor**_ and _**Film**_ , what are common set of operations that can be applied to these two data objects? In most situations we want to have the following operations:

1. Get all records
2. Get paginated set of records
3. Create a new record
4. Get record by it’s primary key
5. Get record by some other attribute
6. Update a record
7. Delete a record

Can you see now how much duplicated code would we have if we implement this for each data object? Sure, for small projects it’s not a big problem, but for large scale applications it’s a bad news.

Now when we have defined common operations, we can create an interface:

```php
interface RepositoryInterface {

    public function all($columns = array('*'));

    public function paginate($perPage = 15, $columns = array('*'));

    public function create(array $data);

    public function update(array $data, $id);

    public function delete($id);

    public function find($id, $columns = array('*'));

    public function findBy($field, $value, $columns = array('*'));
}
```

## Directory structure

Before we continue with creating concrete repository class that will implement this interface, let’s think a bit how we want to organise our code. Usually, when I create something, I like to think component way since I want to be able to reuse that code in other projects. My simple directory structure for the repositories component looks like this:

![Repository - Directory Structure](https://media.bosnadev.com/hub/5b625b8a56fe6eea/2015-03-repository-directory-structure.webp)But it can be different, for example if component have configuration options, or migrations, etc.

Inside src directory I have three other directories: **Contracts** , **Eloquent** and **Exceptions**. As you can see, the folder names are pretty convenient for what we want to put there. In **Contracts** folder we put interfaces, or contracts as we call them earlier. **Eloquent** folder contains abstract and concrete repository class that implements contract. In **Exceptions** folder we put exceptions classes.

Since we are creating a package we need to create composer.json file where we define a mapping for namespaces to specific directories, package dependencies and other package metadata. Here is the content of composer.json for this package:

```json title="composer.json"
{
  "name": "bosnadev/repositories",
  "description": "Laravel Repositories",
  "keywords": [
    "laravel",
    "repository",
    "repositories",
    "eloquent",
    "database"
  ],
  "licence": "MIT",
  "authors": [
    {
      "name": "Mirza Pasic",
      "email": "[email protected]"
    }
  ],
  "require": {
    "php": ">=5.4.0",
    "illuminate/support": "5.*",
    "illuminate/database": "5.*"
  },
  "autoload": {
    "psr-4": {
      "Bosnadev\\Repositories\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "Bosnadev\\Tests\\Repositories\\": "tests/"
    }
  },
  "extra": {
    "branch-alias": {
      "dev-master": "0.x-dev"
    }
  },
  "minimum-stability": "dev",
  "prefer-stable": true
}
```

As you can see, we mapped namespace Bosnadev\Repository to the src directory. Another thing, before we start to implement _RepositoryInterface,_ since it is located in the **Contracts** folder, we need to set correct namespace for it:

```php
<?php namespace Bosnadev\Repositories\Contracts;

interface RepositoryInterface {

...

}
```

We are now ready to start with the implementation of this contract.

## A Repository Implementation

Using repositories enables us to query the data source for the data, map the data to a business entity and persist changes in the business entity to the data source:

![Repository Pattern](https://media.bosnadev.com/hub/1df36c31b7bd095a/2015-03-concrete-repositories.webp)Of course, each concrete child repository should extend our abstract repository, which implements _RepositoryInterface_ contract. Now, how would you implement this contract? Take a look at first method. What can you tell about it just by looking at it?

First method in our contract is conveniently named all() . It’s duty is to fetch all records for the concrete entity. It accepts only one parameter $columns which must be an array. This parameter is used, as its name suggests, to specify what columns we want to fetch from the data source, and by default we fetch them all.

For specific entity, this method could look like this:

```php
public function all($columns = array('*')) {
    return Bosnadev\Models\Actor::get($columns);
}
```

But we want to make it generic, so we can use it wherever we want:

```php diff
 public function all($columns = array('*')) {
-    return Bosnadev\Models\Actor::get($columns);
+    return $this->model->get($columns);
 }
```

In this case $this->model is an instance of Bosnadev\Models\Actor . Thus, somewhere in the repository we need to create a new instance of the given model. Here is one solution how you can implement this:

```php
<?php namespace Bosnadev\Repositories\Eloquent;

use Bosnadev\Repositories\Contracts\RepositoryInterface;
use Bosnadev\Repositories\Exceptions\RepositoryException;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Container\Container as App;

/**
 * Class Repository
 * @package Bosnadev\Repositories\Eloquent
 */
abstract class Repository implements RepositoryInterface {

    /**
     * @var App
     */
    private $app;

    /**
     * @var
     */
    protected $model;

    /**
     * @param App $app
     * @throws \Bosnadev\Repositories\Exceptions\RepositoryException
     */
    public function __construct(App $app) {
        $this->app = $app;
        $this->makeModel();
    }

    /**
     * Specify Model class name
     *
     * @return mixed
     */
    abstract function model();

    /**
     * @return Model
     * @throws RepositoryException
     */
    public function makeModel() {
        $model = $this->app->make($this->model());

        if (!$model instanceof Model)
            throw new RepositoryException("Class {$this->model()} must be an instance of Illuminate\\Database\\Eloquent\\Model");

        return $this->model = $model;
    }
}
```

Since we declared class as abstract, it means it must be extended by concrete child class. By declaring model() method as abstract we force the user to implement this method in the concrete child class. For example:

```php
<?php namespace App\Repositories;

use Bosnadev\Repositories\Contracts\RepositoryInterface;
use Bosnadev\Repositories\Eloquent\Repository;

class ActorRepository extends Repository {

    /**
     * Specify Model class name
     *
     * @return mixed
     */
    function model()
    {
        return 'Bosnadev\Models\Actor';
    }
}
```

Now we can implement the rest of the contract methods:

```php
<?php namespace Bosnadev\Repositories\Eloquent;

use Bosnadev\Repositories\Contracts\RepositoryInterface;
use Bosnadev\Repositories\Exceptions\RepositoryException;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Container\Container as App;

/**
 * Class Repository
 * @package Bosnadev\Repositories\Eloquent
 */
abstract class Repository implements RepositoryInterface {

    /**
     * @var App
     */
    private $app;

    /**
     * @var
     */
    protected $model;

    /**
     * @param App $app
     * @throws \Bosnadev\Repositories\Exceptions\RepositoryException
     */
    public function __construct(App $app) {
        $this->app = $app;
        $this->makeModel();
    }

    /**
     * Specify Model class name
     *
     * @return mixed
     */
    abstract function model();

    /**
     * @param array $columns
     * @return mixed
     */
    public function all($columns = array('*')) {
        return $this->model->get($columns);
    }

    /**
     * @param int $perPage
     * @param array $columns
     * @return mixed
     */
    public function paginate($perPage = 15, $columns = array('*')) {
        return $this->model->paginate($perPage, $columns);
    }

    /**
     * @param array $data
     * @return mixed
     */
    public function create(array $data) {
        return $this->model->create($data);
    }

    /**
     * @param array $data
     * @param $id
     * @param string $attribute
     * @return mixed
     */
    public function update(array $data, $id, $attribute="id") {
        return $this->model->where($attribute, '=', $id)->update($data);
    }

    /**
     * @param $id
     * @return mixed
     */
    public function delete($id) {
        return $this->model->destroy($id);
    }

    /**
     * @param $id
     * @param array $columns
     * @return mixed
     */
    public function find($id, $columns = array('*')) {
        return $this->model->find($id, $columns);
    }

    /**
     * @param $attribute
     * @param $value
     * @param array $columns
     * @return mixed
     */
    public function findBy($attribute, $value, $columns = array('*')) {
        return $this->model->where($attribute, '=', $value)->first($columns);
    }

    /**
     * @return \Illuminate\Database\Eloquent\Builder
     * @throws RepositoryException
     */
    public function makeModel() {
        $model = $this->app->make($this->model());

        if (!$model instanceof Model)
            throw new RepositoryException("Class {$this->model()} must be an instance of Illuminate\\Database\\Eloquent\\Model");

        return $this->model = $model->newQuery();
    }
}
```

Pretty easy, right? Only thing left now is to inject _ActorRepository_ in the _ActorsController_ , or our business side of application:

```php
<?php namespace App\Http\Controllers;

use App\Repositories\ActorRepository as Actor;

class ActorsController extends Controller {

    /**
     * @var Actor
     */
    private $actor;

    public function __construct(Actor $actor) {

        $this->actor = $actor;
    }

    public function index() {
        return \Response::json($this->actor->all());
    }
}
```

## Criteria Queries

As you can imagine, these basic actions are just enough for simple querying. For larger applications you’ll most definitely need to make some custom queries to fetch more specific data set defined by some criteria.

To achieve this, we begin with defining what child (clients) criteria must implement. In other words, we’ll create an abstract non instantiable class with just one method in it:

```php
<?php namespace Bosnadev\Repositories\Criteria;

use Bosnadev\Repositories\Contracts\RepositoryInterface as Repository;
use Bosnadev\Repositories\Contracts\RepositoryInterface;

abstract class Criteria {

    /**
     * @param $model
     * @param RepositoryInterface $repository
     * @return mixed
     */
    public abstract function apply($model, Repository $repository);
}
```

This method will hold criteria query which will be applied in the _Repository_ class on the concrete entity. We also need to extend our _Repository_ class a bit to cover criteria queries. But first, let’s create a new contract for the _Repository_ class:

```php
<?php namespace Bosnadev\Repositories\Contracts;

use Bosnadev\Repositories\Criteria\Criteria;

/**
 * Interface CriteriaInterface
 * @package Bosnadev\Repositories\Contracts
 */
interface CriteriaInterface {

    /**
     * @param bool $status
     * @return $this
     */
    public function skipCriteria($status = true);

    /**
     * @return mixed
     */
    public function getCriteria();

    /**
     * @param Criteria $criteria
     * @return $this
     */
    public function getByCriteria(Criteria $criteria);

    /**
     * @param Criteria $criteria
     * @return $this
     */
    public function pushCriteria(Criteria $criteria);

    /**
     * @return $this
     */
    public function  applyCriteria();
}
```

Now we can extend functionality of our _Repository_ class by implementing _CriteriaInterface_ contract:

```php
<?php namespace Bosnadev\Repositories\Eloquent;

use Bosnadev\Repositories\Contracts\CriteriaInterface;
use Bosnadev\Repositories\Criteria\Criteria;
use Bosnadev\Repositories\Contracts\RepositoryInterface;
use Bosnadev\Repositories\Exceptions\RepositoryException;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Collection;
use Illuminate\Container\Container as App;

/**
 * Class Repository
 * @package Bosnadev\Repositories\Eloquent
 */
abstract class Repository implements RepositoryInterface, CriteriaInterface {

    /**
     * @var App
     */
    private $app;

    /**
     * @var
     */
    protected $model;

    /**
     * @var Collection
     */
    protected $criteria;

    /**
     * @var bool
     */
    protected $skipCriteria = false;

    /**
     * @param App $app
     * @param Collection $collection
     * @throws \Bosnadev\Repositories\Exceptions\RepositoryException
     */
    public function __construct(App $app, Collection $collection) {
        $this->app = $app;
        $this->criteria = $collection;
        $this->resetScope();
        $this->makeModel();
    }

    /**
     * Specify Model class name
     *
     * @return mixed
     */
    public abstract function model();

    /**
     * @param array $columns
     * @return mixed
     */
    public function all($columns = array('*')) {
        $this->applyCriteria();
        return $this->model->get($columns);
    }

    /**
     * @param int $perPage
     * @param array $columns
     * @return mixed
     */
    public function paginate($perPage = 1, $columns = array('*')) {
        $this->applyCriteria();
        return $this->model->paginate($perPage, $columns);
    }

    /**
     * @param array $data
     * @return mixed
     */
    public function create(array $data) {
        return $this->model->create($data);
    }

    /**
     * @param array $data
     * @param $id
     * @param string $attribute
     * @return mixed
     */
    public function update(array $data, $id, $attribute="id") {
        return $this->model->where($attribute, '=', $id)->update($data);
    }

    /**
     * @param $id
     * @return mixed
     */
    public function delete($id) {
        return $this->model->destroy($id);
    }

    /**
     * @param $id
     * @param array $columns
     * @return mixed
     */
    public function find($id, $columns = array('*')) {
        $this->applyCriteria();
        return $this->model->find($id, $columns);
    }

    /**
     * @param $attribute
     * @param $value
     * @param array $columns
     * @return mixed
     */
    public function findBy($attribute, $value, $columns = array('*')) {
        $this->applyCriteria();
        return $this->model->where($attribute, '=', $value)->first($columns);
    }

    /**
     * @return \Illuminate\Database\Eloquent\Builder
     * @throws RepositoryException
     */
    public function makeModel() {
        $model = $this->app->make($this->model());

        if (!$model instanceof Model)
            throw new RepositoryException("Class {$this->model()} must be an instance of Illuminate\\Database\\Eloquent\\Model");

        return $this->model = $model->newQuery();
    }

    /**
     * @return $this
     */
    public function resetScope() {
        $this->skipCriteria(false);
        return $this;
    }

    /**
     * @param bool $status
     * @return $this
     */
    public function skipCriteria($status = true){
        $this->skipCriteria = $status;
        return $this;
    }

    /**
     * @return mixed
     */
    public function getCriteria() {
        return $this->criteria;
    }

    /**
     * @param Criteria $criteria
     * @return $this
     */
    public function getByCriteria(Criteria $criteria) {
        $this->model = $criteria->apply($this->model, $this);
        return $this;
    }

    /**
     * @param Criteria $criteria
     * @return $this
     */
    public function pushCriteria(Criteria $criteria) {
        $this->criteria->push($criteria);
        return $this;
    }

    /**
     * @return $this
     */
    public function  applyCriteria() {
        if($this->skipCriteria === true)
            return $this;

        foreach($this->getCriteria() as $criteria) {
            if($criteria instanceof Criteria)
                $this->model = $criteria->apply($this->model, $this);
        }

        return $this;
    }
}
```

### Creating A New Criteria

With criteria queries, you can now organise your repositories more easily. Your repositories do not need to be thousands of lines long.

![Criteria Structure](https://media.bosnadev.com/hub/882f63ce0f18d04c/2015-03-criteria-structure-321x155.webp)

Your criteria class can look like this:

```php
<?php namespace App\Repositories\Criteria\Films;

use Bosnadev\Repositories\Contracts\CriteriaInterface;
use Bosnadev\Repositories\Contracts\RepositoryInterface as Repository;
use Bosnadev\Repositories\Contracts\RepositoryInterface;

class LengthOverTwoHours implements CriteriaInterface {

    /**
     * @param $model
     * @param RepositoryInterface $repository
     * @return mixed
     */
    public function apply($model, Repository $repository)
    {
        $query = $model->where('length', '>', 120);
        return $query;
    }
}
```

### Using Criteria In The Controller

Now when we have simple criteria, let’s see how we can use it. There is a two ways how you can apply the criteria on the repository. First is by using pushCriteria() method:

```php
<?php namespace App\Http\Controllers;

use App\Repositories\Criteria\Films\LengthOverTwoHours;
use App\Repositories\FilmRepository as Film;

class FilmsController extends Controller {

    /**
     * @var Film
     */
    private $film;

    public function __construct(Film $film) {

        $this->film = $film;
    }

    public function index() {
        $this->film->pushCriteria(new LengthOverTwoHours());
        return \Response::json($this->film->all());
    }
}
```

This method is useful if you need to apply multiple criteria, you can stack them as you wish. However, if you need to apply just one criteria, you can use getByCriteria() method:

```php
<?php namespace App\Http\Controllers;

use App\Repositories\Criteria\Films\LengthOverTwoHours;
use App\Repositories\FilmRepository as Film;

class FilmsController extends Controller {

    /**
     * @var Film
     */
    private $film;

    public function __construct(Film $film) {

        $this->film = $film;
    }

    public function index() {
        $criteria = new LengthOverTwoHours();
        return \Response::json($this->film->getByCriteria($criteria)->all());
    }
}
```

## Package Installation

You can install this package by adding this dependency in your composer require section:

```json
"bosnadev/repositories": "0.*"
```

and just run composer update afterwards.

## Conclusion

Using repositories in your application have multiple benefits. From basic things like reducing code duplication and preventing you to make programming errors to making you application easier to extend, test and maintain.

From architectural point of view you managed to separate concerns. Your controller doesn’t need to know how and where you store the data. Simple and beautiful. Abstract.

You can find this package on [Github](https://github.com/Bosnadev/Repositories "Bosnadev Repositories"), where you can check for latest updates and bug fixes. I also plan to add new features like eager loading, caching and some configs so stay tuned by staring the repository. However, if you want to contribute in the development just fork the repository and send PR.

If you have any thoughts or suggestions, please let me know in the comment section bellow. See ya.

## Credits

This package is largely inspired by [this](https://github.com/prettus/l5-repository) great package by @andersao. [Here](https://github.com/anlutro/laravel-repository/) is another package I used as reference. Also, I find these articles very helpful:

[Creating flexible Controllers in Laravel 4 using Repositories](http://culttt.com/2013/07/08/creating-flexible-controllers-in-laravel-4-using-repositories/)\
[Laravel Repository Pattern](https://web.archive.org/web/20161121115232/http://vegibit.com/laravel-repository-pattern/)\
[The Repository Pattern in Action](https://web.archive.org/web/20151225015552/http://ryantablada.com/post/the-repository-pattern-in-action)\
[Laravel – Using Repository Pattern](http://heera.it/laravel-repository-pattern)
