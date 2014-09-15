Exceptions during loads in Glide are not logged by default, but Glide gives you two ways to view and/or respond to these exceptions.

## Debugging
To simply view exceptions when they occur, you can turn on debug logging for Glide's [GenericRequest](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/GenericRequest.html) class that handles responses for all media loads. To do so, in the command line, run:

```adb shell setprop log.tag.GenericRequest DEBUG``` 

To include verbose request timing logs you can also pass in ``VERBOSE`` instead of ``DEBUG``. To disable logging, run: 

```adb shell setprop log.tag.GenericRequest ERROR```

## RequestListener
Although enabling debug logging is simple, it's only possible if you have access to the device. To integrate Glide with a pre-existing or more sophisticated error logging system, you can use the [RequestListener](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/RequestListener.html) class. [``onException()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/RequestListener.html#onException(java.lang.Exception, T, com.bumptech.glide.request.target.Target, boolean)) will be called when the request fails and will provide the Exception that caused the failure, or null if a decoder was unable to decode anything useful from the data it received. You can pass your listener in to each request using the [``listener()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#listener(com.bumptech.glide.request.RequestListener)) api.

Be sure to return ``false`` from ``onException()`` to avoid overriding Glide's default error handling behavior. 


