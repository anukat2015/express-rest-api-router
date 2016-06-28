# express-rest-api-router
[![npm version](https://badge.fury.io/js/express-rest-api-router.svg)](http://badge.fury.io/js/express-rest-api-router)
[![Build Status](https://travis-ci.org/mallocator/express-rest-api-router.svg?branch=master)](https://travis-ci.org/mallocator/express-rest-api-router)
[![Coverage Status](https://coveralls.io/repos/mallocator/express-rest-api-router/badge.svg?branch=master&service=github)](https://coveralls.io/github/mallocator/express-rest-api-router?branch=master)
[![Dependency Status](https://david-dm.org/mallocator/express-rest-api-router.svg)](https://david-dm.org/mallocator/express-rest-api-router) 

A router that automatically verifies incoming parameters while creating a json document of all available endpoints.

Features:
 * Checks if all required parameters are set on an endpoint
 * Automatically sets default values for missing parameters
 * Generates a json map of all endpoints for documentation 
 * Supports nested documents for json POST bodies,
 * Customizable error handler per router and for each endpoint

It's probably easier to just show an example of what the router does.

Example configuration:

```
var Router = require('express-rest-api-router);

var router = Router();

router.get('/endpoint', {
    description: 'An example endpoint',
    params: {
        name: 'string'
    }
}, (req, res) => {
    res.end('hello ' + req.args.name); 
});

app.use(Router);
```

With the default configuration this is the output of the different requests to the server:

http://myhost/endpoint => 
```
Status Code 422
```

http://myhost/endpoint (developer mode) => 
```
Status Code 422 
{ 
    "error": "Required parameters are missing",
    "params": {
        "name": {
            "type": "string",
            "error": "not set"
        }
    }
}
```

http://myhost/endpoint?name=bob => ```hello bob```

http://myhost/ => 
```
Status Code 200
{
    "/endpoint": {
        "GET": {
            "description": "An example endpoint",
            "params": {
                "name": {
                    "type": "string",
                    "required": true,
                    "default": undefined
                }
            }            
        }
    }
}
```


## API

### Router([configuration])

The router comes with sensible default values preconfigured, but various behavior can be overridden through the configuration
 
```
var Router = require('express-rest-api-router);
var router = Router({
    error: (value, req, res, next) => {},               // A global error handler that overrides the default behavior
    success: (value, req, res, next) => {},             // A success error handler that overrides the default behavior
    validate: (value, req, res, next) => {},            // A global validator the overrides the default behavior
    paramMap: 'arguments',                              // The property on the request object on which to find parsed parameters
    paramOrder: ['body', 'query', 'params', 'coookies'] // The order in which request properties are searched for incoming parameters
                                                        // Once a parameter has been found it's not going to be overwritten by other properties
});
```


### Router.api(req, res);

A standard request handler implementation that will respond with the currently configured api for this router. Can be used to make
it easier for developers to work with your API.

```
app.get('/', router.api);
// => responds via res.jsonp() and prints current endpoint configuration
```

The api handler supports multiple formats that can be specified either through a get or path paramter named format:
 
```
app.get('/api/:format', router.api);
// => can respond to e.g. /api/tree or /api?format=tree (which doesn't require the path variable).
```

The supported formats are ```json``` (default), ```tree```, ```table```, ```csv```, ```xml```


### Router.endpoints;

A map with the configuration of all the endpoints on this Router. Use this to build your own custom api handlers or to do more advanced
configuration.

```
console.log(router.endpoints)
// => { "path": { "method": { "description": "endpoint description", "params": {}}}}
```


### Router.<verb>(endpoint, configuration, ...handler);

The router supports all routing verbs that the standard express router supports, including ```get```,```post```,```put```,```delete```, and ```all```.
The method calls still behave the exact same way as before only that a configuration object is passed in as well.

Example: 
```
var configuration = {} // See API reference for Configuration
var listHandler = (req, res, next) => {};

Router.post('/list', configuration, listHandler);
```

### Configuration

The configuration is where most of the work needs to be done to get the router working the way you want:

#### Router.description

A simple string field that allows to set a description of the endpoint.

```
{
    description: 'A description of the endpoint that is printed in the json map'
}
```

#### Router.params

An object that describes the expected parameters for this endpoint. Each parameter is defined as a property of the params object. The parameter definition
supports either a simple string definition in the format of ```type(default)```:
```
{
    params: {
        name: 'string(bob)'
    }
}
```

or a more complex format as an object with more options:
```
{
    params: {
        name: {
            type: 'string',
            default: 'bob',
            required: false,
            description: 'The users name'       
            min: 3,                                         // min characters for string, min value for number, ignored for boolean
            max: 10,                                        // max characters for string, min value for number, ignored for boolean
            validate: (value, cb) => { cb(err, value); }    // Function to override validation behavior
            error: (error, req, res, next) => {}            // Function to override error behavior
            success: (null, req, res, next) => {}           // Function to trigger on success, does not override request handler
        }
    }
}
```

Valid types that can be used are: ```boolean```, ```number```, ```string```. Complex Objects (which are only
supported in body requests) can be defined by simply nesting definitions:
```
{
    params: {
        user: {
            name: 'string(bob)',
            age: 'number(30)'
        }
    }
}
```

Support for arrays is tbd. most likely it will be supported transparently.


For more examples check out the test directory
