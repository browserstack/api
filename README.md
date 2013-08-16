# API Overview
The following denotes the HTTP-based API for [BrowserStack](http://www.browserstack.com). It provides browser-as-a-service for automated cross-browser testing. The goal is to provide a simple service which can easily be used by any browser testing framework.

### Schema
All requests are made to `http://api.browserstack.com/VERSION/` and all returned data is done so in JSON-format. The version this documentation outlines is 3.

    $ curl -i http://api.browserstack.com/3

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    X-API-Version: 3
    Content-Length: 3

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
All methods need to authenticate who you are. Before spawning browser workers and deleting a worker for example. Authentication is done using your username and the BrowserStack access key within the HTTP request. For example:

    $ curl -u "username:access_key" http://api.browserstack.com/3

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
  'Windows':
    {
      '7': 
        [
        {
          browser: 'ie',
          browser_version: 8.0,
        },
        {
          browser: 'firefox',
          browser_version: 2.0,
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        } ...
        ],
      'XP':
        [
        {
          browser: 'ie',
          browser_version: 7.0,
        },
        {
          browser: 'firefox',
          browser_version: 10.0,
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        } ...
        ],
    },
  'OS X':
    {
      'Snow Leopard':
        [
        {
          browser: 'firefox',
          browser_version: 11.0,
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        }, ...
        ],
      'Lion':
        [
        {
          browser: 'firefox',
          browser_version: 11.0,
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        } ...
        ],
    },
  'ios':
    {
      '4.0':
        [
        {
          browser: "Mobile Safari",
          browser_version: null,
          devices: ["iPhone 4"],
        }
        ],
      '6.0':
        [
        {
          browser: "Mobile Safari",
          browser_version: null,
          devices: ["iPhone 4S (6.0)", "iPhone 5", "iPad 3rd (6.0)"],
        }
        ]
    },
}
``` 

A flat parameter can also be passed to get browsers in a flat structure

    GET /browsers?flat=true
  
### Output

```javascript
[
{
  device: null,
  os_version: "8",
  os: "Windows",
  browser_version: "5.1",
  browser: "safari"
},
{
  device: null,
  os_version: "8",
  os: "Windows",
  browser_version: "16.0",
  browser: "firefox"
},
{
  device: null,
  os_version: "8",
  os: "Windows",
  browser_version: "17.0",
  browser: "firefox"
},
{
  device: null,
  os_version: "8",
  os: "Windows",
  browser_version: "18.0",
  browser: "firefox"
},
{
  device: null,
  os_version: "8",
  os: "Windows",
  browser_version: "10.0",
  browser: "ie"
}....
``` 

You can get beta/dev versions of browsers by using  "all" parameter:

    GET /browsers?all=true
  
### Output

```javascript
{
  'Windows':
    {
      'XP': 
        [
        {
          browser: 'ie',
          browser_version: 7.0,
        },
        {
          browser: 'firefox',
          browser_version: '28.0 beta',
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        } ...
        ],
    },
  'OS X':
    {
      'Snow Leopard':
        [
        {
          browser: 'firefox',
          browser_version: 11.0,
        },
        {
          browser: 'chrome',
          browser_version: '27.0 beta',
        }, ...
        ],
      'Lion':
        [
        {
          browser: 'firefox',
          browser_version: 11.0,
        },
        {
          browser: 'chrome',
          browser_version: 14.0,
        } ...
        ],
    },
}
``` 

## Create a New Browser Worker
A browser worker is simply a new browser instance. A user can start multiple browser worker at a time. All browser workers when created are pushed in a queue and they run when their turn comes. We make sure that your browser worker starts running as soon as possible. Your testing time is calculated from the time when browser worker starts running.

    POST /worker

> This call requires authentication. A `401 Unauthorized` response is given if an unauthorized request is made.

Once a worker has been spawned you can then control this browser instance remotely. You can also look at the testing session status at the automate dashboard: <http://www.browserstack.com/automate>. This will provide you the general details about the session and a live preview of the remote machine.

### Parameters
A valid request must contain a `os`, `os_version`, and a `url`. `timeout` is optional but defaults to 300 seconds.
  `browser` and `browser_version` are optional for mobile os but required for desktop os.

#### os
A valid OS. A list of supported OS's are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### os_version
A valid OS Version. A list of supported OS Version's are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### browser
A valid browser. A list of supported browsers are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### device
A valid device. A list of supported devices are given using the `GET /browsers`. If a device is not provided it defaults to the first device available for that os version. See the _Getting Available Browsers_ above for details.

#### browser_version
A valid browser version. List of supported browser versions are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.
 
#### (timeout=30)
A number in seconds before the worker is terminated. The default value is 300 seconds. Timeout = 0 also defaults to 300 seconds.

> IMPORTANT! Irrespective of timeout parameter, a browser worker is alive for a maximum time of 3600 seconds.

#### (url)
A valid url to navigate the browser to.
> Make sure the url is encoded. JavaScript: encodeURI(url), PHP: urlencode($url), 

#### (name)
Provide a name to the session/worker.

#### (build)
Optional name of the build the session is running under.

#### (project)
Optional name of the project the build is under.

### Response
The response will be returned when the worker has been setup and initialized. This involves loading the HTML data or navigating to the url given depending on the setup parameters. Use the id returned to perform any further communications etc.

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 3
    
    {
      "id": "da39a3ee"
    }
 


### Screenshots
Use this method to take a screenshot at the current state of the worker.

  GET /worker/:id/screenshot(.format)

Acceptable formats are `json`, `xml` and `png`. This information can also be provided via the HTTP `Accept` headrs: `text/json`, `text/xml`, `image/png` respectively.


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
  browser_version: '6.0',
  os: 'Windows',
  os_version: 'XP'
}
```

If you want to know the list of your current workers with their status, use the following method.

		GET /workers

This method will return the list of workers whose status is either `queue` or `running`. For example:

```javascript
[
  {
    status: 'running',
    browser: 'ie', 
    version: '6.0',
    os: 'Windows',
    os_version: 'XP'
  },
  {
    status: 'queue',
    device: 'Samsung Galaxy Tab 8.9',
    os: 'android',
    os_version: '2.2'
  } ...
]
```


## Getting API Status
If you want to know the status of your API, use the following method

    GET /status

This will return the current status of API, like how much API time has been used and how many workers are runnning parallely. All the paid plans have no time limits, only limit on parallel workers you can create.

```javascript
  {
    used_time: 4235.4,
    total_available_time: 6000,
    running_sessions: 1,
    sessions_limit: 1
  }
```
The time returned is in seconds.

If a user runs out of API time, all requests will return following response

```javascript
  {
    message: "You have run out of API time"
  }
```
