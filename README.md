# BrowserStack API
The following denotes the HTTP-based API for BrowserStack. The URL prefix should take the following format for all queries listed on this page.

    http://api.browserstack.com/<version>/<auth>/
    
`version` denotes the version identifier for the API you are targetting. This document outlines the `v1` API specification.
`auth` denotes the authentication token which you can get from your settings page. For example:

    http://api.browserstack.com/1/da39a3ee3255bfef5e6b4b0d/browsers?type=msie

Will list all all MSIE browsers available for the user with token da39a3ee3255bfef5e6b4b0d.

## GET /browsers

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
  
An example for `msie` is shown in the summary section above.


Gets a list of browsers that can be emulated/spawned using the API. Specifying a type as a get parameter will filter the list of browsers to that specific type. An example of this would be `/browsers?type=msie` which will return a list of Internet Explorer browsers versions available. The key to the returned JSON object represents the ID for that browser. The ID is unique and will never change.

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
  
## POST /worker
Spawn a new browser instance as a browser worker. Browser workers are completely untouched. You can perform and ajax get/post calls to remote servers you want. Be sure to perform proper error handling yourself. Workers can continue to be run until you run out of credit.

The POST request will return a header along the following lines with a worker id. You can use this worker ID to find out how long it has been running. The response will block until the worker has been spawned and resolved the url/loaded the data and return a status code that represents the health of it. `200 Success` means its all well and good.

For example:
  
  HTTP/1.1 200 Success
  X-Worker-Id: 3255bfef

### Parameters

#### browser
The browser ID. These can be found from the `/browser` API call outlined above.

#### url
A valid url to navigate the browser to. This should be used instead of `data` and not together.

> Must be base-64 encoded  

#### data
A valid HTML content page to run. This should be used instead of `url` and not together.

> Must be base-64 encoded.