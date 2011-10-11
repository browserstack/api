# BrowserStack API
The following denotes the HTTP-based API for BrowserStack. 

## GET /browsers?type=<type>
Gets a list of browsers that can be emulated/spawned using the API. Specifying a type as a get parameter will filter the list of browsers to that specific type. An example of this would be `/browsers?type=msie` which will return a list of Internet Explorer browsers versions available. The key to the returned JSON object represents the ID for that browser. The ID is unique and will never change.

  {
    "da39a3ee" : {
      id: 'da39a3ee',
      type: 'msie',
      version: 7,
      os: 'win7',
      agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
    },
  {
    id: '5e6b4b0d',
    type: 'firefox',
    version: 2.5,
    os: 'win7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
  },
  {
    id: '3255bfef',
    type: 'firefox',
    version: 3.0,
    os: 'win7',
    agent: 'Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 6.0)'
  } ... }
  
## POST /worker
Spawn a new browser instance as a worker. The window variable will be left un-touched except for 1 method `postMessage`. Similar to the WebWorker API this method is assigned to the global/window object and when called will

### Parameters

#### browser

#### url
  A valid url to navigate the browser to. This should be used instead of `data` and not together.
  
#### callback
A valid url to POST a response to. This will occur when the script calls the `window.postMessage`. The posted data is JSON encoded in the following format:
  
  {
    browser: "da39a3ee",
    date: "2010-04-10T14:10:01-07:00"
    data: ""
  }

`browser` The browser ID that processed the request
`date` The date the page was loaded and processed
`data` The response