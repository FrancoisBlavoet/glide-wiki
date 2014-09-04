## Default Transformations
Glide includes two default transformations, fit center and center crop.

### [Fit center](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/resource/bitmap/FitCenter.html) 
Fit center scales your image down maintaining the original aspect ratio of the image so that the image fits entirely within the given width and height. Fit center performs the minimal possible scale so that one dimension of the image exactly matches the given width or height and the other dimension of the image is less than or equal to the given width or height. 

Fit center performs the same transformation as Android's [ScaleType.FIT_CENTER](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html).

### [Center crop](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/resource/bitmap/CenterCrop.html)
Center crop first scales your image down maintaining the original aspect ratio so that one dimension of the image is exactly equal to your target dimensions and the other dimension of the image is greater than your target dimensions. Center crop then crops out the center portion of the image. 

Center crop performs the same transformation as Android's [ScaleType.CENTER_CROP](http://developer.android.com/reference/android/widget/ImageView.ScaleType.html)


### Usage

To apply fit center, use [``.fitCenter()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/DrawableRequestBuilder.html#fitCenter()):
```java
Glide.with(yourFragment)
    .load(yourUrl)
    .fitCenter()
    .into(yourView);
```

For center crop, use [``.centerCrop()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/DrawableRequestBuilder.html#centerCrop()):
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

In addition to the two built in transformations, you can also define your own custom [Transformation](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/Transformation.html). 

The easiest way to do so is to subclass [BitmapTransformation](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/resource/bitmap/BitmapTransformation.html):

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

You can then use your custom transformation in the same way you can use the default transformations, only replace ``.fitCenter()`` or ``.centerCrop()`` with [``.transform(...)``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/DrawableRequestBuilder.html#transform(com.bumptech.glide.load.resource.bitmap.BitmapTransformation...)).

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
You may have noticed that none of the above examples include any concrete dimensions, so how does Glide determine the size that is passed in to each [Transformation](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/Transformation.html)? 

Glide is smart enough to figure out the size of the [View](http://developer.android.com/reference/android/view/View.html) you're loading your image into and will pass the View's dimensions along with the original image you retrieve into your Transformation so you can make sure your transformation gives you exactly the output you need.

If you want override the View's dimensions, you can do so with the [``.override(int, int)``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/DrawableRequestBuilder.html#override(int, int)) method. If want to load an image for some reason other than display it in a view, see the [Custom targets](https://github.com/bumptech/glide/wiki/Custom-targets) page.

## Caveats
Currently all Transformations must be idempotent, meaning that you should get the same output if you apply your transformation once as if you were to apply it a hundred (or more than one) times in a row. This restriction should be lifted in Glide 3.4, see [Issue #116](https://github.com/bumptech/glide/issues/116) for details.

The workaround for now if you need to have a transformation that is not idempotent is to use ``.diskCacheStrategy(DiskCacheStrategy.SOURCE)``. Using that [DiskCacheStrategy](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/engine/DiskCacheStrategy.html) will only cache the source of your images, not the transformed thumbnails and will therefore make sure that your transformations are only applied once.
