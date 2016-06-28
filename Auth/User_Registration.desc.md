## Objectives

* Explain cryptographic password hashing
* Explain why password hashing is important
* Explain what a salt is
* Explain why using a salt is important
* Explain the process of user registration
* Use knex migration to create a users table
* Add custom routes for user registration
* Use bcrypt to hash and salt passwords

## What is cryptographic password hashing?

**Password hashing** is a security process that takes a user-submitted password and encrypts it, producing an encoded string at a fixed length. Cryptographic hash functions are intended to be one-way functions, meaning it is supposed to be unfeasible to calculate the input based on an encoded string's output.

## Why is password hashing important?

User information is far more vulnerable if passwords are stored in plain-text when a database is compromised, so it's highly recommended to store them as encoded strings. Using defined encryption algorithms, hashing helps protect against hash tables, dictionary attacks, rainbow tables, and other types of brute-force attacks.

Password hashing also provides an extra layer of security between developer and end user. When passwords are stored as cryptographic hashes, the developer never has access to a user's plain-text password.

Some of the properties of a successful hashing function include:

* The process of hashing is quick and inexpensive for any input
* It is impractical for a hash to be the same for two different inputs
* It is impractical to calculate an input based on a hash
* Even small changes to an input will result in a completely new hash that is uncorrelated to a previous hash

### Exercise

Write down in your own words why cryptographic hashing is important. Once you're finished, we'll discuss what you wrote.

## What is a salt?

Before a password is run through a hashing function, it is recommended to add another layer of protection with something called a salt. A **salt** is a random data string that is concatenated to input before it's hashed.

## Why is using a salt important?

Using a salt provides extra security since different random data is generated each time for each password. This prevents attackers from potentially using a table of precomputed hashes of common passwords to gain access.

### Exercise

Turn to your neighbor and discuss what a salt is and how it adds another layer of security before hashing a password. We'll regroup and I'll pick students at random to share what you talked about.

## What is the user registration process?

1. User submits form with various data required to register
1. Database is queried to see if unique information already exists, such as email
1. If not, plain-text password is concatenated with random generated salt
1. Plain-text password + salt is then run through hashing function, which returns an encoded string at a fixed length (hash)
1. Hash is inserted along with user information into database

## Create migrations for users table

To start off with, change into your `trackify` directory and make a new branch called `registration`.

```shell
cd PATH_TO/trackify
git checkout -b registration
```

Open the project directory with atom if you haven't already, and create the migration file to define the schema for the users table.

```shell
atom .
npm run knex migrate:make users
```

Inside the newly created `TIMESTAMP_users.js` migration file, add the following:

```JavaScript
'use strict';

exports.up = function(knex) {
  return knex.schema.createTable('users', (table) => {
    table.increments();
    table.string('email').unique().notNullable().defaultTo('');
    table.string('password').notNullable().defaultTo('');
    table.timestamps(true, true);
  });
};

exports.down = function(knex) {
  return knex.schema.dropTable('users');
};
```

Add and commit your work.

```shell
git add .
git commit -m "create migration file for users table"
```

## Add user registration route

Create a new file for the users routes and open it with Atom.

```shell
touch routes/users.js
atom routes/users.js
```

In the `server.js` file, add the necessary routing middleware for the `/users` endpoint:

```JavaScript
const usersRoutes = require('routes/users');

app.use('/users', usersRoutes);
```

After requiring the necessary dependencies for an Express router and knex, add a `POST` route to the `'/'` path in the `users.js` file. Save the incoming data from `req.body` as constants for the email and password.

```JavaScript
'use strict';

const express = require('express');
const router = express.Router();

const environment = process.env.NODE_ENV || 'development';
const knexConfig = require('../knexfile')[environment];
const knex = require('knex')(knexConfig);

router.post('/', (req, res, next) => {
  const email = req.body.email;
  const password = req.body.password;

  // Check if email already exists in database
});
```

Add and commit your work.

```shell
git add .
git commit -m "add register user route"
```

In the route handler, before hashing the password or inserting data, query the database to see if the email already exists.

```JavaScript
router.post('/', (req, res, next) => {
  const email = req.body.email;
  const password = req.body.password;

  knex('users').where('email', email)
    .then((data) => {
      if (data.length) {
        let err = new Error('Email already exists');
        err.status = 409;
        return next(err);
      }

      // Hash and salt password
    })
    .catch((err) => {
      return next(err);
    });
});
```

Add and commit your work.

```shell
git add .
git commit -m "add unique email check"
```

## How to use bcrypt

For our password hashing, we will use an NPM package called bcrypt. To start off with, install and save bcrypt:

```shell
npm install --save bcrypt
```

### Technique 1 (Generate salt and hash on separate function calls)

```JavaScript
const bcrypt = require('bcrypt');
const plainTextPassword = 'password123';
const saltRounds = 10;

bcrypt.genSalt(saltRounds, (saltErr, salt) => {
  if (saltErr) {
    // Handle saltErr
  }

  bcrypt.hash(plainTextPassword, salt, (hashErr, hash) => {
    if (hashErr) {
      // Handle hashErr
    }

    // Store hash in your password database
  });
});
```

### Technique 2 (Auto-gen a salt and hash)

- The first argument is the plain-text password to be hashed.
- The second argument is the cost of processing the data (defaults to 10).
- The final argument is a callback with two parameters:
  - `err` details any errors
  - `hash` is the encoded string that can be stored in the database.

```JavaScript
bcrypt.hash(plainTextPassword, saltRounds, (err, hash) => {
  if (err) {
    // Handle err
  }

  // Store hash in your password database
});
```

Use the `bcrypt.hash()` method to generate a salt and hash the password.

```JavaScript
router.post('/', (req, res, next) => {
  const email = req.body.email;
  const password = req.body.password;

  knex('users').where('email', email)
    .then((data) => {
      if (data.length) {
        let err = new Error('Email already exists');
        err.status = 409;
        return next(err);
      }

      bcrypt.hash(password, 10, (err, hash) => {
        if (err) {
          return next(err);
        }

        // Store hash in your password database
      });
    })
    .catch((err) => {
      return next(err);
    });
});
```

Add and commit your work.

```shell
git add .
git commit -m "use bcrypt to hash password"
```

Finally, use the correct knex query to insert the email and hashed password into the users table in the trackify_dev database

```JavaScript
router.post('/', (req, res, next) => {
  const email = req.body.email;
  const password = req.body.password;

  knex('users').where('email', email)
    .then((data) => {
      if (data.length) {
        let err = new Error('Email already exists');
        err.status = 409;
        return next(err);
      }

      bcrypt.hash(password, 10, (err, hash) => {
        if (err) {
          return next(err);
        }

        knex('users').returning('*')
          .insert({
            email: email,
            password: hash
          })
          .then((users) => {
            res.status(200).send(users[0]);
          })
          .catch((err) => {
            return next(err);
          });
      });
    })
    .catch((err) => {
      return next(err);
    });
});
```

Add and commit your work, and push to the registration branch. Then, checkout the master branch, and merge registration into master. Delete the registration branch if you'd like.

```shell
git add .
git commit -m "use knex to insert user data with hash into db"
git push origin registration
git checkout master
git merge registration
git branch -d registration
```