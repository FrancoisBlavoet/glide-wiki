### About Glide

Glide is image loading library for Android that provides a simple and high level interface for efficiently displaying large lists of remote images. 

Although Glide is primarily focused on loading images into memory, it includes an integrated interface for fetching remote or local images and includes reference implementations for loading local files, resources, and uris as well as remote urls. The url implementation is based on Google's [Volley](https://android.googlesource.com/platform/frameworks/volley/). The interface is designed to make it easy to integrate any other third party library as well.

To get started, see [[adding Glide to your project]].

You may also want to take a look at the [[tutorials]] or at the [documentation.](http://bumptech.github.io/glide/docs/index.html)

### Features

Glide provides several important features:

1. A simple interface

  For common models (Files and URLs), using Glide requires only a single line of code:

        Glide.load(yourUrlOrFile).into(yourImageView);

  You can also specify how to resize your image and add an animation in the same line:

        Glide.load(yourModel).centerCrop().animate(R.anim.your_anim_id).into(yourImageView);

  For more complex data models, or for when you want to customize your file or url based on the size of your view, you implement the ModelLoader interface and then pass that in along with everything else in your call to load:

        Glide.load(yourModel)
            .with(yourModelLoader)
            .centerCrop()
            .animate(R.anim.your_anim_id)
            .into(yourImageView)
  
  It's also easy to integrate third party libraries if you don't fetch images with http or don't want to use Volley. For more information on using Glide with complex data models or integrating other libraries, see the [[Tutorials]].

2. The exact size the image needs to fill

  Knowing exactly what size image is needed for a given view allows you to download the smallest possible image to fill the view saving time, battery life, and data. It also allows Glide to downsample and crop images as they are loaded in to memory saving space both during the load and in the caches.

3. View recycling

  All you need to do is make a simple one line call to Glide for each call to `getView()` in you ListAdapter. Glide handles the rest including canceling old jobs and ensuring only the latest request populates the view.

4. Caching

  Glide provides lru memory and disk caches to minimize the number of remote fetches and resize operations that have to be performed. The disk cache allows us not only to only resize and crop each image once, but it also makes subsequent loads very fast (on the order of 50ms depending on the image size), so even on older devices images are loaded quickly the second time around.

  The default disk cache uses Jake Wharton's excellent [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) library and the default memory cache is based on the Android project's [LruCache](http://developer.android.com/reference/android/support/v4/util/LruCache.html). Both simply implement interfaces so its easy to plug in your own disk or memory cache if you'd rather.

5. Bitmap reuse  

  On devices with the Honeycomb or later, Glide will reuse bitmaps when loading images from the disk cache to minimize garbage collections. This is done completely transparently as images are loaded and then replaced in recycled views in Android's ListViews. Once images have been resized once and the disk cache is populated, Bitmap reuse makes scrolling smooth, even when images aren't all in the memory cache.

### Glide vs Picasso

Although Glide's interface is heavily influenced by [Picasso](http://square.github.io/picasso/), it isn't yet a drop in replacement. Glide does not currently allow you to load images for objects other than views and it does not allow you to easily specify an arbitrary transformation for an image, both of which Picasso allows. As a result, Glide is somewhat less general than Picasso.

If you need to specify arbitrary transformations or load images for arbitrary objects, Picasso is probably the better choice for now. I'm planning on implementing both of these features in the near future, but they aren't in place yet. If you are ok with the limited selection of transformations and just want the most efficient image loading and the smoothest possible scrolling, Glide is probably better than anything else out there.
  