# Introduction to Service Worker

## Table of Content
| Topics        |
| ------------- |
| Real world problems      |
| Introduction to Service Worker      |
| Benefits of Offline First |
| Registering a Service Worker |
| Lifecycle of Service Worker |
| Real world example |

## Introduction to Service Worker
Service worker is a JavaScript file that, run seperately in its own thread. Intercepts network request and store or retrive data from cache and provide push message to web client even when offline. It doesn't have DOM support and only run on HTTPS.

Service worker doesn't have access to DOM, so what can we do with JavaScript? Lets discus this.

Service worker is like a middle man (or middleware) between the webpage and the internet, it handle the network request. When a webpage request the server for HTML, CSS files or any API request it pass through the service worker, it is up to us how we code the service worker to deal with these request like we can send that request to server or we can look for the request in cache if exist then show from cache. It is up to us what approach is better for us to provide a good user experience.

## Benefits of Offline First
Before service worker, `AppCache` was used by the developer to cache data. In `AppCache` first network request was made by the site to server and then cache the request when it is completed and whenever the request was failed it was retrive from `AppCache`. The drawback of this is, user kept waiting till the network request made or failed which is a bad exprience for the user, so that why `AppCache` has been depriciated and `Service Worker` was introduced on `08 May, 2014`.

[Udacity Course]: https://www.udacity.com/course/offline-web-applications--ud899
Below is the image show the working of `AppCache`: Referenced from [Udacity Course]
[appCache]: images/AppCache-diagram.gif "AppCache diagram"
![alt text][appCache]

As `Service Worker` was introduced, a new approach come into play to develop a web app `Offline First` i.e a request to a server is pass through service worker, it will look for data in cache and show the content of the web page that already cache meanwhile gets the respone from the server and show the rest of the content by this approach user wouldn't wait for page to completely load and shouldn't see the white screen all the time. Below the image describe the flow of service worker. Referenced from [Udacity Course]
[offlineFirst]: images/Offline-First-diagram.gif "Offline First diagram"
![alt text][offlineFirst]

## Registering a Service Worker
To register a service worker, first we need to check if the service worker is supported by the broswer or not, to do so we need to check for service worker object in `navigator`. `Navigator` object contains the information about the user agent and browser.

```javascript
if (navigator.serviceWorker) {
  navigator.serviceWorker.register('/sw.js').then(function(reg) {
    console.log('Yay!!');
  }, function(err) {
    console.log('Boo!!');
  });
}
```
where `sw.js` is the service worker file located at the root level of project (in my case) it could be of any name and located anywhere inside the project but it is good to have it on root level.

