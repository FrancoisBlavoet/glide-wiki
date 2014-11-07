Glide allows you to configure a number of different global options that apply to all requests. To do so you use the [GlideBuilder](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html) class. 

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
Alternatively you can also use Glide's [``isSetup()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/Glide.html#isSetup()) method to optionally setup Glide in an Activity.

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
You can use the GlideBuilder's [``setDiskCache()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html#setDiskCache(com.bumptech.glide.load.engine.cache.DiskCache)) method to set the location and/or maximum size of the disk cache. You can also disable the cache entirely using [DiskCacheAdapter](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/DiskCacheAdapter.html) or replace it with your own implementation of the [DiskCache](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/engine/cache/DiskCache.html) interface. 

#### Size
You can set the size of the disk cache using the [DiskLruCacheWrapper](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html) singleton. To do so, you can use Glide's [``getPhotoCacheDir()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/Glide.html#getPhotoCacheDir(android.content.Context)) to retain the default location and then simply pass in the size you want in bytes:

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

## In memory caches and pools
The GlideBuilder class allows you to set the size and implementation of Glide's [``MemoryCache``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/MemoryCache.html) and [``BitmapPool``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html). 

### Size
Default sizes are determined by the [``MemorySizeCalculator``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html) class. The MemorySizeCalculator class takes into account the screen size available memory of a given device to come up with reasonable default sizes. You can construct your own instance if you want to adjust Glide's defaults:

```java
MemorySizeCalculator calculator = new MemorySizeCalculator(context);
int defaultMemoryCacheSize = calculator.getMemoryCacheSize();
int defaultBitmapPoolSize = calculator.getBitmapPoolSize();
```

If you want to dynamically adjust Glide's memory footprint during certain phases of your application, you can do so by picking a [``MemoryCategory``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/MemoryCategory.html) and passing it to Glide using [``setMemoryCategory()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/Glide.html#setMemoryCategory(com.bumptech.glide.MemoryCategory)):

```java
Glide.get(context).setMemoryCategory(MemoryCategory.HIGH);
```

### Memory Cache
Glide's memory cache is used to hold resources in memory so that they are instantly available without having to perform I/O. 

You can use GlideBuilder's [``setMemoryCache()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html#setMemoryCache(com.bumptech.glide.load.engine.cache.MemoryCache)) method to set the size and/or implementation you wish to use for your memory cache. The [``LruResourceCache``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/LruResourceCache.html) class is Glide's default implementation. You can set a custom maximum in memory byte size by passing in the size you want to the LruResourceCache constructor:

```java
new GlideBuilder(context)
   .setMemoryCache(new LruResourceCache(yourSizeInBytes));
```

### Bitmap Pool
Glide's bitmap pool is used to allow Bitmaps of a variety of different sizes to be re-used which can substantially reduce jank inducing garbage collections caused by Bitmap allocations while images are being decoded.

You can use GlideBuilder's [``setBitmapPool()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html#setBitmapPool(com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool)) method to set the size and/or implementation of the bitmap pool. The [``LruBitmapPool``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html) class is Glide's default implementation. The LruBitmapPool class uses an LRU algorithm to retain the most recently used sizes of Bitmaps. You can set a custom maximum in memory byte size by passing the size you want to the LruBitmapPool constructor:

```java
new GlideBuilder(context)
    .setBitmapPool(new LruBitmapPool(sizeInBytes));
```

## Bitmap Format
The GlideBuilder class also allows you to set a global default for the preferred [Bitmap configuration](http://developer.android.com/reference/android/graphics/Bitmap.Config.html) for your application. 

By default Glide prefers [``RGB_565``](http://developer.android.com/reference/android/graphics/Bitmap.Config.html) because it requires only two bytes per pixel and therefore has half the memory footprint of the higher quality and system default [``ARGB_8888``](http://developer.android.com/reference/android/graphics/Bitmap.Config.html). RGB_565 however can have issues with banding in certain images and also does not support transparency.

If banding is a problem in your application and/or you want the highest possible image quality, you can use GlideBuilder's [``setDecodeFormat``](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/GlideBuilder.java#L133) method to set [``DecodeFormat.ALWAYS_ARGB_8888``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/DecodeFormat.html#ALWAYS_ARGB_8888) as Glide's preferred Bitmap configuration:

```java
new GlideBuilder(context)
    .setDecodeFormat(DecodeFormat.ALWAYS_ARGB_8888);
```
