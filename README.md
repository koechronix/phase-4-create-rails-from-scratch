### STEP 1 :HOW TO CREATE RAILS API FROM SCRATCH
# README

# Creating a Rails API from Scratch

## Learning Goals

- Use the `--api` flag to create an API-only Rails app
- Use the `resource` generator

## Introduction

We've spent a lot of time now focusing on the backend, and now's a great
opportunity to see what we can actually do with all the power of a Rails API to
support a frontend application as well.

Throughout this section, we'll be building a DVD shop. We'll have a Rails API to
support a React frontend application, and we'll be focusing on how that
client-server communication process works, as well as some challenges involved
in communicating between two separate applications.

In this lesson, we'll start by building the Rails backend from scratch and talk
through some of the typical configuration when creating a new Rails API.

## Generating a Rails API

Just like we saw at the beginning of the phase, we can use `rails new` to
generate a new Rails application. We'll run that same command with a few
additional options to optimize our Rails app. Let's generate the backend code
for our `dvd-shop`. Use `cd ..` to navigate out of the lab directory, and run:

```console
$ rails new dvd-shop --api --minimal
```

- `--api`: this flag will create our new application with some additional
  API-specific configuration, and will skip the code for generating `.erb` files
  with ActionView.
- `--minimal`: this flag skips a lot of additional Rails features that we
  won't use in our API, such as code for sending emails and processing images.
  Read more about the [`--minimal` flag][--minimal].

> The reason we ask you to `cd` out of the lab directory is because when you
> generate a new Rails project, it will automatically create a new Git
> repository for your Rails project. Since the lab directory is already a Git
> repository, it's better to create this new project in its own directory, so
> you don't end up with nested Git repositories.

With that code in place, let's generate the code for handling our first request
from the client.

## Using the Resource Generator

One of the main features of our frontend application will be to display a list
of movies. For that feature, we'll want our API to handle a `GET` request to
`/movies`.

To get that request working, we'll need to create a **route** and **controller**
action on our Rails server. We'll also need a **model** to interact with the
database, and a **migration** to generate the corresponding database table for
this model.

For our `Movie` model, we'll want a table with the following attributes:

| Column Name     | Data Type |
| --------------- | --------- |
| title           | string    |
| year            | integer   |
| length          | integer   |
| director        | string    |
| description     | string    |
| poster_url      | string    |
| category        | string    |
| discount        | boolean   |
| female_director | boolean   |

We could create the route, model, controller, and migration individually, but
since this kind of operation is pretty common for a Rails developer, there's a
handy generator that will set up all the code we need: `rails g resource`.

Navigate into the `dvd-shop` directory and run this code in your terminal:

```console
$ rails g resource Movie title year:integer length:integer director description poster_url category discount:boolean female_director:boolean --no-test-framework
```

This command will:

- Generate a migration for creating a `movies` table with the specified attributes
- Generate a `Movie` model file
- Generate a `MoviesController` controller file
- Add `resources :movies` to the `routes.rb` file

It's a powerful command, so make sure to use it sparingly! You should only use
`rails g resource` if you truly need all of that code generated.

## Running the API

To get some sample data into our application, we've provided a `seeds.rb` file
in the root directory of this repo. Copy the contents of this file into your
`db/seeds.rb` file. Then, to set up and seed the database, run:

```console
$ rails db:migrate db:seed
```

Let's update our `routes.rb` file to set up just the one route our frontend
needs, for the time being:

```rb
# config/routes.rb
resources :movies, only: [:index]
```

We can also add the index action to our controller:

```rb
def index
  movies = Movie.all
  render json: movies
end
```

With that code in place, run `rails s` to start the server, and visit
`http://localhost:3000/movies` in the browser to see our movie data. Success!

## Conclusion

When creating a new Rails API project from scratch, you can use the `--api` flag
to have Rails optimize your project for building a web API.

We also saw how to use the `resource` generator, which can help quickly set
up the code we need to create RESTful routes and CRUD functionality for one
single resource.

## Check For Understanding

Before you move on, make sure you can answer the following question:

1. What files are generated when running `rails g resource ResourceName`?

## Resources