## Lifecycle of Service Worker
Service worker handle requests to the server. It has a few steps, and to show that I’ll show your a nice picture from [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers#Browser_compatibility):
[workerLifeCycle]: images/worker-lifecycle.png "Service worker Lifecycle"
![alt text][workerLifeCycle]

### Parsed Event
When we first attempt to register a Service Worker, the user agent parses the script and obtains the entry point. If parsing is successful (and some other requirements, e.g. HTTPS, are met), we will have access to the Service Worker registration object. This contains information about the state of the Service Worker as well as it’s scope.
```javascript
if ('serviceWorker' in navigator) {
	navigator.serviceWorker.register('./sw.js')
	.then(function(registration) {
		console.log("Service Worker Registered", registration);
	})
	.catch(function(err) {
		console.log("Service Worker Failed to Register", err);
	})
}
```
### Install Event
Once the Service Worker script has been parsed, the user agent attempts to install it and it moves to the installing state. In the Service Worker `registration` object, we can check for this state in the `installing` child object.
```javascript
navigator.serviceWorker.register('./sw.js').then(function(registration) {
  if (registration.installing) {
      // Service Worker is Installing
  }
});
```
During the installing state, the install event in the Service Worker script is carried out. The “install” event is the first event a service worker gets, and it only happens once. A promise passed to ```event.waitUntil()``` signals the duration and success or failure of your install. A service worker won't receive events like ```fetch``` and ```push``` until it successfully finishes installing and becomes "active".

By default, a page's fetches won't go through a service worker unless the page request itself went through a service worker. So you'll need to refresh the page to see the effects of the service worker. An example image shown by [Jake Archibald](https://developers.google.com/web/resources/contributors/jakearchibald) in his [blog](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle):
[installEvent]: images/service-worker-install.gif "Service worker install"
![alt text][installEvent]

To listen the install event of a service worker, we need to add the following code in our sw.js file:
```javascript
self.addEventListener('install', function(event) {
  event.waitUntil(
    // Open cache storage and do something
  );
});
```
### Installed / Waiting
If the installation is successful, the Service Worker moves to the installed (also called waiting) state. In this state it is a valid, but not yet active, worker. It is not yet in control of the document, but rather is waiting to take control from the current worker.

In the Service Worker `registration` object, we can check for this state in the `waiting` child object.
```javascript
navigator.serviceWorker.register('./sw.js').then(function(registration) {
  if (registration.waiting) {
      // Service Worker is Waiting
  }
});
```
This can be a good time to notify the app user that they can update to a new version, or automatically update for them.
### Activate Event
After the installation of service worker, if there any existing service worker controlling the client, then this worker goes to a `waiting` state. 
The activate event fires once the old service worker is gone, and your new service worker is able to control clients. This is the ideal time to do stuff that you couldn't do while the old worker was still in use, such as migrating databases and clearing caches.
The waiting phase means you're only running one version of your site at once, but if you don't need that feature, you can make your new service worker activate sooner by calling “self.skipWaiting()”. An example image shown by [Jake Archibald](https://developers.google.com/web/resources/contributors/jakearchibald) in his [blog](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle):
[activateEvent]: images/service-worker-activate.gif "Service worker activate"
![alt text][activateEvent]


To listen the activate event of a service worker, we need to add the following code in our sw.js file:
```javascript
self.addEventListener('activate', function(event) {
  event.waitUntil(
    // Delete cache
  );
});
```

### Activated
If activation is successful, the Service Worker moves to the active state. In this state, it is an active worker in full control of the document. In the Service Worker registration object, we can check for this state in the active child object.
```javascript
navigator.serviceWorker.register('./sw.js').then(function(registration) {
  if (registration.active) {
      // Service Worker is Active
  }
});
```
When a Service Worker is active, it can now handle the functional events - fetch, and message.
```javascript
self.addEventListener('fetch', function(event) {
  // Do stuff with fetch events
});

self.addEventListener('message', function(event) {
  // Do stuff with postMessages received from document
});
```
### Redundant 
A Service Worker can become redundant for one of the following reasons -

- If the installing event failed

- If the activating event failed

- If a new Service Worker replaces it as the active service worker
## CacheStorage
The Cache API is a system for storing and retrieving network requests and corresponding responses. The Cache API was created to enable Service Workers to cache network requests so that they can provide appropriate responses even while offline. However, the API can also be used as a general storage mechanism.
### What can be stored? 
The caches only store pairs of “Request” and “Response” objects, representing HTTP requests and responses. However, the requests and responses can contain any kind of data that can be transferred over HTTP.
The API is exposed via the global “caches” property, so you can test for the presence of the API with a simple feature detection:
```javascript
if('caches' in window) {
  // HAS SUPPORT!!
}
```
The API can be accessed from a window, iframe, worker, or service worker
### Methods of CacheStorage
| Methods       | Description           |
| ------------- |:-------------:|
| open(cacheName)    | Returns a Promise that resolves to the Cache object matching the cacheName (a new cache is created if it doesn’t already exist).  |
| match()    | Checks if a given Request is a key in any of the Cache objects that the CacheStorage tracks, and returns a Promise that resolves to that match.      |
| has(cacheName)    | Returns a Promise that resolves to “TRUE” if a Cache object matching the cacheName exits.     |
| keys()    | Returns a Promise that will resolve an array containing strings corresponding to all of the named Cache objects tracked by the CacheStorage. Use this method to iterate over a list of all the Cache object.     |
| delete(cacheName)    | Finds the Cache object matching the cacheName if found, delete the Cache object and returns a Promise that resolves to “TRUE”. If no Cache object is found, it returns “FALSE”.     |

#### Creating or opening a cache
```javascript
// “caches” is a global read-only variable, which is the instance of CacheStorage.
caches.open('my-cache').then((cache) => { 
  // do something with cache…
});
```
> Note: Give your cache names a unique prefix so that it don’t override the other domain cache
For ex: ‘mysite-static-v1’

#### Adding to cache
```javascript
caches.open('test-cache').then(function(cache) {
  cache.addAll(['/', '/images/logo.png'])
    .then(function() {
	// Cached!
    });
});
caches.open('test-cache').then(function(cache) {
  cache.add('/images/logo.png')
    .then(function() {
	// Cached!
    });
});
```
> The addAll method accepts an array of URLs that should be cached, to add single URL, use the add method

#### put()
put() is similar to add() but in, put() we can add response object.
```javascript
fetch('/page/1').then(function(response) {
  return caches.open('test-cache')
    .then(function(cache) {
       return cache.put('/page/1', response);
    });
});
```
#### Retrieving Cache
```javascript
caches.open('test-cache').then(function(cache) {
  cache.keys().then(function(cachedRequests) {
    console.log(cachedRequests); // [Request, Request]
  });
});

caches.open('test-cache').then(function(cache) {
  cache.match('/page/1')
    .then(function(matchedResponse) {
       console.log(matchedResponse);
    });
});
/*
 Request {
   bodyUsed: false
   credentials: "omit"
   headers: Headers
   integrity: ""
   method: "GET"
   mode: "no-cors"
   redirect: "follow"
   referrer: ""
   url: "https://staging.pitchworthy.co/images/logo.png"
}*/
/*Response {
   body: (...),
   bodyUsed: false,
   headers: Headers,
   ok: true,
   status: 200,
   statusText: "OK",
   type: "basic",
   url: "https://staging.pitchworthy.co/users/me"
 }
 */

```
#### caches.keys
To get the names of existing caches, use “caches.keys”
```javascript
caches.keys().then(function(cacheKeys) {
  console.log(cacheKeys); // ex: ["test-cache"]
});
```
#### Deleting a Cache
```javascript
caches.delete('test-cache').then(function() {
  console.log('Cache successfully deleted!');
});
```
#### Delete an old cache and add new one
You often need to delete an old cache and add new one.
```javascript
// Assuming `CACHE_NAME` is the newest name. Time to clean up the old!
var CACHE_NAME = 'version-8';
caches.keys().then(function(cacheNames) {
  return Promise.all(
    cacheNames.map(function(cacheName) {
      if(cacheName != CACHE_NAME) {
        return caches.delete(cacheName);
      }
    })
  );
});
```