Glide's [ModelLoader](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/model/ModelLoader.java) interface provides developers with the size of the view they are loading an image into and allows them to use that size to choose a url to download an appropriately sized version of the image. 

Using appropriately sized image saves bandwidth, storage space on the device, and improves the performance of the app.

The 2014 Google I/O app team wrote a post about how they used the ModelLoader interface to adjust the size of the images they loaded on the Github page for the [I/O app source](https://github.com/google/iosched/blob/master/doc/IMAGES.md).

To implement your own ModelLoader to download images over http or https, you can extend [BaseGlideUrlLoader](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/model/stream/BaseGlideUrlLoader.java): 

```java
public interface MyDataModel {
    public String buildUrl(int width, int height);
} 

public class MyUrlLoader extends BaseGlideUrlLoader<MyDataModel> {
	@Override
	protected String getUrl(MyDataModel model, int width, int height) {
		// Construct the url for the correct size here.
        return model.buildUrl(width, height);
	}
}
```

You can then use your custom ModelLoader to load images, and everything else works automatically:

```java
Glide.with(yourFragment)
    .using(new MyUrlLoader())
    .load(yourModel)
    .into(yourView);
```

If you want to avoid calling ``.using(new MyUrlLoader())`` each time, you can also implement a custom [ModelLoaderFactory](https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/load/model/ModelLoaderFactory.java) and register it with Glide in your [``GlideModule``][1].

```java
public class MyGlideModule implements GlideModule {
    ...
    @Override
    public void registerComponents(Context context, Glide glide) {
        glide.register(MyDataModel.class, InputStream.class, 
            new MyUrlLoader.Factory());
    }
}
```

After registering the ModelLoaderFactory, you can skip the ``.using()`` call and just call:

```java
Glide.with(yourFragment)
    .load(yourModel)
    .into(yourView);
```
 
For another example of how to load a variety of image sizes using a custom ModelLoader, see Glide’s [Flickr sample app](https://github.com/bumptech/glide/blob/master/samples/flickr/src/main/java/com/bumptech/glide/samples/flickr/FlickrModelLoader.java) or Glide’s [Giphy sample app](https://github.com/bumptech/glide/blob/master/samples/giphy/src/main/java/com/bumptech/glide/samples/giphy/GiphyModelLoader.java).

[1]: https://github.com/bumptech/glide/wiki/Configuration