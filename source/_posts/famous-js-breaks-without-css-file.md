title: Famous JS breaks without CSS File
date: 2014-04-23 00:51:55
tags:
---
I spent the last three hours figuring out why there was a bug in my
famous code. It turns out famous.css, found in core/, is a hard
requirement for the famous code to work. I did not know this and I
learned the hard way.

When the CSS was not linked in the header, I could never set a 100%
height on a famous-surface by using 'undefined' in size: [200,
undefined]. 

    require('./famous-polyfills/index');

    require('domready')(function () {
      var Engine = require('./famous/core/Engine');
      var ImageSurface = require('./famous/surfaces/ImageSurface');
      var Surface = require('./famous/core/Surface');
      var StateModifier = require('./famous/modifiers/StateModifier');
      var ContainerSurface = require('./famous/surfaces/ContainerSurface');

      // create the main context
      var mainContext = Engine.createContext();

      var surface = new Surface({
         size: [undefined, undefined],
         properties: {
           backgroundColor: 'blue',
           color: 'red',
         }
      });

      mainContext.add(surface);
    });
