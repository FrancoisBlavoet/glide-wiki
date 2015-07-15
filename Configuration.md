# Lazy Configuration
Starting in Glide 3.5, you can use the [`GlideModule`][1] interface to lazily configure Glide and register components like [`ModelLoaders`][2] automatically when the first Glide request is made.

## Creating a GlideModule
To use and register a `GlideModule`, first implement the interface with your configuration and components:

```java
package com.mypackage;

public class MyGlideModule implements GlideModule {
    @Override public void applyOptions(Context context, GlideBuilder builder) {
        // Apply options to the builder here.
    }
   
    @Override public void registerComponents(Context context, Glide glide) {
        // register ModelLoaders here.
    }
}
```

Then add your implementation class to your `proguard.cfg` file to allow your module to be instantiated via reflection. The performance overhead is minimal because each module is instantiated a single time when the first request with Glide is made.

```proguard
-keepnames class com.mypackage.MyGlideModule
# or more generally:
#-keep public class * implements com.bumptech.glide.module.GlideModule
```

Finally add a meta-data tag to your `AndroidManifest.xml` file so Glide can find your module:

```xml
<meta-data
    android:name="com.mypackage.MyGlideModule"
    android:value="GlideModule" />
```

You can implement as many `GlideModule`s as you like, although each one must be added to `proguard.cfg` and each one must have its own meta-data tag in the manifest.

## Library projects

Library projects may define one or more `GlideModule`s. If a library project adds a module to its manifest, applications built using Gradle (or any system with a manifest merger) that depend on the library project will automatically pick up the library's module. If no manifest merger is available, the library module will have to be manually listed in the application's manifest.

## Conflicting GlideModules

Although Glide allows multiple `GlideModule` to be registered per application, it doesn't call them in any particular order. As a result, users ares responsible for avoiding conflicts between registrations if their application defines more than one or depends on multiple libraries that define `GlideModule`s.

If a conflict is unavoidable, applications should default to defining their own module that manually resolves the conflict and provides all dependencies required by the library modules. Gradle users, or any other user using a manifest merger, can exclude conflicting modules by adding a tag to their `AndroidManifest.xml` to exclude the corresponding meta-data tags:

```xml
<meta-data android:name=”com.mypackage.MyGlideModule” tools:node=”remove” />
```

# Global Configuration
Glide allows you to configure a number of different global options that apply to all requests. To do so you use the [`GlideBuilder`][3] provided to you in `GlideModule#applyOptions`. The code examples in this section will refer to it as `builder`.

## Disk Cache
You can use the `GlideBuilder`'s [`setDiskCache()`][4] method to set the location and/or maximum size of the disk cache. You can also disable the cache entirely using [`DiskCacheAdapter`][5] or replace it with your own implementation of the [`DiskCache`][6] interface. Disk caches are built on background threads to avoid strict mode violations using the [`DiskCache.Factory`][7] interface.

By default Glide uses the [`InternalCacheDiskCacheFactory`][9] class to build disk caches. The internal cache factory places the disk cache in your application's internal cache directory and sets a maximum size of 250MB. Using the cache directory rather than the external SD card means no other applications will be able to access the images you download. See Android's [Storage Options][10] doc for more details.

#### Size
You can set the size of the disk cache using the [`InternalCacheDiskCacheFactory`][9]:

```java
builder.setDiskCache(
  new InternalCacheDiskCacheFactory(context, yourSizeInBytes));
```

#### Location
Setting the location of the disk cache is also possible. 

You can use the built in [`InternalCacheDiskCacheFactory`][9] to place your cache in your applications private internal cache directory:

```java
builder.setDiskCache(
  new InternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));
```

You can also use the built in [`ExternalCacheDiskCacheFactory`][12] to place your cache in your applications public cache directory on the sd card:

```java
builder.setDiskCache(
  new ExternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));
```

If you'd like to use some other custom location, you can leverage the [`DiskLruCacheFactory`][13] class's constructors:

```java
// If you can figure out the folder without I/O:
// Calling Context and Environment class methods usually do I/O.
builder.setDiskCache(
  new DiskLruCacheFactory(getMyCacheLocationWithoutIO(), yourSizeInBytes));

// In case you want to specify a cache folder ("glide"):
builder.setDiskCache(
  new DiskLruCacheFactory(getMyCacheLocationWithoutIO(), "glide", yourSizeInBytes));

// In case you need to query the file system while determining the folder:
builder.setDiskCache(new DiskLruCacheFactory(new CacheDirectoryGetter() {
    @Override public File getCacheDirectory() {
        return getMyCacheLocationBlockingIO();
    }
}), yourSizeInBytes);
```

