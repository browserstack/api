# API Overview
The following denotes the HTTP-based API for BrowserStack.

### Schema
All requests are made to `http://api.browserstack.com/VERSION/` and all returned data is done so in JSON-format. The version this documentation outlines is 1.

    $ curl -i http://api.browserstack.com/1

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    X-API-Version: 1
    Content-Length: 2

    {}

All date formats are given in ISO-8601 which can be processed natively with `Date.parse` in compatible JavaScript environments.

### Error Handling
All requests are pre-processed and validated. This section outlines how we handle errors within the API and respond to them.

1. Sending Invalid JSON-content will result in the following response.
    
    ```
    HTTP/1.1 400 Bad Request
    Content-Length: 26
    
    {"message":"Invalid JSON"}
    ```
    
2. All API requests are validated. The following is an example output for a required parameter that wasn't given.
  
    ```
    HTTP/1.1 422 Unprocessable Entity
    Content-Length: 136
    
    {
      "message": "Validation Failed",
      "errors": [
        {
          "field": "type",
          "code": "required"
        }
      ]
    }
    ```
    
    Possible error codes are `required` and `invalid`.

### Authentication
Where necessary you will need to authenticate who you are. Before spawning browser workers and getting browser screenshots for example. Authentication is done using your username/password within the HTTP request. For example:

    $ curl -u "username:PASSWORD" https://api.browserstack.com/1
    
### JSON-P Callbacks
Add a `?callback` parameter to any GET request and the JSON response will be wrapped in the given callback function name. This is particularly useful for cross-domain browser security constraints.

### HTTP Verbs
The API is kept concise and simple by making use of relevant HTTP verbs on each requests. The specifications for these are vague and their use within this API is specific but in general we follow the following rules:

  * `HEAD` - Performs the request to asses the status of a resource and expects no content response.
  * `GET` - Used to retrieve resources.
  * `POST` - Used to create new resources.
  * `PUT` - Used to update resources.
  * `DELETE` - Used to delete resources.

# API Documentation

## Getting Available Browsers
Fetches all available browsers and IDs that represent them. IDs will be unique to each browser and will never change for the continued existence of that API version.

    GET /browsers(/:type)

### Parameters

#### Type
The browser type. Currently theses are:

  * ie (Internet Explorer)
  * ff (Mozilla FireFox)
  * chrome (Google Chrome)
  * ios (Apple iOS Safari)
  * opera (Opera)
  * android (Google Android Chrome)
  * safari (Apple Safari)
  * ...
  
### Output

```javascript
[
  {
    name: 'Internet Explorer',
    type: 'ie',
    version: 7,
    tag: 'ie7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
  },
  {
    name: 'FireFox',
    type: 'ff',
    version: 2.0,
    tag: 'ff2',
    agent: 'Mozilla/5.0 (Windows; U; Windows NT 5.1; en-GB; rv:1.8.1.6) Gecko/20070725 Firefox/2.0.0.6	'
  },
  {
    name: 'Internet Explorer',
    type: 'ie',
    version: 6.0,
    tag: 'ie6',
    agent: 'Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322)'
  } ...
]
```
  
## Create a New Browser Worker
A browser worker is simply a new browser instance with `window.ondata` and `window.postData` added to the global window object. Everything else is left untouched.

    POST /worker

> This call requires authentication. A `401 Unauthorized` response is given if an unauthorised request is made.

Once a worker has been spawned you can see it listed in your dashboard account online at browserstack.com. You can then control this browser instance remotely if necessary.

`ondata` is an event that's called when the client initiates a `PUT /worker/:id` request. See the _Getting results_ section for more details.

`postData` is a method that's used to send messages back to the client. Every time it's called a new object is added to a stack which is emptied into the response of a `GET /worker/:id` request. See the _Sending message/data_ section for more details.

### Parameters
A valid request must contain a `browser` and either a `url` or `data` parameter but not both. `timeout` is optional but defaults to 30 seconds.

#### browser
A valid browser tag. A list of supported browser tags are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.
 
#### (timeout=30)
A number in seconds before the worker is terminated. Set this to 0 to keep the worker alive indefinitely.

> IMPORTANT! If you have set the timeout to 0. Make sure you remember to terminate the worker otherwise it will continue to use up your credits.

#### (url)
A valid url to navigate the browser to. This should be used instead of `data` and not together.

> Make sure the url is encoded. JavaScript: encodeURI(url), PHP: urlencode($url), 

#### (data)
A valid HTML content page to run. This should be used instead of `url` and not together.

> Must be a url-safe base-64 encoded string.

### Response
The response will be returned when the worker has been setup and initialised. This involves loading the HTML data or navigating to the url given depending on the setup parameters. Use the id returned to perform any further communications etc.

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1
    
    {
      "id": "da39a3ee"
    }
    
> Note this can take up to 1 minute to complete.

