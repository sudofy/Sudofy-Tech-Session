# Introduction to Service Worker

## Table of Content
| Topics        |
| ------------- |
| Overview      |
| App Introduction      |
| Installing/Running the App |
| Registering our first Service Worker |
| Exploring the Dev Tools |
| Hijacking Request |
| Caching and Serving Assets |
| Unobtrosive App Updates |
| Trigger latest version |
| Update Data using the IndexedDB |

## Overview
Service worker is a JavaScript file that, run seperately in its own thread. Intercepts network request and store or retrive data from cache and provide push message to web client even when offline. It doesn't have DOM support and only run on HTTPS.

Service worker doesn't have access to DOM, so what can we do with JavaScript? Lets discus this.

Service worker is like a middle man (or middleware) between the webpage and the internet, it handle the network request. When a webpage request the server for HTML, CSS files or any API request it pass through the service worker, it is up to us how we code the service worker to deal with these request like we can send that request to server or we can look for the request in cache if exist then show from cache. It is up to us what approach is better for us to provide a good user experience.

## App Introduction
We have an app called `Wittr`, it is a real time social networking web app where user share their posts and give comments/feedback on them. This site is fully developed but not a total breakdown on low connectivity and offline mode. So we going to implement the service worker to tackle the blank screen on low connectivity and offline mode.