- [The Rails Command Line](https://guides.rubyonrails.org/command_line.html)

[--minimal]: https://bigbinary.com/blog/rails-6-1-adds-minimal-option-support


* ...



















### STEP 3:HOW TO CREATE REACT AND USE FOREMAN
# Adding React to Rails

## Learning Goals

- Use `create-react-app` to generate a new React application within a Rails
  project
- Proxy requests from React to Rails in development
- Use the `foreman` gem to run React and Rails together

## Introduction

In the last lesson, we created a Rails API from scratch. Now it's time to see
how we can add a React frontend application as well.

There are a number of ways to use React and Rails together, such as using the
[`webpacker` gem](https://github.com/rails/webpacker) to manage JavaScript as
part of your Rails application. However, we like the simplicity and the tooling
that you get out of using [`create-react-app`](https://create-react-app.dev/) to
generate a new React application within Rails. If you've used `create-react-app`
before, you should feel right at home! We can also add a few additional tools to
the process to make running React and Rails together a bit easier.

## Generating a React Application

To get started, let's spin up our React application using `create-react-app`:

```console
$ npx create-react-app client --use-npm
```

This will create a new React application in a `client` folder, and will use npm
instead of yarn to manage our dependencies.

When we're running React and Rails in development, we'll need two separate
servers running on different ports — we'll run React on port 4000, and
Rails on port 3000. Whenever we want to make a request to our Rails API from
React, we'll need to make sure that our requests are going to port 3000.

We can simplify this process of making requests to the correct port by using
`create-react-app` in development to [proxy the requests to our API][proxy].
This will let us write our network requests like this:

```js
fetch("/movies");
// instead of fetch("http://localhost:3000/movies")
```

To set up this proxy feature, open the `package.json` file in the `client`
directory and add this line at the top level of the JSON object:

```json
"proxy": "http://localhost:3000"
```

Let's also update the "start" script in the the `package.json` file to specify a
different port to run our React app in development:

```json
"scripts": {
  "start": "PORT=4000 react-scripts start"
}
```

With that set up, let's try running our servers! In your terminal, run Rails:

```console
$ rails s
```

Then, open a new terminal, and run React:

```console
$ npm start --prefix client
```

This will run `npm start` in the client directory. Verify that your app is
working by visiting:

- [http://localhost:4000](http://localhost:4000) to view the React application
- [http://localhost:3000/movies](http://localhost:3000/movies) to view the Rails
  application

We can also see how to make a request using `fetch`. In the React application,
update your `App.js` file with the following code:

```jsx
import { useEffect } from "react";

function App() {
  useEffect(() => {
    fetch("/movies")
      .then((r) => r.json())
      .then((movies) => console.log(movies));
  }, []);

  return <h1>Hello from React!</h1>;
}

export default App;
```

This will use the `useEffect` hook to fetch data from our Rails API, which you
can then view in the console.

## Running React and Rails Together

Since we'll often want to run our React and Rails applications together, it can
be helpful to be able to run them from just one command in the terminal instead
of opening multiple terminals.

To facilitate this, we'll use the excellent [foreman][] gem. Install it:

```console
$ gem install foreman
```

Foreman works with a special kind of file known as a Procfile, which lists
different processes to run for our application. Some hosting services, such as
Heroku, use a Procfile to run applications, so by using a Procfile in
development as well, we'll simplify the deploying process later.

In the root directory, create a file `Procfile.dev` and add this code:

```txt
web: PORT=4000 npm start --prefix client
api: PORT=3000 rails s
```

Then, run it with Foreman:

```console
$ foreman start -f Procfile.dev
```

This will start both React and Rails on separate ports, just like before; but
now we can run both with one command!

**There is one big caveat to this approach**: by running our client and server
in the same terminal, it can be more challenging to read through the server logs
and debug our code. Furthermore, `byebug` will not work. If you're doing a lot of
debugging in the terminal, you should run the client and server separately to
get a cleaner terminal output and allow terminal-based debugging with `byebug`.

You can run each application separately by opening two terminal windows and
running each of these commands in a separate window:

```console
$ npm start --prefix client
$ rails s
```

This will run React on port 4000 (thanks to the configuration in the
`client/package.json` file), and Rails on port 3000 (the default port).

## Conclusion

In the past couple lessons, we've seen how to put together the two pieces we'll
need for full-stack applications by using `rails new` to create a new Rails API,
and `create-react-app` to create a React project. Throughout the rest of this
module, we'll focus on how our two applications communicate and share data.

## Check For Understanding

Before you move on, make sure you can answer the following questions:

1. What options do you have for running Rails and React at the same time?
2. What are the advantages and disadvantages of using `foreman` as described in
   this lesson?

## Resources

- [Proxying API Requests in Create React App][proxy]

[proxy]: https://create-react-app.dev/docs/proxying-api-requests-in-development/
[foreman]: https://github.com/ddollar/foreman
