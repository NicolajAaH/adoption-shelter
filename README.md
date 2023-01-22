# Assignment 2: Pet Shelter

In the previous assignment, you created a minimal version of *itslearning*, where you learned how to build your own web application using MVC. In this application, you will implement authentication and authorization features in another application: a pet shelter board.

The pet shelter board is an application where registered users can post pets they are giving in for adoption. Moreover, registered users can also adopt pets posted by other users.

You must follow the instructions **to the letter**, your application will be tested automatically by the same system of the previous assignment (Please be patient, we are improving the system right now). This time, we are going to provide you with a base application, where all the main components that you learned from the previous assignment are already implemented. Your job is to implement the components in charge of the authentication and authorization features.

This assignment can be done individually or in groups of two people. Because of the time constraint of the assignment and [pair programming practices](https://en.wikipedia.org/wiki/Pair_programming), we heavily encourage you to work in groups. To create groups, you have to use the group feature in our automatic testing application (*ie,* it is not necessary to create groups in itslearning). If you want to work in pairs but you do not know anyone else in the course who wants that, you can use the dedicated channel in Discord to look for a partner.

<span style="color:red">**Disclaimer:** Unless otherwise instructed, do not in any way, modify the contents of the `/tests` directory or the `.gitlab-ci.yml` file. Doing so will be considered cheating, and will in the best case result in your assignment being failed.</span>
**We have seen projects that changed some of these files in the previous assignment. For this assignment, we are going to be more strict.**

## Setup

1. Clone your project locally.
2. Run `composer install` to install php dependencies.
3. Create a copy of the .env.example file named .env. This can be done with the command `cp .env.example .env`
4. Run `php artisan key:generate` to generate a random encryption key for your application
5. Run `php artisan serve` to boot up your application

### The project

In this assignment, you are given an already coded application, where you have to fill up the gaps in the source code. This means,it is not necessary for you to create any file, although you can create files if you think they are needed for your application to pass the tests. The different files you need to modify to pass this assignment are listed in its correspondent section.

### The database
The project requires a connection to a database. Luckily, thanks to docker, this is extremely simple and platform agnostic. To spin up a MySQL server, simply run the `docker-compose up -d` within the directory. This will pull a MySQL server, port-forward it to port 3306 on your machine, and start it in detached mode. 

Additionally, we have included an installation of _phpmyadmin_ that you can use to explore the database (this will start as part of the docker command), simply go to [http://localhost:8036](http://localhost:8036) and you should see something like this:

![](db-explorer.png)
(if the database is empty, you haven't migrated it yet)

You are of course still free to use whichever tool you prefer.

The connection to the database is defined as follows:
- host: `localhost`
- port: `3306`
- username: `root`
- password: `secret`
- database: `adoption`

If you followed the steps mentioned earlier and copied your `.env.example` to `.env`, then Laravel should already be configured with the correct connection details.

_Hint: your JetBrains Student subscription comes bundled with __DataGrip__, which can be used to explore your database._

### Relevant commands

- `php artisan migrate` - This will synchronize your database structure to your migrations (read more [here](https://laravel.com/docs/8.x/migrations#introduction)), these can be viewed under `database/migrations`. Laravel comes bundled with some by default, which you can either ignore or delete.
- `php artisan migrate:fresh` - Deletes everything within your database and starts the migration from scratch, very useful during development.
- `php artisan migrate:fresh --seed` - Deletes everything within your database and starts the migration from scratch, and seeds the database with some dummy data and cute animals.
- `php artisan make:controller {name of Controller}` - This creates a controller with a specified name. Controllers in Laravel use a singular noun with the `Controller` suffix (HomeController, UserControler... e.g.)
- `php artisan make:model {name of model}` - Creates a model with a specified name (usually singular (User, House, Apartment, Animal...))
- `php artisan make:model {name of model} -mr` - Allows us to create a model with a given name, as well as a controller for it and a migration.
- `php artisan serve` - Starts the development server for the application.

### Testing your solution

Every time you push your code to our source control (gitlab.sdu.dk) (which you will have to do to pass), your code will be validated to see if it meets the requirements of this assignment. This can be slow, especially if other people are also doing it simultaneously (then you will most likely be put in a queue). To mitigate this, you can run your tests locally. 

#### Running browser tests

You should run our browser tests using Laravel Dusk.

The first time you run the tests on your machine, you will have to install the latest `Chrome` binaries; this can be done with the `php artisan dusk:chrome-driver` command (make sure you have the latest version of chrome).

In another terminal, run `php artisan serve` - this is needed as dusk actively uses the server to test your implementation. Make sure the server is up and running every time you test your implementation.

In your main terminal, run: `php artisan dusk` and `php artisan test` - this will start running your tests.

### Debugging Screenshots

The tests are by default running in sequentiel order with the ``php artisan dusk` command. However, you can specify a filter to test specific functionality of the application with the php artisan dusk --filter <test-name> parameter - these are also provided for each section of the assignment

Naturally, when the tests are running, the developer doesn't have any visual insight of progress of the test, unless using the `--browse` argument is included. However, a screenshot will be generated in the folder: Tests/Browser/Screenshots/..., which might be helpful in debugging the current failing test.

## Logic

### Base application

As we mentioned, you are given a base pet shelter application. This application should be modified to integrate authentication and authorization features. In the original application, users can: 
1. See a list of every pet given for adoption, 
2. Give pets for adoptions
3. Adopt pets. 

As you can see, these are all the features you learned from the previous assignment.

The application has two models: User and Adoption. Currently, User is used to saving the people who are giving a pet for adoption, and to knowing which person is adopting a pet. The Adoption model contains the information necessary to give for adoption a pet: its name and a description. Moreover, it contains the foreign keys listed_by and adopted_by, which are used to link the current adoption to the user giving for adoption and to the user adopting the pet, respectively. You can see the ER diagram of the database here:

![](er.png)

We also provided two controllers: Home and Adoption. The Home controller is in charge of the Home page (of course) and the authentication logic (sign in, sign up, etc). The Adoption controller is in charge of all the Adoption logic (index, show adoptions, adopt a pet, etc). You need to modify these controllers as instructed.

### Route overview

The following routes are created for the pet shelter application:

| URL                          | Method | Controller         | Description                                                  |
|------------------------------|--------|--------------------|--------------------------------------------------------------|
| /                            | GET    | HomeController     | Shows home page                                              |
| /adoptions                   | POST   | AdoptionController | Creates a new listing for an adoption                        |
| /adoptions/create            | GET    | AdoptionController | Displays the form that creates a new listing for an adoption |
| /adoptions/mine              | GET    | AdoptionController | Lists the pets that you have adopted                         |
| /adoptions/{adoption}        | GET    | AdoptionController | Shows the details for a given {adoption}                     |
| /adoptions/{adoption}/adopt  | POST   | AdoptionController | Allows you to adopt a given {adoption}                       |
| /login                       | GET    | HomeController     | Shows the login page                                         |
| /login                       | POST   | HomeController     | Processes the login request and logs the user in             |
| /logout                      | GET    | HomeController     | Logs out the current authenticated user                      |
| /register                    | GET    | HomeController     | Displays the form that creates a new user                    |
| /register                    | POST   | HomeController     | Creates a new user                                           |



#### Tests

`php artisan test` tests if the user is unable to adopt their own pets by posting to the adopt route.
