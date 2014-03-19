title: How does express.static() work?
date: 2014-03-11 11:13:12
tags: express, node
---
app.use(express.static(‘public’));

express.static(‘public’) makes the contents of the public  directory available at the root domain. I suspect it utilizes metacode to retrieve said public assets.  The algorithm is probably structured like this:

1.	Parse out the file name from a request
2.	Check for the existence of the requested file
3.	If it exists, serve it through a response
4.	If it doesn’t exist, respond with a 404

is like:

    app.get(‘/application.css’, function (req,res) {
      fs.readFile(‘./views/index.html’, function(err, data) {
          res.send(data.toString());
       })
    });


