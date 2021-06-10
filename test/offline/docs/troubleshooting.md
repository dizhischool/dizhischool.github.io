# `offline-plugin` isn't working

___________________________________

#### The ServiceWorker is not being updated

Make sure the ServiceWorker (`sw.js`) is not being cached by the server (`Cache-Control: no-store`) and that it has a deterministic name (no hashing).
  
#### AppCache is being used instead of ServiceWorker

Make sure your webapp is served with HTTPS (or localhost for development).

#### I cannot see any changes in my browser

Make sure the browser has cached the latest resources, follow the steps outlined in the [updates](updates.md) doc.

#### ServiceWorker isn't being updated automatically

Automatic updates isn't the default behavior, but can be achieved using the [autoUpdate option](options.md#autoupdate-true--number)

#### All routes/resources aren't being cached

`offline-plugin` automatically caches all resources generated by webpack. Any other resource needs to be explicitly cached as [external](options.md#externals-arraystring).

#### Some pages / resources are not being served at all

The ServiceWorker can only serve resources within its [scope](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register). Serve your sw file from the root of your public folder, or supply the correct [`ServiceWorker.scope` option](options.md#serviceworker-object--null--false).

#### The ServiceWorker gets overridden by another app on the same domain

Specify a unique [`ServiceWorker.cacheName`](options.md#serviceworker-object--null--false) to avoid it being overridden.

#### `offline-plugin` isn't working in dev mode

`webpack-dev-server` compatibility is unknown for the moment, so if you are using hot reloading, ServiceWorker might not work as it's supposed to.

#### cookies are not included in the ServiceWorker pre-fetch request

Make sure you are using the correct [prefetchRequest.credentials](options.md#serviceworker-object--null--false) to allow for cookies.

#### ServiceWorker doesn't work for subpages

`offline-plugin` is probably not configured to handle these routes. See the [`appShell`](app-shell.md) option.

#### ServiceWorker is wrongly caching api requests

There are multiple ways to fix this depending on your setup. One way is to set [cacheMaps.requestTypes](cache-maps.md) to `['navigate']`, caching only those requests.

#### AppCache events are not working properly

`AppCache.events` are known to be a bit buggy (see [updates](updates.md)). Try to avoid using them if possible.

#### Resources served from a CDN are not being cached

`offline-plugin` can cache resources served from a CDN, given the correct configuration. Make sure the resources are served with the correct headers.

#### DOMException: Failed to register a ServiceWorker: The script has an unsupported MIME type ('text/html') Error

Make sure your webserver is serving your assets (including `sw.js`) with the correct mime type. Also, make sure you are using relative / absolute paths correctly (see the [`FAQ`](FAQ.md) regarding this). Also make sure that ServiceWorker and/or AppCache files are generated into the correct folder. You may use `ServiceWorker.output` and `AppCache.output` to change the output path. See [options](options.md) for details.

#### Fetching the ServiceWorker results in an error

Some errors, like being unable to fetch the ServiceWorker file, originates from the browser, network or server and are out of control of `offline-plugin`. Here is a list of common errors and their causes.

> TypeError: Failed to update a ServiceWorker: A bad HTTP response code (500) was received when fetching the script.

This is a server error. Whatever happens there - it shouldn't. Could be that the server is temporarily down (e.g. changing a deployment on Zeit).

> TypeError: Failed to fetch

This is the most generic error you will receive. It typically happens when there's no network connection - it simply can't fetch the file.

> AbortError: Failed to update a ServiceWorker: The request to fetch the script was interrupted.

This is a network error - something interrupted the connection. It might be due to the network connection being changed or temporarily lost (it happens all the time in the wild).

> TypeError: Script URL https://my-page.com/sw.js fetch resulted in error: An SSL error has occurred and a secure connection to the server cannot be made.

This is a proxy issue. Ie. someone sitting at an airport or on a train and that network injects fake HTTPS certificates (man in the middle).

#### How to handle out-of-control errors

As the above errors are out of the control of `offline-plugin`, there is nothing we can do to prevent them. What you may do if you receive any of them is to catch them in your code and try to reschedule the update. Also make sure that your server works correctly and remove any potential bugs / error situation causing HTTP 500 Internal Server Error.