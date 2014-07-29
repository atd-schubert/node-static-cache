# Static Cache
## Description

This module uses the serve-static module to serve cached data.

## Installation
Just run in your console:
```
npm install --save static-cache
```
## How to use
You can simply require the module in node. The required module is a class. You have call the class to make a new object with the folder path for the cache. Additional you can add the maxAge option to set the maxAge of a cached resource.

```
"use strict";

var Cache = require("static-cache")

var cache = new Cache({path: __dirname+"/_cache", maxAge: 1000*60*60*24}); // maxAge := one day

```

Please look at the API description for further informations about the cache object.

### Example usage
```
"use strict";

var Cache = require("static-cache")

var cache = new Cache({path: __dirname+"/_cache"});
cache.cache("/path/to/file.txt",                    //the absolute path in cache not in the filesystem!
  "This is the content of the cached file",
  function(err){
    if(err) return console.log("The following error occured:", err);
    cache.get("/path/to/file.txt", function(err, data){
      if(err) return console.log("The following error occured:", err);
      console.log("The cached data is: ", data.toString());
      cache.clean("/path/to/file.txt", function(err){
        if(err) return console.log("The following error occured:", err);
        console.log("'/path/to/file.txt' is deleted");
      });
    });
  });
  
cache.cache("/just/10/seconds.txt", "this file only exists for 10 seconds because of the ttl option", function(err){
  if(err) return console.log("The following error occured:", err);
  cache.ttl("/just/10/seconds.txt", 1000*10);
  cache.get("/just/10/seconds.txt", function(err, data){
    if(err) return console.log("The following error occured:", err);
    console.log("The cached data is currently available: ", data.toString());
  });
  
  setTimeout(function(){
    cache.get("/just/10/seconds.txt", function(err, data){
      if(err) return console.log("The resource is already deleted and now we got this error: ", err);
      console.log("The cached data is currently available, but shouldn't: ", data.toString());
    });
  }, 1000*11);
  
});
```

## API
### cache(path, content, callback)
Use this for adding contents to the cache. The callback will return nothing or an error, if there is one.
### clean(path, callback)
Use this for removing content. Path is optional and matches also on a folder. In this case the whole folder would be deleted. If there is no path specified the whole cache will be deleted.
### ttl(path, duration)
This function calls clean(path) after the specified duration. You are able to cache a resource only for a specified duration.
### get(path, callback)
Get the content of a saved resource as a stream. The callback will receive cb(err, stream).
### serve(req, res, next)
Use this method as a middleware for connect or express.
```
app = require("express")();
app.use(cache.serve)
```