**Note**: *Please remember, hardcoding any value is never a good idea, especially directory paths since [Android 4.2, which enables multiple users on a single device](http://developer.android.com/about/versions/android-4.2.html#MultipleUsers).*

If you want full control of your cache creation you can also implement the [`DiskCache.Factory`][7] interface yourself, and use the [`DiskLruCacheWrapper`][11] to create a new cache in your desired location:

```java
builder.setDiskCache(new DiskCache.Factory() {
    @Override public DiskCache build() {
        File cacheLocation = getMyCacheLocationBlockingIO();
        cacheLocation.mkdirs();
        return DiskLruCacheWrapper.get(cacheLocation, yourSizeInBytes);
    }
});
```

## Memory caches and pools
The `GlideBuilder` class allows you to set the size and implementation of Glide's [`MemoryCache`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/MemoryCache.html) and [`BitmapPool`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html). 

### Size
Default sizes are determined by the [`MemorySizeCalculator`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/MemorySizeCalculator.html) class. The MemorySizeCalculator class takes into account the screen size available memory of a given device to come up with reasonable default sizes. You can construct your own instance if you want to adjust Glide's defaults:

```java
MemorySizeCalculator calculator = new MemorySizeCalculator(context);
int defaultMemoryCacheSize = calculator.getMemoryCacheSize();
int defaultBitmapPoolSize = calculator.getBitmapPoolSize();
```

If you want to dynamically adjust Glide's memory footprint during certain phases of your application, you can do so by picking a [`MemoryCategory`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/MemoryCategory.html) and passing it to Glide using [`setMemoryCategory()`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/Glide.html#setMemoryCategory(com.bumptech.glide.MemoryCategory)):

```java
Glide.get(context).setMemoryCategory(MemoryCategory.HIGH);
```

### Memory Cache
Glide's memory cache is used to hold resources in memory so that they are instantly available without having to perform I/O. 

You can use `GlideBuilder`'s [`setMemoryCache()`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html#setMemoryCache(com.bumptech.glide.load.engine.cache.MemoryCache)) method to set the size and/or implementation you wish to use for your memory cache. The [`LruResourceCache`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/cache/LruResourceCache.html) class is Glide's default implementation. You can set a custom maximum in memory byte size by passing in the size you want to the `LruResourceCache` constructor:

```java
builder.setMemoryCache(new LruResourceCache(yourSizeInBytes));
```

### Bitmap Pool
Glide's bitmap pool is used to allow `Bitmap`s of a variety of different sizes to be re-used which can substantially reduce jank inducing garbage collections caused by pixel array allocations while images are being decoded.

You can use `GlideBuilder`'s [`setBitmapPool()`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/GlideBuilder.html#setBitmapPool(com.bumptech.glide.load.engine.bitmap_recycle.BitmapPool)) method to set the size and/or implementation of the bitmap pool. The [`LruBitmapPool`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/bitmap_recycle/LruBitmapPool.html) class is Glide's default implementation. The `LruBitmapPool` class uses an LRU algorithm to retain the most recently used sizes of `Bitmap`s. You can set a custom maximum in memory byte size by passing the size you want to the `LruBitmapPool` constructor:

```java
builder.setBitmapPool(new LruBitmapPool(sizeInBytes));
```

## Bitmap Format
The `GlideBuilder` class also allows you to set a global default for the preferred [Bitmap configuration](http://developer.android.com/reference/android/graphics/Bitmap.Config.html) for your application.

By default Glide prefers [`RGB_565`](http://developer.android.com/reference/android/graphics/Bitmap.Config.html#RGB_565) because it requires only two bytes per pixel (16 bit) and therefore has half the memory footprint of the higher quality and system default [`ARGB_8888`](http://developer.android.com/reference/android/graphics/Bitmap.Config.html#ARGB_8888). `RGB_565` however can have issues with banding in certain images and also does not support transparency.

If banding is a problem in your application and/or you want the highest possible image quality, you can use `GlideBuilder`'s [`setDecodeFormat`](https://github.com/bumptech/glide/blob/3.0/library/src/main/java/com/bumptech/glide/GlideBuilder.java#L157) method to set [`DecodeFormat.ALWAYS_ARGB_8888`](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/DecodeFormat.html#ALWAYS_ARGB_8888) as Glide's preferred Bitmap configuration:

```java
builder.setDecodeFormat(DecodeFormat.ALWAYS_ARGB_8888);
```

[1]: https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/module/GlideModule.java
[2]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/model/ModelLoader.html
[3]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/GlideBuilder.html
[4]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/GlideBuilder.html#setDiskCache(com.bumptech.glide.load.engine.cache.DiskCache.Factory)
[5]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/cache/DiskCacheAdapter.html
[6]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/cache/DiskCache.html
[7]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/cache/DiskCache.Factory.html
[8]: http://developer.android.com/reference/android/content/Context.html#getCacheDir()
[9]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/cache/InternalCacheDiskCacheFactory.html
[10]: http://developer.android.com/guide/topics/data/data-storage.html#filesInternal
[11]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/cache/DiskLruCacheWrapper.html
[12]: http://bumptech.github.io/glide/javadocs/360/com/bumptech/glide/load/engine/cache/ExternalCacheDiskCacheFactory.html
[13]: http://bumptech.github.io/glide/javadocs/360/com/bumptech/glide/load/engine/cache/DiskLruCacheFactory.html
