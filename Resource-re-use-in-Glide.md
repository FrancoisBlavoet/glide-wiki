# Why
Glide re-uses resources to avoid unnecessary allocations. Dalvik (before Lollipop) has two basic modes for garbage collection, ``GC_CONCURRENT`` and ``GC_FOR_ALLOC``. ``GC_CONCURRENT`` blocks the main thread twice for about 5ms for each collection. Since each operation is less than a single frame (16ms), ``GC_CONCURRENT`` tends not to cause your application to drop frames. In contrast, ``GC_FOR_ALLOC`` is a stop the world collection that can block the main thread for 125+ms. GC_FOR_ALLOC virtually always causes your application to drop multiple frames, resulting in visible stuttering, particularly while scrolling.

Unfortunately Dalvik seems to handle even modest allocations (a 16kb buffer for example) poorly. Repeated moderate allocations, or even a single large allocation (say for a Bitmap), will cause GC_FOR_ALLOC. Therefore, the more you allocate, the more stop the world garbage collections you incur, and the more frames your application drops.

By re-using moderate to large resources, Glide helps keep your app jank free by avoiding a substantial number of stop the world garbage collections. 

# How
Glide takes a permissive approach to re-use resource, which means that although Glide will opportunistically re-use resources when it believes it is safe to do so, Glide does not require users to recycle resources after each request.

## Signals
Glide uses two simple signals to identify resources that can be re-used:

1. [``Glide.clear()``][1]
    
    Calling ``clear()`` on a [View][2] or a [Target][3] indicates both that Glide should cancel any in progress loads and that it is safe for Glide to return any resources (Bitmaps, byte arrays etc) displayed in the Target to a resource pool. Users may manually call ``clear()`` at any time, however they are not typically required to do so due to #2:

2.  View or Target re-use

    Whenever a user starts a new load into an existing View or Target, Glide first calls [``clear()``][1] on the View or Target to cancel any previous load and/or re-use any displayed resources. As a result, Glide will automatically pool resources and manage loads for users who re-use Views in ListView or RecyclerView. 

## Reference counting
To avoid unnecessary work, if two requests are made that map to the same resource, Glide will the single resource to both callers. As a result, a single signal that a resource is no longer used is not sufficient. To avoid recycling resources that are used by one caller, but not the other, Glide uses reference counting to track resources.

Providing a given resource to a Target or a View increments the reference count for the resource by one. Clearing a given Target or View decrements the reference count by one. When the reference count drops to zero, Glide will recycle the resource and return its contents to any available pools.

## Pooling
Glide's [Resource][4] API contains a [``recycle()``][5] method that is called when Glide believes a resource has no more consumers.  

Glide currently provides a [BitmapPool][6] interface that allows Resources to obtain ``Bitmap`` and re-use Bitmap objects. Glide's BitmapPool can be obtained from any Context using the Glide singleton:

```java
Glide.get(context).getBitmapPool();
```

[ResourceDecoders][7] are free to return any implementation of the [Resource][4] API they wish, so users can customize or provide additional pooling for novel types by implementing their own Resources and ResourceDecoders. 

Similarly users who want more control over Bitmap pooling are free to implement their own [BitmapPool][6], which they can then provide to Glide using a [GlideModule][8].

# Common mistakes 
Unfortunately pooling makes it difficult to assert that a user isn't misusing a resource or a Bitmap. However, there are two primary indicators that something might be going wrong with Bitmap pooling in Glide.

## Symptoms

1. ``Cannot draw a recycled Bitmap``

    Glide's BitmapPool has a fixed size. When Bitmaps are evicted from the pool without being re-used, Glide will call [``recycle()``][9]. If an application inadvertently continues to hold on to the Bitmap even after indicating to Glide that it is safe to recycle it, the application may then attempt to draw the Bitmap, resulting in a crash.

2. Views flicker between images or the same image shows up in multiple views

    If a Bitmap is returned to the BitmapPool multiple times, or is returned to the pool but still held on to by a View, another image may be decoded into the Bitmap. If this happens, the contents of the Bitmap are replaced with the new image. Views may still attempt to draw the Bitmap during this process, which will result either in artifacts or in the original View showing a new image.

## Causes
The two most common causes of these issues are:

1. Attempting to load two different resources into the same Target.
   
    There is no safe way to load multiple resources into a single Target in Glide. Users can use the [``thumbnail()``][10] API to load a series of resources into a Target, but it is only safe to reference each resource until the next call to [``onResourceReady()``][11]. 

    Users who want to load multiple resources into the same View can do so by using two separate Targets. To make sure that the loads don't cancel each other, users either need to avoid [ViewTarget][12] subclasses, or use a custom ViewTarget subclass and override [``setRequest()``][13] and [``getRequest()``][14] so that they do not use the View's tag to store the Request.

2. Loading a resource into a Target, clearing or reusing the Target, and continuing to reference the resource.

    The easiest way to avoid this error is to make sure that all references to a resource are nulled out when [``onLoadCleared()``][15] is called. It is generally safe to load a Bitmap and then de-reference the Target. It is not safe to load a Bitmap, clear the Target, and continue to reference the Bitmap. 

[1]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/Glide.html#clear(com.bumptech.glide.request.target.Target)
[2]: http://developer.android.com/reference/android/view/View.html
[3]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/request/target/Target.html
[4]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/Resource.html
[5]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/Resource.html#recycle()
[6]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html
[7]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/ResourceDecoder.html
[8]: https://github.com/bumptech/glide/wiki/Configuration#including-a-glidemodule
[9]: http://developer.android.com/reference/android/graphics/Bitmap.html#recycle()
[10]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/DrawableRequestBuilder.html#thumbnail(com.bumptech.glide.DrawableRequestBuilder)
[11]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/request/target/Target.html#onResourceReady(R,%20com.bumptech.glide.request.animation.GlideAnimation)
[12]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/request/target/ViewTarget.html
[13]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/request/target/ViewTarget.html#setRequest(com.bumptech.glide.request.Request)
[14]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/request/target/ViewTarget.html#getRequest()
[15]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/target/Target.html#onLoadCleared(android.graphics.drawable.Drawable)