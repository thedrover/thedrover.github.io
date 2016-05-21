---
layout: post
title: Async Api Integration Testing for Mobile Clients - Android Recipe
---

This post provides a simple recipe to help write integration tests to
validate the interactions between an Android application or SDK and a
server API. This is a useful to detect any problems where the server
API is not implemented exactly as documented, or there is some
misinterpretation of the API spec in the client development. Three
examples of common HTTP request methods are presented:
[`HTTPUrlConnection`](http://developer.android.com/reference/java/net/HttpURLConnection.html),
[OK HTTP](http://square.github.io/okhttp/), and the deprecated [HTTP
client](http://enoent.fr/blog/2015/10/01/use-apache-http-client-on-android-sdk-23/). The
recipe is readily extended to other approaches, or even non-HTTP
interactions. The recipe demonstrates how to execute the tests using
either the Android test runner or robolectric.


Save cost
by detecting before QA. provides a test be for the client dev that is
quicker to work with. 

what about androidJUnit4?

also this https://developers.google.com/api-client-library/java/google-http-java-client/unit-testing

consistency of approach...e.g remove legacy HTTPClient while verisfying 

note about branching and versioning...to validate versions...
...separate post.

|                  | Roboletric | Instrumentation test  |
|                  | ---------- |:-------------:        |
|HttpURLConnection | YES        | YES                   |
|OK Http           | YES        | YES                   | 
| HttpClient       | YES        | YES                   | 

## General Approach


Requires an event/callback class. Test extension implements callable.

simple use of the functionality provided by the `java.util.concurrent` package.

implements [Callable](http://developer.android.com/reference/java/util/concurrent/Callable.html)

```java

HttpResult resultHandler = new HttpResult();
Callable<HttpResult> result = new HttpResultWatcherTask(resultHandler);

// Set up a future that will be interrogated until 
// the asynchronous event is received or the test times out.
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Boolean> future = executor.submit(result);

// MUT
mHttpRequestor.makeRequestOnThread("http://httpbin.org/headers", resultHandler);

HttpResult r = future.get(TestUtil.TIMEOUT_MILLIS, TimeUnit.MILLISECONDS);
```

single worker thread.

result in a [`Future`](http://developer.android.com/reference/java/util/concurrent/Future.html)

```java


```

The details can be changed to fit with the problem. The basic
requirement is that there is some way for the test case to check the
status of the HTTP request and response.

In order to avoid copying and pasting test setup in a number of test
classes I've used parameterised test runs to iterate over all
approaches in a sinple test class.

 
simple pattern
make HTTP GET request
if successful, decode json response from httpbin
if a failure, reecord the response code and message.

the result holdre is passed into the API method being tezted.

In practice the callable may be notified of yje completion of the API response in some way. e.g. some evebt, but the testing principle is teh same

## HttpURLConnection

## OK HTTP

## HttpClient




http://tools.android.com/tech-docs/unit-testing-support
Build variants and version is key: http://learningtofit.blogspot.com.au/2015/02/android-testing-gradle-android-studio.html?view=classic


Provide code download example...(github URL)
