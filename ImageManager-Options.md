#### ImageManager

The [ImageManager](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/resize/ImageManager.java) is the heart of Glide. It runs and cancels individual image load jobs, manages the memory and disk caches, and coordinates bitmap reuse. 

Although it is created with reasonable defaults, it is also possible to change many of the options. If you would like to do so and use the [high level interface](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/Glide.java) for Glide, you will need to do the following in `onCreate()` of your activity[s].

    public void onCreate(Bundle savedInstanceState) {
        final Glide glide = Glide.get();
        if (!glide.isImageManagerSet()) {
            glide.setImageManagerBuilder(new ImageManager.Builder(this)
                .setXyzOption()
                ...
                .build()
            );
        }
    }

This ensures that Glide will use the options you set without creating a new ImageManager every time your activity is created.

#### Defaults

Without any options, Glide will set a number of defaults that are meant to work for most users:

1. An LRU memory cache that is 1/10th the size of the memory available to the application. This is conservative, but decreases the risk of OutOfMemory errors on older devices
2. A 30mb LRU disk cache in the external cache directory for the application (defaults to internal if external is not available). 
3. A single background thread to load images from the disk cache and manage internal state.
4. A FixedThreadPool executor service with as many threads as the device as processors to use to resize and cache images.
5. On devices with Honeycomb or newer, an LRU Bitmap pool 1/10th the size of the memory available to the application to use to store Bitmaps of different sizes for reuse. 

By default the ImageManager will also save images to the disk cache as JPEGS and compress with quality at 90. If your images have transparent layers you will probably need to save images as PNG instead.

#### Changing options

ImageManagers are constructed using the builder pattern. For all the options available see [ImageManager.Builder](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/resize/ImageManager.java). The disk cache, memory cache, executor service, and bitmap pool all implement relatively simple interfaces so you can easily integrate other implementations.  

To change the disk cache size or location, you can instantiate a [DiskLruCacheWrapper](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/resize/cache/DiskLruCacheWrapper.java) your self and pass in whatever size and location you wish. You can get the default location from `ImageManager.getPhotoCacheDir()`.

Similarly, to change the memory cache size, you can instantiate an [LruPhotoCache](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/resize/cache/LruPhotoCache.java) your self and pass in whatever size you wish. 

To disable the disk cache, memory cache, or bitmap reuse, you can call `disableMemoryCache()`, `disableDiskCache()` and/or `disableBitmapRecycling()` respectively on the ImageManager.builder.

#### Bitmap reuse tradeoffs

Allowing Bitmaps to be reused can drastically reduce the number of garbage collections when scrolling large lists of images. There are a few tradeoffs to consider however.

Bitmaps can only be reused if their dimensions exactly match those of the image that is to be loaded into them. This means that bitmap reuse works best when images are being loaded from the disk cache at uniform sizes. It also means that it is difficult to reuse bitmaps when resizing them so you will still cause garbage collections when fetching and resizing images for the first time. 

Reusing bitmaps means maintaining some amount of memory to hold bitmaps that have been discarded, but not yet reused. If you tune your memory cache size exactly, this means you will need to use a smaller memory cache in order to allow space for the bitmap pool. 

Similarly, the larger the memory cache, the less effective the bitmap pool. Until you fill your memory cache with images, you will need to constantly allocate new bitmaps and will have none available for reuse. This means that the larger your memory cache, the more the user will have to scroll before it stops appearing to drop frames because of garbage collections. 

If you didn't use a memory cache at all, for example, you could start recycling bitmaps after the user scrolled past the first full page. However, every time the user scrolled backwards, they would always have to wait a brief period while the image was loaded asynchronously from the disk cache. 

There isn't a perfect memory and bitmap pool size for all applications. Instead it might make sense to customize it based on your users usage patterns. Short lists with images would probably benefit more from a larger memory cache since you could conceivably fit all of the images in the memory cache. Long lists would probably benefit more from a larger bitmap pool since you will never fit all of the images in the memory cache  and the user is likely to spend more time scrolling.  