# Lumen ALA

[![Latest Version](https://img.shields.io/github/v/release/kiwfy/ala-microservice.svg?style=flat-square)](https://github.com/kiwfy/ala-microservice/releases)
[![codecov](https://codecov.io/gh/kiwfy/ala-microservice/branch/master/graph/badge.svg)](https://codecov.io/gh/kiwfy/ala-microservice)
[![Build Status](https://img.shields.io/circleci/build/github/kiwfy/ala-microservice/master?label=CI%20Build&token=34d8b3820b7229d742897f0a6982ced5bf6a99c8)](https://github.com/kiwfy/ala-microservice)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

API Rest Full based in lumen using query builder that auto generate base code for simple crud (with unit tests and feature tests)

Requires [PHP](https://php.net) 8.1.

### Installation

(optional) Stop all other containers to avoid conflict.

```sh
docker stop $(docker ps -qa)
```

Start project with Docker using compose tool.

```sh
docker-compose up -d
```

Access the container

```sh
docker exec -it ala-microservice-php bash
```

Run [Composer](https://getcomposer.org/) to install all dependencies.

```sh
composer install --no-dev --prefer-dist
```

Ensure the folder ./storage are with all rights to save log and cache
```sh
chmod -R 777 ./storage
```

To see what is build for this project look at docker -> dev folder.

Create .env file

```sh
cp .env.example .env
```

Put key value in `APP_KEY` and `JWT_APP_SECRET`.

**You can use `/health/key` uri to generate this keys or use another value if you want.**

Using [Postman](https://www.postman.com/downloads/) to consulting the routes created and put the new routes.

`ala_microservice.postman_collection.json` 
`ala_microservice.postman_environment.json`

Or you can access [Here](http://localhost:8101).

### Automatic CRUD

For create a new Domain with a complete CRUD use the command:

```sh
php artisan create:domain {YOU_DOMAIN_NAME_HERE}
```

This command create another folder in `app/Domains`, new file in `routes`, `database/migrations` and `database/seeds` folder.

**If your domain name has 2 words use underline (_) to separate.**

All your test unit and feature about your new domain already created to.

### Configure new Domain

- Configure your migration file in `database/migrations`
- Open your domain and configure your fields and the ordenations in `app/Domains/YOUR_DOMAIN/Http/Parameters`
- Your validator rules in `app/Domains/YOUR_DOMAIN/Http/Validators`
- All your businesses you put in `app/Domains/YOUR_DOMAIN/Businesses`
- Your routes in `bootstrap/{YOUR_DOMAIN}_routes` folder

### Ulid

For primary key value, this project using [Ulid](https://github.com/kiwfy/ulid-php) value, but you can pass other pattern in Insert route if you don't want to use this type of value.

You can use the validate reserved word `ulid` in `app/Domains/YOUR_DOMAIN/Http/Validators` folder. Config in `app/Providers/AppServiceProvider.php`

For example:

```php
/**
 * get rules for this request
 * @return array
 */
public function getRules(): array
{
    return [
        'another_id' => 'required|ulid',
    ];
}
```

### JWT

In auth route this projet use [JWT](https://github.com/kiwfy/jwt-manager-php) lib. This token will be generate if your secret, token and context is correct. This configuration stay in [Config](https://github.com/kiwfy/ala-microservice/blob/master/config/token.php) folder.

```php
return [
    'data' => [
        //Token
        '32c5a206ee876f4c6e1c483457561dbed02a531a89b380c3298bb131a844ac3c' => [
            // Context
            'name' => 'app-test',
            // Secret
            'secret' => 'a1c5930d778e632c6684945ca15bcf3c752d17502d4cfbd1184024be6de14540',
        ],
    ],
];
```

### Request Service

To make request between two or more services, this project use [Request Service](https://github.com/kiwfy/request-service-php) lib.

### Response

The pattern used to return all request is json and the layout is configure in your [Response](https://github.com/kiwfy/response-json-php) lib.

### Filters

Follow this steps to configure a new field to accepted a filter in list route

- In validator folder `app/Domains/YOUR_DOMAIN/Http/Validators` configure de list rules `{YOU_DOMAIN_NAME}ListValidator`. For example:

Configure a `name` field.

```php
/**
 * get rules for this request
 * @return array
 */
public function getRules() : array
{
    return [
        'class' => 'string|in:"asc","desc"',
        'fields' => 'string',
        'order' => 'string',
        'page' => 'integer|min:1',
        'filter_name' => 'string|filter',
    ];
}
```

After that, you need to finish in `app/Domains/YOUR_DOMAIN/Filters`.

The parameter accept equal, not equal and like query.

**To see another types look at `FiltersTypesConstants` class in `app/Constants`.**

```php
/**
 * set filter rules for this domain
 */
public $filter = [
    'name' => [
        'validate' => 'string|min:3',
        'permissions' => [
            FiltersTypesConstants::FILTER_EQUAL,
            FiltersTypesConstants::FILTER_NOT_EQUAL,
            FiltersTypesConstants::FILTER_LIKE,
        ],
    ],
    'created' => [
        'validate' => 'date',
        'permissions' => [
            FiltersTypesConstants::FILTER_LESS_THAN,
            FiltersTypesConstants::FILTER_GREATER_THAN,
            FiltersTypesConstants::FILTER_GREATER_THAN_OR_EQUAL,
            FiltersTypesConstants::FILTER_LESS_THAN_OR_EQUAL,
        ],
    ],
    'modified' => [
        'validate' => 'date',
        'permissions' => [
            FiltersTypesConstants::FILTER_LESS_THAN,
            FiltersTypesConstants::FILTER_GREATER_THAN,
            FiltersTypesConstants::FILTER_GREATER_THAN_OR_EQUAL,
            FiltersTypesConstants::FILTER_LESS_THAN_OR_EQUAL,
        ],
    ],
];
```

After that you can send this param in url query, for example:

`/{YOUR_DOMAIN}/list?filter_name=lik,vitor` OR `/{YOUR_DOMAIN}/list?filter_name=eql,vitor`.

**To see the reservate works look at `FiltersTypesConstants` class in `app/Constants`.**

### Recomendations

Use this project with [MySql](https://www.mysql.com/) with no relationship keys and NOT use JOIN.

In this way you can use all database maturity with as fast as possible.

Use [Clear Linux](https://clearlinux.org/) image in your PHP container to get more 50% speed and 50% less memory.

### Production

Don't forget to change `APP_ENV` to `production` value. Don't use that in develop mode because this parameter cache all your project.

The production docker is located in `docker/prod` and you can change the Nginx config or PHP if you want.

### Development

Want to contribute? Great!

The project using a simple code.
Make a change in your file and be careful with your updates!
**Any new code will only be accepted with all viladations.**

To ensure that the entire project is fine:

First install all the dev dependences

```sh
composer install --dev --prefer-dist
```

Second run all validations
```sh
composer checkall
```

You can run all validations plus test coverage metrics
```sh
composer checkallcover
```

**Kiwfy - Open your code, open your mind!**
