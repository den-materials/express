<!--
	creator: Ilias Tsangaris
	market: SF
	adapted by: Zeb Girouard
	market: DEN
-->

![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

<!-- There were SOOO many questions in this class, we didn't finish

<!--9:30 5 minutes -->
<!--Hook: Remember when we were using web APIs like Shakeitspeare and OMDB?  Raise your hand if you'd like to make one of those yourself.  Either way (good/too bad), that's what we're doing today. -->

# Building Web Servers with Express

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->

Express is a simple, flexible JavaScript library that enables us to more easily build a web server in Node.

### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

* **Use** npm to require Express
* **Illustrate** the request response cycle
* **Write** a web server using Express
* **Serve** JSON data over the server
* **Serve** static files over the server

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

* **Write** proficient JavaScript
* **Build** a simple web server with Node's `http` module

<!--9:35 15 minutes -->

## Request Response Cycle

![request](http://i.imgur.com/YXgj8.png)

## npm

*npm* started as the "node package manager", but according to npm it now apparently doesn't stand for anything. Either way, it's a tool that allows us to easily download community-built node modules.

You can initialize a new node project in an empty directory using npm with `npm init -y` and install packages like `express` with `npm install --save express`.

npm uses a file called `package.json` to track *dependencies* and *metadata* associated with the project.

A separate `node_modules` directory is where the modules your project requires will be housed. When using git, be mindful to add `node_modules` to a `.gitignore` file so it is *not tracked by git*.

At this point you can share the project code and future developers will run `npm install` to get the packages listed in `package.json` (instead of through git), which saves git from also having to track a large `node_modules` folder.

<!-- Catch-up, when turning over to students, allow them to view projector but *no copy-paste* -->

## Setup Express

> Challenge: initialize a new project and use npm to require `express` in it. Follow the best practices listed above so you could easily share the project with another developer.

To review:

1. `mkdir first-express-app`
2. `cd first-express-app`
3. `npm init -y`
4. `npm install express --save`
5. `touch .gitignore` and put node_modules inside

Now let's make our server:

6. `touch server.js` in first-express-app directory

**server.js**

```javascript
'use strict'

const express = require('express');
const app     = express();
const port    = process.env.PORT || 3000;

app.get('/', function(req, res) {
  res.send("You're Home!");
});


// start server
app.listen(port, function() {
  console.log('Server started on', port); 
});
```

Notice the `GET` verb in combination with the `/` path here. These are the two things needed to route an HTTP request.

>Can you give the verb/path combination that would be appropriate for reading the "about" page?

Now let's run our server

```bash
node server.js
```

Navigate to `http://localhost:3000` and voila!

<!-- 9:50 5 minutes -->

## Routes & Controllers

Your application's **routes** comprise all the possible HTTP verb & path combinations it will respond to. A **controller** is the callback that is executed after any given route is hit. Let's look at it again.

```js
app.get('/', function(req, res) {
  res.send("You're Home!");
});
```

Could be rewritten as:

```js
// controller
function homeController(req, res) { // a controller that handles a specific request
  res.send("You're Home!");
}
// route
app.get('/', homeController); // a GET to "/" routes to homeController
```

<!--9:55 10 minutes -->

## MiddleWare

#### Logging in Express with Middleware

**Middleware** is simply any function that gets called *after* the application's router receives a request but *before* the controller handles it.

> Order of events: HTTP Request --> Router --> Middleware --> Controller --> HTTP Response 

Add the following to your app.js file:

```javascript
// Middleware
app.use(function(req, res, next) {
  console.log("middleware hit");
  console.log("%s request to %s", req.method, req.path);
  next();
});

app.get('/', function(req, res) {
  console.log("home controller hit");
  res.send("You're Home!");
});
```

Let's go through this. After setting up our app and before our routes we tell our app to use a new function we are providing. That's all Middleware is! When writing custom Middleware, it's best practice to pass in the **req** object, the **res** object and finally **next**, _even if we don't use it!_ In this case, we are simply logging out the request method ('GET') and the request path ('/').

<!--Make sure you show this result in the terminal, and emphasize the order -->

<!--10:05 15 minutes -->

## Render JSON

Let's explore how we could serve up some data in an express app under the path `/api/taquerias`. Let's add some dummy data to display.

**server.js**
```js
// dummy data
const taquerias = [
  { name: "La Taqueria" },
  { name: "El Farolito" },
  { name: "Taqueria Cancun" }
]
```
Now we can render that data when the client sends a GET request to that specified endpoint. 

```js
// taquerias api index route
app.get('/api/taquerias', function (req, res) {
  res.json(taquerias); // render all taquerias
});
```

At this point our server should look something like:

```js
'use strict'

// dependencies
const express = require('express');
const app     = express();
const port    = process.env.PORT || 3000;

// dummy data
const taquerias = [
  { name: "La Taqueria" },
  { name: "El Farolito" },
  { name: "Taqueria Cancun" }
]

// middleware
app.use(function(req, res, next) {
  console.log("middleware hit");
  console.log("%s request to %s", req.method, req.path);
  next();
});

// home controller
app.get('/', function(req, res) {
  console.log("home controller hit");
  res.send("You're Home!");
});

// taquerias api index route
app.get('/api/taquerias', function (req, res) {
  res.json(taquerias); // render all taquerias
});

// start server
app.listen(port, function() {
  console.log('Server started on', port); 
});
```

> Challenge: render a list of cafes as JSON under the route `/api/cafes`. Include 3 cafes, each with a `name` and a `rating` (1-5). Test it by going to your server's address. BONUS: Try testing it with Postman. 

<!--10:20 10 minutes -->

## Render HTML

Maybe instead of sending the client back a JSON file we want to send them HTML instead. Let's make a directory called `views` and inside of it, touch a file `index.html`. Here's some code to get you started.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Oh ^_^ hai</title>
</head>
<body>
  <h1>Express is so lovely</h1>
  <img src="http://i.giphy.com/euYz83GfIvXpu.gif" alt="cat tracking computer mouse movement">
</body>
</html>
```

Now when the user visits the root route (home), instead of sending back just a string, let's send back an HTML file instead.

**server.js**

```js
// home controller
app.get('/', function(req, res) {
  console.log("home controller hit");
  res.sendFile(__dirname + "/views/index.html");
});
```

> Note: __dirname is the path of where `server.js` is located.

> Pro Tip: Restarting the server after every change is annoying. Consider using [nodemon](http://nodemon.io/) to help solve this problem.

<!-- 10:30 10 minutes -->
#### Serve a Public Directory

HTML files often require many assets, such as scripts, stylesheets, images, videos, icons, etc. As a result we should probably have a place we can store all those public assets.

1. Make a directory in your project called `public`; then create a `public/scripts` subdirectory. Make an `app.js` in this subdirectory.

1. Set up the express app to serve the static files in the public directory.

  ```js
    // server.js
    app.use(express.static('public'));
  ```

1. Get a `console.log("Sanity Check: JS is working!")` from your `app.js` to appear in your browser dev tools console.

>Tip: Reference the path `/scripts/app.js` in your HTML page (the public folder now lives at the root of your application's directory).

> Challenge: Add a `public/styles` directory with a `main.css` file inside. Use that stylesheet to change your home page's background color to pink.

<!--10:40 5 minutes -->

## Example Express File Tree

Here's a recommended way to structure a simple web application:

```
├── server.js  // your server code
├── package.json    // lists dependencies; changed by npm install --save somePackage
├── public  // i.e. client-side
│   ├── images  // images to serve to client
│   ├── scripts
│       └── app.js   // client-side javascript file
│   └── styles
│       └── style.css
├── views  // html files that we'll serve
│   └── index.html
└── node_modules  // don't edit files in here!
    ├── express // etc
```

<!--10:45 5 minutes -->

## Closing Thoughts

* What is a server?
* How do we require Express in our project?
* Why is it bad practice to track `node_modules` through git?
* What is the difference between a path, an http verb, and a route?
* What is a controller?
* Why do we sometimes want to render JSON and sometimes want to render HTML?
* Why do we usually serve up a public directory with Express?

## Additional Resources

* [What is npm?](https://www.youtube.com/watch?v=x03fjb2VlGY)
* [Representational State Transfer (REST)](https://en.wikipedia.org/wiki/Representational_state_transfer) and [example RESTful actions](http://guides.rubyonrails.org/routing.html#crud-verbs-and-actions) (Rails specific example, but it doesn't matter)

## Licensing
All content is licensed under a CC­BY­NC­SA 4.0 license.
All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.
