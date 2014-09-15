Glide allows you to configure a number of different global options that apply to all requests. To do so you use the [GlideBuilder](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/GlideBuilder.html) class. 

#### In your Application
The simplest way to setup Glide is to do so in your [Application](http://developer.android.com/reference/android/app/Application.html) object in [``onCreate()``](http://developer.android.com/reference/android/app/Application.html#onCreate()):

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        Glide.setup(new GlideBuilder(this)
            .setDiskCache(...)
            .setMemoryCache(...)
        );
    }
}
```

#### In your Activity
Alternatively you can also use Glide's [``isSetup()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/Glide.html#isSetup()) method to optionally setup Glide in an Activity.

```java
public MyActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        if (!Glide.isSetup()) {
            Glide.setup(new GlideBuilder(this)
                .setDiskCache(...)
                .setMemoryCache(...)
            );
        }
    }
}
```
#### With background loads
Keep in mind that if you start loads in the background you should surround your ``Glide.isSetup()`` and ``Glide.setup()`` with a ``synchronized`` block:

```java
synchronized(Glide.class) {
    if (!Glide.isSetup()) {
        Glide.setup(...);
    }
}
```

The synchronized block is only necessary if you start loads in the background, and typically only when you're doing so in an Activity. Calling ``Glide.setup()`` after the Glide singleton has been created either via a ``setup()`` call or via a ``Glide.get()`` or ``Glide.with()`` call will throw an ``IllegalArgumentException``.

## Disk Cache
You can use the GlideBuilder's [``setDiskCache()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/GlideBuilder.html#setDiskCache(com.bumptech.glide.load.engine.cache.DiskCache)) method to set the location and/or maximum size of the disk cache. You can also disable the cache entirely using [DiskCacheAdapter](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/engine/cache/DiskCacheAdapter.html) or replace it with your own implementation of the [DiskCache](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/engine/cache/DiskCache.html) interface. 

#### Size
You can set the size of the disk cache using the [DiskLruCacheWrapper](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html) singleton. To do so, you can use Glide's [``getPhotoCacheDir()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/Glide.html#getPhotoCacheDir(android.content.Context)) to retain the default location and then simply pass in the size you want in bytes:

```java
new GlideBuilder(context)
    .setDiskCache(DiskLruCacheWrapper.get(Glide.getPhotoCacheDir(context), yourSizeInBytes));
```

#### Location
You can also use the DiskLruCacheWrapper to change the default location in the same manner as above:

```java
new GlideBuilder(context)
    .setDiskCache(DiskLruCacheWrapper.get(yourCacheLocation, yourSizeInBytes));
```

By default Glide places the disk cache in your application's cache directory and sets a maximum size of 250mb. Using the cache directory rather than the external sd card means no other applications will be able to access the images you download. See Android's [Storage Options](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) doc for more details.