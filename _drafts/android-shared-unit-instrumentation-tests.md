---
layout: post
title: Sharing test code between unit tests and instrumentation tests
summary: x
categories: android unit testing instrumentation testing robolectric junit
---

This post....


Give a general overiew of unit versus instrumentation tests.
try to keep it simpe. so no esperessos. activity tests etc.

Fr example, there might bea fike of canned test data that can be re-used in the unit and intrumentaion tests. The sample code demonstrates how to load this data froma common source in both cases.

In a similar vein, the tests may depenr on an application assset that resides in the assets directory.

The test classes inherit from a ahred tests class that contains an example test. This test will be executed by both test classes so it can be used to run quickly and often from the IDE or in the context of an instrumentatio. test suite for the final APK. This won't always make sense, but the zbiloty to share test utilties classses and methods will always come in handy. 




The Android JUnit 4 test runner (link) is used to provide compatibilty with modern unit testing approaches.
A UiThreadTestRuke is used to keep things simple in the test example. This can be exchanged for an ActivityTestRule ifbthe instrumentation tests are exercising an Activity in the application.


The unit testing aspect rewuires something to mock the system. Examples with Robolectric and the Android !ick, which have been available since...are both provided.

as