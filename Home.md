### About Glide

Glide is image loading library for Android that provides a simple and high level interface for efficiently displaying large lists of remote images. 

Although Glide is primarily focused on loading images into memory, it includes an integrated interface for fetching remote or local images and includes reference implementations for loading local files, resources, and uris as well as remote urls. The url implementation is based on Google's [Volley](https://android.googlesource.com/platform/frameworks/volley/). However, the interface is designed to make it easy to integrate any other third party library as well.

To get started, see [[adding Glide to your project]].

You may also want to take a look at the [[tutorials]] or at the [documentation.](http://bumptech.github.io/glide/docs/index.html)

### Features

1. Glide has a simple interface

  For common models (Files and URLs), using Glide requires only a single line of code:

        Glide.load(yourUrlOrFile).into(yourImageView);

  You can also specify how to resize your image and add an animation in the same line:

        Glide.load(yourModel).centerCrop().animate(R.anim.your_anim_id).into(yourImageView);

  Glide also allows you to load images into arbitrary targets and/or apply arbitrary transformations:

        Glide.load(yourModel).transform(new CustomTransformation()).into(myNotificationTarget);

  For more complex data models, or for when you want to customize your file or url based on the size of your view, you implement the ModelLoader interface and then pass that in along with everything else in your call to load:

        Glide.using(yourModelLoader)
            .load(yourModel)
            .centerCrop()
            .animate(R.anim.your_anim_id)
            .into(yourImageView)
  
  It's also easy to integrate third party libraries if you don't fetch images with http or don't want to use Volley. For more information on using Glide with complex data models or integrating other libraries, see the [[Tutorials]].

2. Glide tells you the size image it needs

  Knowing exactly what size image is needed for a given view allows you to download the smallest possible image to fill the view saving time, battery life, and data. In addition, Glide uses the sizing information to downsample and crop images as they are loaded in to memory saving space both during the load and in the memory and disk caches. Glide is also robust enough to handle views whose dimensions are determined by layout weights so you don't need to do a bunch of unpleasant and error prone math yourself. 

3. Glide handles view recycling and job cancellation

  All you need to do is make a simple one line call to Glide for each call to `getView()` in you ListAdapter. Glide handles the rest including canceling old jobs and ensuring only the latest request populates the view.

4. Glide handles memory, disk, and http response caching

  Glide provides lru memory and disk caches to minimize the number of remote fetches and resize operations that have to be performed. The disk cache allows us not only to only resize and crop each image once, but it also makes subsequent loads very fast (on the order of 50ms depending on the image size), so even on older devices images are loaded quickly the second time around.

  The default disk cache uses Jake Wharton's excellent [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) library and the default memory cache is based on the Android project's [LruCache](http://developer.android.com/reference/android/support/v4/util/LruCache.html). Both simply implement interfaces so its easy to plug in your own disk or memory cache if you'd rather.

5. Glide recycles bitmaps to minimize jank inducing garbage collections 

  On devices with the Honeycomb or later, Glide will reuse bitmaps when loading images from the disk cache and while cropping images to minimize garbage collections. This is done completely transparently as images are loaded and then replaced in recycled views in Android's ListViews. Once images have been resized once and the disk cache is populated, Bitmap recycling makes scrolling smooth by drastically reducing garbage collections.

6. Glide reads and obeys exif orientation data

  Glide automatically orients jpeg and tiff images according to the exif data in the images' headers. The included exif parser reads directly from InputStreams so exif data can be read from any source without any additional IO. In contrast, Android's ExifInterface handles fields other than orientation, but is limited to files and requires two IO operations to both load the image and read the exif data.