## Installing/Running the App
To install the app on you local machine, you first need to install [Git](https://git-scm.com/) if you have it that's great. If you dont know about `Git` that's not an issue we are just using the basic of `Git`. Those who dont know Git, it is a version controlling system.
To download or clone the project to your local machine, run command on your terminal or command-line.

`git clone https://github.com/farazsarwar113/wittr`

After cloning the project run command `npm install` to install all the dependencies required by the project.
To run the app run `gulp serve`. It will run the app on `http://localhost:8888` and server on `http://localhost:8889`.

## Registering our first Service Worker
We need to work on only two files throughout the app `public/js/sw/index.js` and `public/js/main/IndexController.js`. This project is configured in a way that when we run command `gulp serve` it compiled all the css files in css folder and javascript files in js folder and service worker file in the root of the project with the `sw.js`.

To register a service, first we need switch branch.

`git reset --hard` if you have any local changes run this command

`git checkout task-register-sw`

We will work on `public/js/sw/index.js` inside `_registerServiceWorker` function. First we need to check if the service worker is supported by the broswer or not, to do so we need to check for service worker object in `navigator`. `Navigator` object contains the information about the user agent and browser.

```javascript
if (!navigator.serviceWorker) return;
navigator.serviceWorker.register('/sw.js').then(function(reg) {
  console.log('Registeration successfull');
}, function(err) {
  console.log('Error in registeration');
});
```

## Exploring the Dev Tools

## Hijacking Request
### Custom Response
We can give a custom response to browser request using service worker, to handle the network request we will add a `Event Listner` on `public/js/sw/index.js`.

```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response('Hello World')
  );
})
```
This will be the output we get for any request browser do.

```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response('Hello World <b> Faraz </b> ')
  );
})
```
What if i add the html tag inside my `Hello World` string. let see.
It will render as a string because in the response header the `Content-Type` is set to be `plain/text` by default, so we set it to `text/html`.
```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response('Hello World <b> Faraz </b> ', { headers: { 'Content-Type': 'text/html' }})
  );
})
```
### Respond with gif
To get to the position of code done above run:

`git reset --hard`

`git checkout task-customer-response`

To respond with gif for the requested url:

```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response(
      fetch('/imgs/devil.gif')
    )
  );
})
```

The above code respond every request with gif but that's not the better way, now we respond only the request of img with gif response. To do so `log` event and check for the url and react according to that, let me show how to do that:

`git reset --hard`

`git checkout gif-response`

```javascript
self.addEventListner('fetch', function(event) {
  console.log(event.request.url);
  if (event.request.url.endsWith('.jpg')) {
    event.respondWith(
      new Response(
        fetch('/imgs/devil.gif')
      )
    );
  }
})
```
### Handle 404 and failed state
To handle 404 and failed state we look at then response status:

```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response(
      fetch(event.request.url).then(function(response) {
        if (response.status === 404) {
          return new Response('NOT FOUND');
        }
      }).catch(function(err) {
        return new Response('ERROR'); 
      })
    );
  );
})
```
Now we will show the `devil.gif` to 404 status: To do this first get on the branch
`git reset --hard`

`git checkout error-handling`
```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    new Response(
      fetch(event.request.url).then(function(response) {
        if (response.status === 404) {
          return fetch('/imgs/devil.gif')
        }
      }).catch(function(err) {
        return new Response('ERROR'); 
      })
    );
  );
})
```

## Caching and Serving Assets
To cache the static files we have to create cache and add the files on install event of service worker.

`git reset --hard`
`git checkout task-install`

```javascript
self.addEventListner('install', function(event) {
  event.waitUntil(
    caches.open('wittr-v1').then(function(cache) {
      cache.addAll([
        '/',
        '/style.css',
        '/main.js'
      ])
    })
  )
})
```

Now we cached the data in the CacheStorage, but we request the site we will still be kept waiting. Now we have to check if there is any data in cache we will show it from there otherwise show it from network.

`git reset --hard`
`git checkout task-cache-response`

```javascript
self.addEventListner('fetch', function(event) {
  event.respondWith(
    caches.match(event.request.url).then(function(response) {
      if (response) return response;
      return fetch(event.request)
    })
  )
})
```

Till now we acheive to show something rather than a blank page or a dinosur but we have still to do alot, we are not getting the updated app or nor doesnt know about the update is here. So now what problems we have to acheive right now:

- Unobtrusive app updates
- Get the user on to the latest version
- Continously update the cache of posts
- Cache photos
- Cache avatar


## Unobtrusive app updates
If we update our css and force reload is checked we can see the changes in browser but if we unchecked the force update and the change the css the broswer doent detch the changes and show the old one because we didnt update the service worker and another service worker is active.

`git reset --hard`
`git checkout task-handling-updates`

To deal with this we will delete the old cache on the activate event. 
The easy step is:

```javascript
self.addEventListner('activate', function(event) {
  event.waitUntil(
    caches.delete('wittr-static-v1')
  )
})
```
The scalable way:
```javascript
var staticCacheName = 'wittr-static-v2';

self.addEventListner('activate', function(event) {
  event.waitUntil(
    Promise.all(
      caches.keys().then(function(cacheNames){
        cacheNames.filter(function(cacheName){
          return cacheName.startWith('wittr-') && cacheName != staticCacheName;
        }).map(function(cacheName) {
          return caches.delete(cacheName)
        })
      })
    )
  )
})
```
## Trigger latest version
Now we will look at how we will notify user that there is an update ready to install. First we look at the properties and method given by service worker.

`git reset --hard`
`git checkout sw-register-methods`

`git checkout task-update-notify`

```javascript

IndexController.prototype._registerServiceWorker = function() {
  if (!navigator.serviceWorker) return;

  var indexController = this;

  navigator.serviceWorker.register('/sw.js').then(function(reg) {
    // TODO: if there's no controller, this page wasn't loaded
    // via a service worker, so they're looking at the latest version.
    // In that case, exit early
    if (!navigator.serviceWorker.controller) return;

    // TODO: if there's an updated worker already waiting, call
    // indexController._updateReady()
    if(reg.waiting) {
      indexController._updateReady()
      return;
    }
    // TODO: if there's an updated worker installing, track its
    // progress. If it becomes "installed", call
    // indexController._updateReady()
    if (reg.installing) {
      indexController._trackInstalling(reg.installing);
      return;
    }

    // TODO: otherwise, listen for new installing workers arriving.
    // If one arrives, track its progress.
    // If it becomes "installed", call
    // indexController._updateReady()
    reg.addEventListner('updatefound', function() {
      indexController._trackInstalling(reg.installing);
    })
  });
};

IndexController.prototype._trackInstalling = function(worker) {
  var indexController = this;
  worker.addEventListner('statechange', function() {
    if (worker.state === 'installed') {
      indexController._updateReady()
    }
  });
}

```

By the above code we can now sent notification to user about the update but how we will trigger the update. let see: First we see some of the methods and properties of service worker.

`self.skipWaiting()` is a function to manually tell service worker to active the waiting service worker.
form the registration object we can emit data to service worker which can be recieve in `message` event

`reg.installing.postMessage({ bar: foo })`;
```javascript
self.addEventListner('message', function(event) {
  console.log(event.data);
});
```
There is an another event that attached on navigator controller to check if the controller is changed
```javascript
navigator.serviceWorker.addEventListner('controllerchange', function() {
  // change happens reload the page
})
```
Now we are going to trigger the update with the use of above information

`git reset --hard`
`git checkout task-update-reload`

```javascript
IndexController.prototype._updateReady = function(worker) {
  var toast = this._toastsView.show("New version available", {
    buttons: ['refresh', 'dismiss']
  });

  toast.answer.then(function(answer) {
    if (answer != 'refresh') return;
    // TODO: tell the service worker to skipWaiting
    worker.postMessage({ action: 'skipWaiting' });
  });
};

// in sw/index.js

self.addEventListner('message', function(event) {
  if (event.data.action === 'skipWaiting') {
    self.skipWaiting();
  }
});

// in indexController.js inside register function

navigator.serviceWorker.addEventListner('controllerchange', function() {
  window.location.reload();
})
```

## Update Data using the IndexedDB
### Introduction to IndexedDB
In general, there are two different types of databases: relational and document (also known as NoSQL or object). Relational databases like SQL Server, MySQL, and Oracle store sets of data in tables. Document databases like MongoDB, CouchDB, and Redis store sets of data as individual objects. IndexedDB is a document database that exists in a sandboxed context. It contains the object stores, you can create multiple databases with whatever names you choose, but generally there is one database per app.
#### Object store
An object store is an individual bucket to store data. You can think of object stores as being similar to tables in traditional relational databases.
#### Index
An Index is a kind of object store for organizing data in another object store (called the reference object store) by an individual property of the data. The index is used to retrieve records in the object store by this property. For example, if you're storing people, you may want to fetch them later by their name, age, or favorite animal.
#### Transaction
A transaction is wrapper around an operation, or group of operations, that ensures database integrity. If one of the actions within a transaction fail, none of them are applied and the database returns to the state it was in before the transaction began. All read or write operations in IndexedDB must be part of a transaction. This allows for atomic read-modify-write operations without worrying about other threads acting on the database at the same time.
####Cursor 
A mechanism for iterating over multiple records in database.

### Create a db
```javascript
//check for support
if (!('indexedDB' in window)) {
  console.log('This browser doesn\'t support IndexedDB');
  return;
}

var dbPromise = idb.open('test-db1', 1);
```
### Create a Object store
```javascript
//check for support
if (!('indexedDB' in window)) {
  console.log('This browser doesn\'t support IndexedDB');
  return;
}

var dbPromise = idb.open('test-db2', 1, function(upgradeDb) {
  console.log('making a new object store');
  if (!upgradeDb.objectStoreNames.contains('firstOS')) {
    upgradeDb.createObjectStore('firstOS');
  }
});
```
`upgradeDb.createObjectStore('people', {keyPath: 'email'});` KeyPath is used to define the primary key

### Defining indexes
`objectStore.createIndex('indexName', 'property', options);`

```javascript
//check for support
if (!('indexedDB' in window)) {
  console.log('This browser doesn\'t support IndexedDB');
  return;
}

var dbPromise = idb.open('test-db4', 1, function(upgradeDb) {
  if (!upgradeDb.objectStoreNames.contains('people')) {
    var peopleOS = upgradeDb.createObjectStore('people', {keyPath: 'email'});
    peopleOS.createIndex('gender', 'gender', {unique: false});
    peopleOS.createIndex('username', 'username', {unique: true});
  }
});
```

`git checkout task-idb-store`

```javascript
function openDatabase() {
  // If the browser doesn't support service worker,
  // we don't care about having a database
  if (!navigator.serviceWorker) {
    return Promise.resolve();
  }

  return idb.open('wittr', 1, function(upgradeDb) {
    var store = upgradeDb.createObjectStore('wittrs', {
      keyPath: 'id'
    });
    store.createIndex('by-date', 'time');
  });
}

IndexController.prototype._onSocketMessage = function(data) {
  var messages = JSON.parse(data);

  this._dbPromise.then(function(db) {
    if (!db) return;

    var tx = db.transaction('wittrs', 'readwrite');
    var store = tx.objectStore('wittrs');
    messages.forEach(function(message) {
      store.put(message);
    });
  });

  this._postsView.addPosts(messages);
};
```
`git checkout task-show-stored`

```javascript
IndexController.prototype._showCachedMessages = function() {
  var indexController = this;

  return this._dbPromise.then(function(db) {
    // if we're already showing posts, eg shift-refresh
    // or the very first load, there's no point fetching
    // posts from IDB
    if (!db || indexController._postsView.showingPosts()) return;

    var index = db.transaction('wittrs')
      .objectStore('wittrs').index('by-date');

    return index.getAll().then(function(messages) {
      indexController._postsView.addPosts(messages.reverse());
    });
  });
};
```
