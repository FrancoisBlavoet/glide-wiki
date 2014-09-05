In addition to loading images, video stills, and animated GIFs into [Views](http://developer.android.com/reference/android/view/View.html), you can also load media into custom [Target](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/Target.html) implementations.

## [SimpleTarget](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/SimpleTarget.html)

If you simply want to load a [Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html) so that you can interact with it in some special way other than displaying it directly to the user, maybe to show in a notification, or upload as a profile photo, Glide has you covered. 

[SimpleTarget](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/SimpleTarget.html) provides reasonable default implementations for the much larger [Target](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/Target.html) interface and let's you focus on handling the result of your load.

To use SimpleTarget, you need to provide the pixel width and height you'd like to load your resource at in to SimpleTarget's constructor, and you need to implement [``onResourceReady(T resource, GlideAnimation animation)``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/Target.html#onResourceReady(R, com.bumptech.glide.request.animation.GlideAnimation)). 

A typical SimpleTarget use case looks something like this:

```java
int myWidth = 512;
int myHeight = 384;

Glide.with(yourApplicationContext))
    .load(youUrl)
    .asBitmap()
    .into(new SimpleTarget<Bitmap>(myWidth, myHeight) {
        @Override
        public void onResourceReady(Bitmap bitmap, GlideAnimation anim) {
            // Do something with bitmap here.
        }
    };
```

### Caveats
Normally when you load resources, you're loading them into Views. When you fragment or activity is paused or destroyed, Glide will pause or cancel your load for you to make sure you're not wasting time or resources fetching media you're not going to display. 

For most SimpleTarget implementations, this isn't desirable behavior, so in your ``Glide.with(context)`` call, pass in your application context instead of your fragment or activity. 

In addition, consider that long running operations may lead to memory leaks. If you're going to perform a long running operation, consider using a static inner class instead of an anonymous inner class.
       
## [ViewTarget](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/ViewTarget.html)
You can override ViewTarget and/or its subclasses when you want to load an image into a [View](http://developer.android.com/reference/android/view/View.html) but you want to observe or override some part of Glide's default behavior.

ViewTarget is a base class you can use when you want Glide to handle determining the size of your View as normal, but when you want to handle starting an animation or setting the resource on the View yourself. Subclassing ViewTarget is particularly appropriate if you're loading an image into a custom View class or something other than an ImageView where Glide's built in [ImageViewTarget and subclasses](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/ImageViewTarget.html) won't work.

To create your own ViewTarget either statically define a new ViewTarget subclass, or pass in an anonymous inner class to your load call:

```java
Glide.with(yourFragment)
    .load(yourUrl)
    .into(new ViewTarget<YourViewClass, GlideDrawable>(yourViewObject) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation anim) {
            YourViewClass myView = this.view;
            // Set your resource on myView and/or start your animation here.
        }
    });
```

Note that if you want to load specifically a Bitmap or a GifDrawable, add ``.asBitmap()`` or ``.asGif()`` immediately after your ``.load(yourUrl)`` call and replace ``GlideDrawable`` in your ViewTarget's type parameter with the corresponding type you're loading.

For even more control, you can also implement the [LifecycleListener](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/manager/LifecycleListener.html) callbacks on your Target, including ``onStart()``, ``onStop()``, and/or ``onDestroy()`` which will be called in sync with the lifecycle of the [Fragment](http://developer.android.com/guide/components/fragments.html) containing your View.

## Overriding default behavior
If all you want to do is observe Glide's default behavior without changing it, you can subclass one of Glide's two default concrete ImageViewTargets:

* [GlideDrawableImageViewTarget](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/GlideDrawableImageViewTarget.html) - The default Target for normal loads and loads that use ``asGif()``.

* [BitmapImageViewTarget](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/BitmapImageViewTarget.html) - The default Target for loads that use ``asBitmap()``.

As long as you call super() for each method, the default behavior will remain the same and you can add in whatever functionality you wish.

For example to start generating a [Palette](http://chris.banes.me/2014/07/04/palette-preview/) you can do something like:

```java
Glide.with(yourFragment)
    .load(yourUrl)
    .asBitmap()
    .into(new BitmapImageViewTarget(yourImageView)) {
        @Override
        public void onResourceReady(Bitmap bitmap, GlideAnimation anim) {
            super.onResourceReady(bitmap, anim);
            Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {  
                @Override
                public void onGenerated(Palette palette) {
                    // Here's your generated palette
                }
            });
        }
    });
```

Although this is a great simple example, in general I wouldn't recommend generate Palettes this way. Instead checkout Glide's [ResourceTranscoder interface](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/resource/transcode/ResourceTranscoder.html) and [``.transcode()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/BitmapTypeRequest.html#transcode(com.bumptech.glide.load.resource.transcode.ResourceTranscoder, java.lang.Class)) method and consider returning a custom resource containing both your Bitmap and a Palette generated on a background thread. More on this coming soon...
 
        