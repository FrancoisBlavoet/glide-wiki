## Default Transformations
Glide includes two default transformations, fit center and center crop. For other types of transformations you may want to consider an [unaffiliated transformations library][1].

### [Fit center](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/resource/bitmap/FitCenter.html) 
Fit center scales your image down maintaining the original aspect ratio of the image so that the image fits entirely within the given width and height. Fit center performs the minimal possible scale so that one dimension of the image exactly matches the given width or height and the other dimension of the image is less than or equal to the given width or height. 

Fit center performs the same transformation as Android's [ScaleType.FIT_CENTER](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html).

### [Center crop](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/resource/bitmap/CenterCrop.html)
Center crop first scales your image down maintaining the original aspect ratio so that one dimension of the image is exactly equal to your target dimensions and the other dimension of the image is greater than your target dimensions. Center crop then crops out the center portion of the image. 

Center crop performs the same transformation as Android's [ScaleType.CENTER_CROP](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html)

### Usage

To apply fit center, use [``.fitCenter()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#fitCenter()):
```java
Glide.with(yourFragment)
    .load(yourUrl)
    .fitCenter()
    .into(yourView);
```

For center crop, use [``.centerCrop()``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#centerCrop()):
```java
Glide.with(yourFragment)
    .load(yourUrl)
    .centerCrop()
    .into(yourView);
```

You can also apply either of the default transformations if you're loading Bitmaps or GIFs:
```java
// For Bitmaps:
Glide.with(yourFragment)
    .load(yourUrl)
    .asBitmap()
    .centerCrop()
    .into(yourView);

// For gifs:
Glide.with(yourFragment)
    .load(yourUrl)
    .asGif()
    .fitCenter()
    .into(yourView);
```

You can even apply transformations when transcoding between types, for example to get the jpeg encoded bytes of a transformed image, you can use:

```java
Glide.with(yourFragment)
    .load(yourUrl)
    .asBitmap()
    .toBytes()
    .centerCrop()
    .into(new SimpleTarget<byte[]>(...) { ... });
```

## Custom transformations

In addition to the two built in transformations, you can also define your own custom [Transformation](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/Transformation.html). 

The easiest way to do so is to subclass [BitmapTransformation](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html):

```java
private static class MyTransformation extends BitmapTransformation {

    public MyTransformation(Context context) {
       super(context);
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, 
            int outWidth, int outHeight) {
       Bitmap myTransformedBitmap = ... // apply some transformation here.
       return myTransformedBitmap;
    }

    @Override
    public String getId() {
        // Return some id that uniquely identifies your transformation.
        return "com.example.myapp.MyTransformation";
    }
}
```

You can then use your custom transformation in the same way you can use the default transformations, only replace ``.fitCenter()`` or ``.centerCrop()`` with [``.transform(...)``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#transform(com.bumptech.glide.load.resource.bitmap.BitmapTransformation...)).

For example:

```java
// For the default drawable type:
Glide.with(yourFragment)
    .load(yourUrl)
    .transform(new MyTransformation(context))
    .into(yourView);

// For Bitmaps:
Glide.with(yourFragment)
    .load(yourUrl)
    .asBitmap()
    .transform(new MyTransformation(context))
    .into(yourView);

// For Gifs:
Glide.with(yourFragment)
    .load(yourUrl)
    .asGif()
    .transform(new MyTransformation(context))
    .into(yourView);
```

### Sizing
You may have noticed that none of the above examples include any concrete dimensions, so how does Glide determine the dimensions that are passed in to each [Transformation](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/Transformation.html)? 

The dimensions that are passed into each Transformation are those of the View or Target the load is for. Glide is smart enough to figure out the sizes of [Views](http://developer.android.com/reference/android/view/View.html) that use layout weights or match_parent as well as the sizes of those that have concrete dimensions. Having the dimensions of your View or Target as well as the original image allow your transformation to produce an image of exactly the right size.

If you want to specify custom dimensions instead of using those of your View or Target, you can do so using the [``.override(int, int)``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#override(int, int)) method. If want to load an image for some reason other than display it in a View, see the [Custom targets](https://github.com/bumptech/glide/wiki/Custom-targets) page.

### Bitmap re-use
To reduce garbage collections, you can use the provided [``BitmapPool``](http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/bitmap_recycle/BitmapPool.html) interface to release unwanted [``Bitmaps``](http://developer.android.com/reference/android/graphics/Bitmap.html) or re-use existing Bitmaps. Typically you re-use a Bitmap in a Transformation by getting a Bitmap from the pool, using the Bitmap from the pool to back a [``Canvas``](http://developer.android.com/reference/android/graphics/Canvas.html), and then drawing your original Bitmap onto the Canvas using a [Matrix](http://developer.android.com/reference/android/graphics/Matrix.html), [Paint](http://developer.android.com/reference/android/graphics/Paint.html), or [Shader](http://www.curious-creature.org/2012/12/11/android-recipe-1-image-with-rounded-corners/) to transform the image. To efficiently and correctly re-use Bitmaps in Transformations that are a couple of rules to follow:

1. Never recycle the Resource or return to the BitmapPool the Bitmap you are given in ``transform()``, this is done automatically. 
2. If you get more than one Bitmap from the BitmapPool or do not use a Bitmap you get from the BitmapPool, be sure to return any extras to the BitmapPool.
3. If your Transformation does not replace the original resource (for example if your Transformation returns early because an image already matches the size you need), return the original Resource or Bitmap from the ``transform()`` method.
 
A typical pattern looks something like this:

```java
protected Bitmap transform(BitmapPool bitmapPool, Bitmap original, int width, int height) {
    Bitmap result = bitmapPool.get(width, height, Bitmap.Config.ARGB_8888);
    // If no matching Bitmap is in the pool, get will return null, so we should allocate.
    if (result == null) {
        // Use ARGB_8888 since we're going to add alpha to the image.
        result = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
    }
    // Create a Canvas backed by the result Bitmap.
    Canvas canvas = new Canvas(result);
    Paint paint = new Paint();
    paint.setAlpha(128);
    // Draw the original Bitmap onto the result Bitmap with a transformation.
    canvas.drawBitmap(original, 0, 0, paint);
    // Since we've replaced our original Bitmap, we return our new Bitmap here. Glide will
    // will take care of returning our original Bitmap to the BitmapPool for us. 
    return result;
}
```

[1]: https://github.com/wasabeef/glide-transformations
