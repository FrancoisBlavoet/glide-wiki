### About Glide

Glide is image loading library for Android that provides a simple and high level interface for efficiently displaying large lists of remote images.


***

Glide provides several important features:

1. Determining the size of views at runtime

  Knowing exactly what size image is needed for a given view allows you to download the smallest possible image to fill the view saving time, battery life, and data. It also allows Glide to downsample and crop images as they are loaded in to memory saving space both during the load and in the caches.

2. Managing view recycling 

  All you need to do is make a simple one line call to Glide for each call to `getView()` in you ListAdapter. Glide handles the rest including canceling old jobs and ensuring only the latest request populates the view.

3. Caching

  Glide provides lru memory and disk caches to minimize the number of remote fetches and resize operations that have to be performed. The disk cache allows us not only to only resize and crop each image once, but it also makes subsequent loads very fast (on the order of 50ms depending on the image size), so even on older devices images are loaded quickly the second time around.

  The default disk cache uses Jake Wharton's excellent [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) library and the default memory cache is based on the Android projects [LruCache](http://developer.android.com/reference/android/support/v4/util/LruCache.html). Both simply implement interfaces so its easy to plug in your own disk or memory cache if you'd rather.

4. Bitmap reuse  

  On devices with the Honeycomb or later, Glide will recycle bitmaps when loading images from the disk cache to minimize garbage collections. This is done completely transparently as images are loaded and then replaced in recycled views in Android's ListViews. Once images have been resized once and the disk cache is populated, Bitmap reuse makes scrolling smooth, even when images aren't all in the memory cache.

5. A simple interface

  Inspired by Square's [Picasso](http://square.github.io/picasso/), Glide provides a simple high level interface. Users looking to integrate third party libraries or perform more advanced customizations can also take a look at the [tutorials] for help implementing the various available interfaces. 

  For common models (Files and URLs), using Glide requires only a single line of code:

        Glide.load(yourUrlOrFile).into(yourImageView).begin();

  You can also specify how to resize your image and add an animation in the same line:

        Glide.load(yourModel).into(yourImageView).centerCrop().animate(R.anim.your_anim_id).begin();

  For more complex data models, or for when you want to customize your file or url based on the size of your view, you implement a simple interface and then pass that in along with everything else in your call to load:

        Glide.load(yourModel)
            .into(yourImageView)
            .with(yourModelLoader)
            .centerCrop()
            .animate(R.anim.your_anim_id)
            .begin();
   
  For more information implementing the ModelStreamLoader interface to handle more complex data models, see the [tutorial].
  
