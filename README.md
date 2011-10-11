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

# API Documentation

## Getting Available Browsers
Fetches all available browsers and IDs that represent them. IDs will be unique to each browser and will never change for the continued existence of that API version.

    GET /browsers(/:type)

### Parameters

#### Type
The browser type. Currently theses are:

  * msie
  * firefox
  * chrome
  * ios
  * opera
  * android
  * ...
  
### Output

```javascript
{
  "da39a3ee" : {
    id: 'da39a3ee',
    type: 'msie',
    version: 7,
    os: 'win7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
  },
  "5e6b4b0d": {
    id: '5e6b4b0d',
    type: 'firefox',
    version: 2.5,
    os: 'win7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
  },
  "3255bfef": {
    id: '3255bfef',
    type: 'firefox',
    version: 3.0,
    os: 'win7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
  } ...
}
```
  
## Create a New Browser Worker
Spawn a new browser instance as a browser worker. Browser workers are completely untouched. You can perform and ajax get/post calls to remote servers you want. Be sure to perform proper error handling yourself. Workers can continue to be run until you run out of time credit.

    POST /worker

> This call requires authentication.

Once a worker has been spawned you can see it listed in your dashboard account online at browserstack.com. You can then login to this worker VM and do any interactions you want if you need to.

### Parameters
A valid request must contain a `browser` and either a `url` or `data` parameter but not both. `Timeout` is optional but defaults to 30seconds.

#### browser
The browser ID. These can be found from the `/browser` API call outlined above.

#### (timeout=30)
A number in seconds before the worker is terminated. Set this to 0 to keep the worker alive indefinitely.

> IMPORTANT! If you have set the timeout to 0. Make sure you remember to terminate the worker otherwise it will continue to use up your credits.

#### (url)
A valid url to navigate the browser to. This should be used instead of `data` and not together.

> Must be base-64 encoded  

#### (data)
A valid HTML content page to run. This should be used instead of `url` and not together.

> Must be base-64 encoded.

### Response
The response will be returned when the worker has been spawned and the url loaded. The ID can be used to retrieve screenshots or terminate the worker later on.

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1
    
    {
      "id": "da39a3ee"
    }

## Getting a Worker Screenshot
This method will allow you to get a screenshot image of the browser window.

    GET /worker/screen/:id
    
`id` represents the worker id. This is returned from a successful `POST /worker` call and is required to get a screenshot (obviously).

> This call requires authentication. And requires you to be the same user who originally created the worker.

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