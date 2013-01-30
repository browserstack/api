# API Overview
The following denotes the HTTP-based API for [BrowserStack](http://www.browserstack.com). It provides browser-as-a-service for automated cross-browser testing. The goal is to provide a simple service which can easily be used by any browser testing framework.

### Schema
All requests are made to `http://api.browserstack.com/VERSION/` and all returned data is done so in JSON-format. The version this documentation outlines is 2.

    $ curl -i http://api.browserstack.com/2

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    X-API-Version: 2
    Content-Length: 2

    {}

All date formats are given in ISO-8601 which can be processed natively with `Date.parse` in compatible JavaScript environments.

### Error Handling
All requests are pre-processed and validated. This section outlines how we handle errors within the API and respond to them.

1. All API requests are validated. The following is an example output for a required parameter that wasn't given.
  
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
All methods need to authenticate who you are. Before spawning browser workers and deleting a worker for example. Authentication is done using your username/password within the HTTP request. For example:

    $ curl -u "username:PASSWORD" http://api.browserstack.com/2

> A `401 Unauthorized` response is given if an unauthorized request is made.    

### HTTP Verbs
The API is kept concise and simple by making use of relevant HTTP verbs on each requests. The specifications for these are vague and their use within this API is specific but in general we follow the following rules:

  * `HEAD` - Performs the request to asses the status of a resource and expects no content response.
  * `GET` - Used to retrieve resources.
  * `POST` - Used to create new resources.
  * `PUT` - Used to update resources.
  * `DELETE` - Used to delete resources.



# API Documentation

## Getting Available Browsers
Fetches all available browsers.

    GET /browsers
  
### Output

```javascript
{
  'win':
    [
      {
        browser: 'ie',
        version: 7.0,
      },
      {
        browser: 'firefox',
        version: 2.0,
      },
      {
        browser: 'chrome',
        version: 14.0,
      } ...
    ]
  ,
  'mac':
    [
      {
        browser: 'firefox',
        version: 11.0,
      },
      {
        browser: 'chrome',
        version: 14.0,
      } ...
    ]
  ,
  'ios':
    [
      {
        device: 'iPhone 4',
        version: '4.0'
      },
      {
        device: 'iPad 2',
        version: '4.3.2'      
      }...
    ]
}
``` 


## Create a New Browser Worker
A browser worker is simply a new browser instance. A user can start multiple browser worker at a time. All browser workers when created are pushed in a queue and they run when their turn comes. We make sure that your browser worker starts running as soon as possible. Your testing time is calculated from the time when browser worker starts running.

    POST /worker

> This call requires authentication. A `401 Unauthorized` response is given if an unauthorized request is made.

Once a worker has been spawned you can then control this browser instance remotely.

### Parameters
A valid request must contain a `os`, `browser` OR `device`, `version`, and a `url`. `timeout` is optional but defaults to 300 seconds.

#### os
A valid OS. A list of supported OS's are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### browser
A valid browser. A list of supported browsers are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### device
A valid device. A list of supported devices are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### version
A valid browser version. List of supported browser versions are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.
 
#### (timeout=30)
A number in seconds before the worker is terminated. The default value is 300 seconds. Timeout = 0 also defaults to 300 seconds.

> IMPORTANT! Irrespective of timeout parameter, a browser worker is alive for a maximum time of 3600 seconds.

#### (url)
A valid url to navigate the browser to.

> Make sure the url is encoded. JavaScript: encodeURI(url), PHP: urlencode($url), 


### Response
The response will be returned when the worker has been setup and initialized. This involves loading the HTML data or navigating to the url given depending on the setup parameters. Use the id returned to perform any further communications etc.

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 2
    
    {
      "id": "da39a3ee"
    }
 


## Terminating a worker
Use this method to terminate a worker. Useful if you set the worker up to run indefinitely or if you've received all the information needed and you want to save on credit time.

  DELETE  /worker/:id
  
The id is the id returned when you first created the worker. Once called the browser instance will be immediately terminated and will no longer be accessible.

> This call requires authentication. If the request was made unauthorized a `401 Unauthorized` response is given. Alternatively if the authorized user is not the owner of the worker or id does not exist a `403 Forbidden` response is given.


## Getting Worker Status
Sometimes you will need to check on the status of a worker. Not to be confused with the state of the javascript environment within the worker this method simply determines whether the worker is in queue, running or terminated.

		GET /worker/:id

> This call requires authentication. If the request was made unauthorized a `401 Unauthorized` response is given. Alternatively if the authorized user is not the owner of the worker a `403 Forbidden` response is given.

If the worker has been terminated an empty response is given. Otherwise you get a response with status and browser details.

```javascript
{
  status: 'running',
  browser: 'ie', 
  version: '6.0',
  os: 'win'
}
```

If you want to know the list of your current workers with their status, use the following method.

		GET /workers

This method will return the list of workers whose status is either `queue` or `running`. For example:

```javascript
[
  {
    id: 3253,
    status: 'running',
    browser: 'ie', 
    version: '6.0',
    os: 'win'
  },
  {
    id: 3254,
    status: 'queue',
    device: 'Samsung Galaxy Tab 8.9',
    version: '2.2',
    os: 'android'
  } ...
]
```


## Getting API Status
If you want to know the status of your API, use the following method

    GET /status

This will return the current status of API, like how much API time has been used and how many workers are runnning parallely.

```javascript
  {
    used_time: '4235.4',
    total_available_time: '6000', 
    running_windows_sessions: '1',
    windows_sessions_limit: '1'
    running_mac_sessions: '1',
    mac_sessions_limit: '2'
  }
```
The time returned is in seconds.

If a user runs out of API time, all requests will return following response

```javascript
  {
    message: "You have run out of API time"
  }
```
