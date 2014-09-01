### What's new in 3.0
Glide 3.0 includes a large number of new features. Some of the more notable features include:
* **Animated GIF decoding** - Just use the same ``Glide.with(...).load(...)`` call and if the image you load is an animated GIF, Glide will load and display a custom animated drawable containing the gif. For more control you can use ``Glide.with(context).load(...).asBitmap()`` to always load a static image, or ``Glide.with(context).load(...).asGif()`` to fail unless the image is an animated gif.

* **Local video stills** - In addition to decoding GIFs, Glide can also now decode stills from videos on your device. Just like with GIFs, the same ``Glide.with(...).load(...)`` call will work for any local video Android can decode.

* **Thumbnail support** - You can now load multiple images into the same view at the same time so you can minimize the amount of time your users spend looking at loading spinners without sacrificing quality. To first load a thumbnail at 1/10th the size of your view and then load the full image on top, use:  ``Glide.with(yourFragment).load(yourUrl).thumbnail(0.1f).into(yourView)``. For more control you can also pass in a completely new request into the ``.thumbnail()`` call.

* **Lifecycle integration** - Requests are now paused automatically in onStop and restarted in onStart. Animated GIFs are also paused in onStop to avoid draining battery in the background. In addition when the device's connectivity status changes, all failed requests are automatically restarted to make sure no request permanently fails because of transient connectivity problems.

* **Transcoding** - In addition to decoding resources, Glide's ``.toBytes()`` and ``.transcode()`` methods allow you to fetch, decode, and transform an image in the background as normal, but also in the same call, transcode the image into some more useful format. For example, to upload the bytes of a 250px by 250px profile photo for a user:

```java
Glide.with(context)
	.load(“/user/profile/photo/path”)
	.asBitmap()
	.toBytes()
	.centerCrop()
	.into(new SimpleTarget<byte[]>(250, 250) {
		@Override
		public void onResourceReady(byte[] data, GlideAnimation anim) {
			// Post your bytes to a background thread and upload them here.
		}
	});
```

* **Animations** - Glide 3.x adds support for cross fades (``.crossFade()``), and view property animations (``.animate(ViewPropertyAnimation.Animator)``), in addition to the original [view animations](http://developer.android.com/reference/android/view/animation/package-summary.html) introduced in Glide 2.0.

* **OkHttp and Volley Support** - You can now choose to use either OkHttp, or Volley, or Glide's HttpUrlConnection default as your network stack. OkHttp and Volley can be included by adding a dependency on the corresponding integration library and registering the corresponding ModelLoaderFactory. See the ReadMe for more details.

* **Many More** - Including the ability to use Drawables objects as placeholders during loads, request prioritization, width and height overrides and the ability to cache transformed thumbnails and/or the original source. 

### Migrating from 2.0 to 3.0

* All ``Glide.load()`` calls are now ``Glide.with([fragment/activity/context]).load()``

* All custom load calls, ``Glide.load(url).into(new SimpleTarget(){ ... }).with(context)`` are now just ``Glide.with(context).load(url).into(new SimpleTarget(width, height) { ... })``

### Features
In addition to the new features introduced in the the 3.x version, Glide carries over all of the originals from the 2.x version:

* Background image loading.

* Automatic job cancellation in lists where views are re-used.

* Memory and disk caching

* Bitmap and resource pooling to minimize jank.

* Arbitrary transformations.
