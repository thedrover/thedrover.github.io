---
layout: post
title: Async Api Integration Testing for Mobile Clients - Android edition 
---

http://tools.android.com/tech-docs/unit-testing-support
Build variants and version is key: http://learningtofit.blogspot.com.au/2015/02/android-testing-gradle-android-studio.html?view=classic

Document api asyn api test methods for ios and android....two posts perhaps?

Provide code download example...

Option 1: Test synchronous part for integration test. Asynchronous production code invokes. May lead to distortion of internal inerfaces.

Option 2. Async testing...here's a pattern. Check for others.
This example demonsrayetes usage witrh Robolectric, but that is optional.Android.
Consider other Android test frameworks.

Requires an event/callback class. Test extension implements callable.

What about running in async task?? Can wait for the result there. Done??!!!
Think it works. Try examples! But implies API is a direct reflection of AsyncTask...Need to call AsynTask.get() and block on that. That API i convenient for tests because it blocks...but you can't call it from the UI becasue it will block the UI thread (Asunc tasks must be created in the UI thread). See http://stackoverflow.com/questions/9273989/how-do-i-retrieve-the-data-from-asynctasks-doinbackground/14129332#14129332 for callbacks...reference this approach in demo.

 @Override
  protected void doSetup() {
    super.doSetup();
    // Use real HTTP layer with Robolectric
    // Robolectric.getFakeHttpLayer().interceptHttpRequests(false);

    mEventMgr = (EventManager) mServiceBroker.getService(EventManager.class);
    mHttpClient = new HttpRestClient();

    if (shouldUseMockServices()) {
      mMockContext.checking(new Expectations() {
        {

          oneOf(mMockPrefs).getString("SDK_VERSION", "unknown");
          will(returnValue("test"));

        }
      });
    }

    mHttpClient.configure();

    // Future test setup
    mFutureEventListener = new TestEventListener();

  }



    // Set up a future that will be interrogated until the asynchronous event is
    // received or the test times out.
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    mFuture = executorService.submit(mFutureEventListener);
    mEventMgr.addEventListener(mFutureEventListener, request.getEventService());
    mHttpClient.makeGetRequest(request);
    assertFutureSucceded();

    protected void assertFutureSucceded() throws InterruptedException, ExecutionException {
      Assert.assertTrue("Future did not return a successful connection result: " + mFutureEventListener, mFuture.get());
    }



/**
 * Event listener that can be set up as a future that will be interrogated until
 * the asynchronous event is received or the test times out.
 * 
 */
public class TestEventListener implements Callable<Boolean>, EventListener {

  /**
   * 
   */
  private static final int LOOP_CHECK_COUNT = 100;

  /**
   * 
   */
  private static final int WAIT_INTERVAL = 100;

  private boolean eventReceived = false;

  // Set to values that will never be used for service or action ids
  private int eventService = Integer.MIN_VALUE;
  private int eventAction = Integer.MIN_VALUE;
  // Set to object so null payload correct can be confirmed.
  private Object eventPayload = new Object();

  @Override
  public void onEvent(int service, int action, Object data) {
    eventReceived = true;
    StringBuilder msg = new StringBuilder("Received test event:").append(service).append(", ").append(action)
        .append(", ").append(data);
    LogIncoming.i(TestEventListener.class.getSimpleName(), msg.toString());

    eventService = service;
    eventAction = action;
    eventPayload = data;
  }

  @Override
  public Boolean call() throws Exception {
    // Approximate timeout is fine
    int waitInterval = WAIT_INTERVAL;
    int count = LOOP_CHECK_COUNT;
    int i = 0;
    while (!eventReceived) {
      synchronized (this) {

        wait(waitInterval);
        i++;
        if (i > count) {
          return false;
        }

      }
    }
    return eventReceived;
  }

  public int getEventService() {
    return eventService;
  }

  public int getEventAction() {
    return eventAction;
  }

  public Object getEventPayload() {
    return eventPayload;
  }

  @Override
  public String toString() {
    int waitDurationMs = WAIT_INTERVAL * LOOP_CHECK_COUNT;
    return "TestEventListener - Async event wait time = " + waitDurationMs;
  }

}
