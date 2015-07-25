
---
title: Writing Database Schema with Bookshelf and Knex
tags: Javascript, Bookshelf.js, Knex.js
---

Using an object-relational mapper (ORM) to interact with a database in your node application can be a huge saver.  Two of the better known ORMs for interacting with SQL databases are Bookshelf and Sequelize.  In this post, I will guide you through setting up your database connection, schema and queries using Bookshelf.

Before we get started, it’s important to understand the relationship between Bookshelf and Knex.  Bookshelf is an ORM for Node.js that is build on the Knex SQL query builder.

### 1. Connect to Database

First we must establish our connection to the database.  Client is the SQL server you are working with (e.g. MySQL, Postgres, SQLite, etc.).  In a development environment, your host will most likely be 127.0.0.1.  You’re user and password are specific to the login credentials you established when you set up your SQL server.  After you establish your connection, you should pass this connection into require(‘bookshelf’) so that your Bookshelf instances have access to the database.  I usually contain this code within a db.js file.

```
var knex = require('knex')({
  client: 'mysql',
  connection: process.env.CLEARDB_DATABASE_URL || {
    host: process.env.RDS_HOSTNAME,
    user: process.env.RDS_USERNAME,
    password: process.env.RDS_PASSWORD,
    port: process.env.RDS_PORT,
    database: process.env.DB_NAME,
    charset: 'utf8'
  }
});

```

### 2.  Write Schema and Define Relationships

I usually start by writing a function to check if a table exists.  If it doesn’t exist, I create a new table, in this case, called ‘posts.  In the createTable callback function, you can establish new schema by tagging them on to the object passed in (I usually call this object the singular name of the table object, but you can call it whatever you want).  

There are few schema worth highlighting.  The first schema called id is the primary.  I tell knex this is the primary key by invoking the built-in primary() function.  I set the data type to increments so that every new row that gets added to the table will be different.  The ‘user_id’ column is this tables’ foreign key.  There are two steps to establishing a foreign key relationship.  The first step is adding .unsigned().references(foreign_key_table.foreign_key_table_column).   The second step occurs in the next section when we define our models.

```
db.knex.schema.hasTable('posts').then(function(exists) {
  /* Drops the table if it exists.  This is useful to uncomment when you are working on editing the schema */
  // if (exists) {
  //   db.knex.schema.dropTable('posts').then(function() {
  //     console.log("Removed Post Table");
  //   });
  //   exists = false;
  // }

  /* Create users table if it doesn't exist. */
  if (!exists) {
    db.knex.schema.createTable('posts', function(post) {
      post.increments('id').primary();
      post.string('title', 255);
      post.string('summary', 255);
      post.string('file', 255);
      post.string('url', 255);
      post.integer('views').defaultTo(0);
      post.integer('user_id').unsigned().references('users.id').onDelete('CASCADE');
      post.timestamp('created_at').notNullable().defaultTo(db.knex.raw('now()'));
    }).then(function(table) {
      console.log('Created Posts Table');
    });
  }
});
```

### 3. Model Definition

When you define a model , you must provide the tableName that corresponds to the SQL table this model is associated with.  You must provide relationship context.  In this example, there is a one to many relationships between users and posts.  Recall that in the previous section, we added the users.id foreign key column to the posts table.  When we define our Post model, we add a user function (name of the function doesn’t matter) that returns this.belongsTo(User, ‘user_id).  In your User model, we add a posts function that returns this.hasMany(Post).  As a general best practice, if you have trouble defining your schema, always try to be as explicit as possible.

```
var User = exports.User = db.Model.extend({
  tableName: 'users',
  posts: function() {
    return this.hasMany(Post);
  },
  comments: function() {
    return this.hasMany(Comment);
  },
  votes: function() {
    return this.hasMany(Vote);
  }
});

var Post = exports.Post = db.Model.extend({
  tableName: 'posts',
  user: function() {
    return this.belongsTo(User, 'user_id');
  },
  tags: function() {
    return this.belongsToMany(Tag, 'tag_id');
  },
  comments: function() {
    return this.hasMany(Comment);
  },
  votes: function() {
    return this.hasMany(Vote);
  }
});
```