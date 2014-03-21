title: How does request.pipe(response) work?
date: 2014-03-21 00:33:44
tags: express, node
---

    readableStream.pipe(writeableStream);
    request.pipe(response);

The code above is so beatiful, but how does it work? Once the request
stream is done piping data, the response stream will shoot off its
headers and payload. But, how does it know how to do that?

We know that request/response inherit from the Stream interface, which,
in turn, inherits from EventEmitter. Readable streams have two very
important events called 'data' and 'end'. Writable streams has an event
called 'finish'. When the readable stream finishes piping to writable
stream, the writable stream emits the event 'finish'.


Let's do an experiment:

    var writeStream = new fs.WriteStream("write.txt");
    writeStream.on('finish', function () {
      console.log("I am called when the readable stream has finished piping");
    });

    var readStream = new fs.ReadStream("prepared-text.txt");
  
    readStream.pipe(writeStream);
    
Result:

    "I am called when the readable stream has finished piping"
        

Looks like it works!


You can read more about streams at nodejs.org/api/stream.html and
nodeschool.io.