If there was an error decoding the data or the URL failed to load then a `HTTP/1.1 422 Unprocessable Entity` response is given with the error description as the response body. An example for a Page Not Found error on the given URL would be:

```  
POST /worker
  url="http://some-non-existant-domain.tld"
  browser="ie7"
```

```
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
X-API-Version: 1

{
  "message": "URL Could Not Be Found",
  "errors": [
    {
      "field": "url"
      "message": "Page Not Found"
      "code": 404
    }
  ]
}
```

## Getting a Worker Screenshot
Once a worker has been started you can get a screenshot of the browser window at any point whilst the worker is still running. If the worker has been terminated a `404 Not Found` response will be given instead.

    GET /worker/screen/:id
    
`id` represents the worker id. This is returned from a successful `POST /worker` call and is required to get a screenshot of that worker instance.

> This call requires authentication. If the request was made unauthorised a `401 Unauthorized` response is given. Alternatively if the authorised user is not the owner of the worker a `403 Forbidden` response is given.

### Parameters

#### (type=jpg)
The image type. This defaults to standard-quality jpg.

#### (resolution=800x600)
The resolution of the image. This can be either:

* 800x600
* 1024x768
* 1280x1024

### Response
The returned object will return the base64 encoded image data. You can show this on a page by simply doing:

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1

    {
      "mime": "image/jpg",
      "data": "base64...encoded...image...data"
    }

This can be displayed on any standard HTML page with the following code snippit:

```html
<img src="data:{mime};base64,{data}">
```

Where {mime} is replaced with the returned mime in the json object and {data} is replaced by the base64 encoded data returned in the data tag.

## Terminating a worker
Use this method to terminate a worker. Useful if you set the worker up to run indefinitely or if you've received all the information needed and you want to save on credit time.

  DELETE  /worker/:id
  
The id is the id returned when you first created the worker. Once called the browser instance will be immediately terminated and will no longer be accessible.

> This call requires authentication. If the request was made unauthorised a `401 Unauthorized` response is given. Alternatively if the authorised user is not the owner of the worker a `403 Forbidden` response is given.

## Getting results
This is the easiest way to fetch results given by the worker. Every worker instance is given an extra method to the `window` object called `postData` which if called all arguments of which will be placed on a stack which are fetch-able using this HTTP call. If the worker has been terminated or doesn't exist a `404 Not Found` response is given.

  GET /worker/:id
  
> NOTE! Once you've fetched the results they'll be removed from the stack. It's therefore recommended to poll this method for updates.

> This call requires authentication. If the request was made unauthorised a `401 Unauthorized` response is given. Alternatively if the authorised user is not the owner of the worker a `403 Forbidden` response is given.
  
### Response
The response is JSON encoded array containing all the posted data being stored on the stack, for example:

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1

    [ 'hello', 'world', 'foo', 'bar' ]
      
If the stack is empty an empty array `[]` is returned.

#### Example Request-Flow

```javascript
window.postData('hello', 'world');
window.postData('bob');
```

```
GET /worker/:id
  -> [ "hello", "world", "bob" ]
```  

```javascript
window.postData({ foo: 'bar'});
```    

```
GET /worker/:id
  -> [ { "foo": "bar" } ]
```
        
## Sending message/data
Just like a WebWorker if this method is called the `window.ondata` function is called. Bind to this event and every time this API call is made the data passed to it will be forwarded to this method. Again all data is JSON encoded.

  PUT /worker/:id

> The server will close the request after the method call has been processed and invoked. Allowing for immediate postData responses to be polled for immediately afterwards.

> This call requires authentication. If the request was made unauthorised a `401 Unauthorized` response is given. Alternatively if the authorised user is not the owner of the worker a `403 Forbidden` response is given.

### Example Request-Flow

```javascript
window.ondata = function(names) {
  window.postData('Hello, ' + names.join(' ') + '!');
}
```  

```
PUT /worker/:id
[ "John", "Thomas" ]
```

```
GET /worker/:id
  -> "Hello, John Thomas!"
```
    
## Getting account credit
It's useful to know how much time credit you have left in your account. This API call will help you find out.

    GET /account/credit
  
> Note this call required authentication. If an unauthorised request was made a `401 Unauthorized` response is returned.

### Response Example

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1

    { "remaining": 3598, "used": 2 }

`remaining` Is the number of seconds remaining.
`used` Is the number of seconds used (in total).

## Getting Worker Status
Sometimes you will need to check on the status of a worker. Not to be confused with the state of the javascript environment within the worker this method simply determines whether the worker is terminated or not.
    
    HEAD /worker/:id
    
> This call requires authentication. If the request was made unauthorised a `401 Unauthorized` response is given. Alternatively if the authorised user is not the owner of the worker a `403 Forbidden` response is given.

If the worker has been terminated a `404 Not Found` response is given. Otherwise a `200 Success` is returned.