To make loading and interacting with media on background threads easier, Glide provides two APIs in addition to the normal ``Glide.with(fragment).load(url).into(view)`` API:

* [``downloadOnly(int, int)``][2]
* [``into(int, int)``][7]

### downloadOnly
Glide's ``downloadOnly()`` API allows you to download the bytes of an image into the disk cache so that it will be available to be retrieved later. You can use ``downloadOnly()`` [asynchronously][1] on the ui thread or [synchronously][2] on a background thread. Note that the arguments are slightly different, the asynchronous api takes a [``Target``][3] and the synchronous api takes an integer width and height.

To download images on a background thread, you must use the [synchronous][2] version:

```java
FutureTarget<File> future = Glide.with(applicationContext)
    .load(yourUrl)
    .downloadOnly(500, 500);
File cacheFile = future.get();
```

Once the future returns, the bytes of the image are available in the cache. Typically the ``downloadOnly()`` API is only used to make sure the bytes are available on disk. Although you are given access to the underlying cache File, you usually don’t want to interact with it.

Instead when you later want to retrieve your image, you can do so using a normal call with one exception:

```java
Glide.with(yourFragment)
    .load(yourUrl)
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    .into(yourView);
```

By passing in [``DiskCacheStrategy.ALL``][4] or [``DiskCacheStrategy.SOURCE``][5], you make sure Glide will use the bytes you cached using ``downloadOnly()``. 

### into
If you actually want to interact with a decoded image on a background thread, instead of using ``downloadOnly`` you can use the version of [``into()``][7] that returns a [``FutureTarget``][6]. For example, to get the center cropped bytes of a 500px by 500px image:

```java
Bitmap myBitmap = Glide.with(applicationContext)
    .load(yourUrl)
    .asBitmap()
    .centerCrop()
    .into(500, 500)
    .get()
```

Although [``into(int, int)``][7] works well on background threads, note that you must not use it on the main thread. Even if the synchronous version of into did not throw an exception when used on the main thread, calling get() would block the main thread, reducing the performance and responsiveness of your app.

[1]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableTypeRequest.html#downloadOnly(Y) 
[2]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableTypeRequest.html#downloadOnly(int,%20int)
[3]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/target/Target.html
[4]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/DiskCacheStrategy.html#ALL
[5]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/DiskCacheStrategy.html#SOURCE
[6]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/FutureTarget.html
[7]:
http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GenericRequestBuilder.html#into(int,%20int)
