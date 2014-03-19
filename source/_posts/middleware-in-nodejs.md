title: Middleware in NodeJS
date: 2014-03-19 00:23:29
tags: middleware
---

From a web development perspective, middleware is a pipe (or series of pipes) that can read, update, or respond to in-transit HTTP requests before they reach their intended destination/route (ie. HTTP GET Request to www.wundercode.net/about). If you understand how it works, you can do infinitely fun and useful things with it. 

In ExpressJS, we can register custom or prebuilt middleware functions to the application  as a callback with the use() method. Note that middleware functions take a request, response, and next object as parameters. I will explain what 'next' does later.

Let's start with an example. When someone visits www.wundercode.net their request will be handled by the route below. 

    app.get('/', function (req, res) {
      fs.readFile('./views/index.html', function (err, data) {
        res.send(data.toString());
      });
    }); 

But, before the request is sent to that route, it will first go through our middleware function.


    var randomString = "Chris-";

    app.use(function (req, res, next) {
      res.random = randomString;
      console.log("1st random string: " + res.random);
      console.log("I think middleware");
      next();
    });

In the above example, I wrote middleware that assigns 'randomString' to res.random. I then pass the request/response objects onto the next middleware/route in queue by calling next. If I do not call next, the request/response objects will not be passed on and now more middleware will be triggered.

The use method takes two parameters; an optional path and middleware function. By default, if you do not set the path option, all requests will be processed by middleware. If you decided to assign path a value, only requests with a host header value of path will be processed for the related middleware function.  

If you want only one middleware function to trigger for a specific route, make sure to call use(path, yourMiddleware) first before any others. Express will generate a chain of middleware that your request will go through based on the order of use calls.  

Let's go through another example to hammer it in: 

    app.use(function (req, res, next) {
      res.random = randomString;
      console.log("1st random string: " + res.random);
      console.log("I think middle ware");
      next();
    });

    //more middleware here similar to the ones 
    //directly located below and above this comment

    app.use(function (req, res, next) {
      res.random += "456"
      console.log("4th random string: " + res.random);
      console.log("through pipes");
      next();
    });

    app.use('/application.css', function (req, res) {
      console.log("Processing application css");
    });

    app.use('/application.js', function (req, res) {
      console.log("Processing application js");
    });

The output when a request to www.wundercode.net occurs: 

    1st random string: Chris-
    I think middleware
    2nd random string: Chris-123
    is like a stream
    3rd random string: Chris-123456
    that flows and flows and flows
    4th random string: Chris-123456456
    through pipes
    Served index.html at Tue Mar 11 2014 16:23:49 GMT-0700 (PDT)

    1st random string: Chris-
    I think middleware
    2nd random string: Chris-123
    is like a stream
    3rd random string: Chris-123456
    that flows and flows and flows
    4th random string: Chris-123456456
    through pipes
    Processing application css

    1st random string: Chris-
    I think middleware
    2nd random string: Chris-123
    is like a stream
    3rd random string: Chris-123456
    that flows and flows and flows
    4th random string: Chris-123456456
    through pipes
    Processing application js

I will walk you through what just happened:

1.  When the http GET request for www.wundercode.net hits the server, it runs it through the middleware we set.
2.  The middleware for /application.js and /application.css isn't triggered because the request's header value for host doesn't contain the path /application.js and /application.css.
3.  Once the request reaches the route, the server responds with index.html.
4.  Index.html contains link tags to /application.js and /application.css and the browser will request for those resources as the dom is being rendered.

As you can see, asset (css, js, images, etc) requests will be processed by most of the middleware stack with this configuration, which is what we don't want. We want asset requests to be only processed by asset middleware. 

The next example shows you why order matters and how the next() function comes into play. As you can see, the middleware for css and js assets are sandwiched between the middleware that console logs random strings.  Our expectation is any request on /application.css or /application.js will now only trigger the first two randomString middleware because of the way we structured the use() method calls. 

    app.use(function (req, res, next) {
      res.random = randomString;
      console.log("1st random string: " + res.random);
      console.log("I think middle ware");
      next();
    });

    app.use(function (req, res, next) {
      res.random += "123"
      console.log("2nd random string: " + res.random);
      console.log("is like a stream");
      next();
    });

    app.use('/application.css', function (req, res) {
      console.log("Processing application css");
    });

    app.use('/application.js', function (req, res) {
      console.log("Processing application js");
    });

    app.use(function (req, res, next) {
      res.random += "456"
      console.log("3rd random string: " + res.random);
      console.log("that flows and flows and flows");
      next();
    });

    app.use(function (req, res, next) {
      res.random += "456"
      console.log("4th random string: " + res.random);
      console.log("through pipes");
      next();
    });

Our hunch was correct! The result should be:

    1st random string: Chris-
    I think middle ware
    2nd random string: Chris-123
    is like a stream
    3rd random string: Chris-123456
    that flows and flows and flows
    4th random string: Chris-123456456
    through pipes
    Served index.html at Tue Mar 11 2014 16:26:10 GMT-0700 (PDT)

    1st random string: Chris-
    I think middle ware
    2nd random string: Chris-123
    is like a stream
    Processing application css

    1st random string: Chris-
    I think middle ware
    2nd random string: Chris-123
    is like a stream
    Processing application js

Finally, our expectation now is an asset requests will only trigger related asset middleware with this configuration. 

    app.use('/application.css', function (req, res) {
      console.log("Processing application css");
    });

    app.use('/application.js', function (req, res) {
      console.log("Processing application js");
    });

    app.use(function (req, res, next) {
      res.random = randomString;
      console.log("1st random string: " + res.random);
      console.log("I think middle ware");
      next();
    });

    app.use(function (req, res, next) {
      res.random += "123"
      console.log("2nd random string: " + res.random);

      console.log("is like a stream");
      next();
    });

    //more middleware here similar to the ones 
    //directly located below and above this comment

    app.use(function (req, res, next) {
      res.random += "456"
      console.log("4th random string: " + res.random);

      console.log("through pipes");
      next();
    });

It works! The result:

    1st random string: Chris-
    I think middle ware
    2nd random string: Chris-123
    is like a stream
    3rd random string: Chris-123456
    that flows and flows and flows
    4th random string: Chris-123456456
    through pipes
    Served index.html at Tue Mar 11 2014 16:28:09 GMT-0700 (PDT)

    Processing application css

    Processing application js


So, what is the point? The ordering of your middleware is very important
in express that middleware is fun and conceptually simple!

Please email me at chris@wundercode.net if you have any questions or comments.

P.S. Here's a nice example of where middleware can also hijack requests and send its own responses:

    app.use(function (req, res, next) {
      console.log("I'm an angry middleware and I'm going to block all access to the site!");
      res.send("You're blocked because middleware is angry!");
    });

    app.get('/', function (req, res) {
      fs.readFile('./views/index.html', function (err, data) {
        res.send(data.toString());
      });
    });

