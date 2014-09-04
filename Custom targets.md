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
    .into(new SimpleTarget(myWidth, myHeight) {
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
       

