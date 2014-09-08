Cache invalidation is a relatively complex topic and ideally you have to think about it as little as possible. The goal of this page is to give you a rough idea of how cache keys are generated in Glide and some hints as to how to make the cache work best for you.

## Cache Keys
Cache keys in Glide are comprised of three main parts:

* The String returned from the [``getId()``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/data/DataFetcher.html#getId()) method in DataFetcher - Defaults to the String version of the model you provide, so your url if you provide a url, or your file path if you provide a file path etc.

* The width and height passed in to the [``override(int, int)``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/DrawableRequestBuilder.html#override(int, int)) method if called, or by default the width and height provided by your [Target's ``getSize()`` method.](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/request/target/Target.html#getSize(com.bumptech.glide.request.target.SizeReadyCallback))

* The Strings returned from the ``getId()`` methods of the various decoders and encoders used to load and cache your image. Only the decoders and encoders that affect the retrieved bytes include ids. For example if you have an encoder that simply writes a byte array to disk, that encoder will not have an id because it does not affect the data in any way.

All of these keys are hashed in a particular order to create a unique and safe File name to save a particular image on disk.

## Cache Invalidation
Because File names are hashed keys, there is no good way to simply delete all of the cached files on disk that correspond to a particular url or file path. The problem would be simpler if you were only ever allowed to load or cache the original image, but since Glide also caches thumbnails and provides various transformations, each of which will result in a new File in the cache, tracking down and deleting every cached version of an image is practically impossible.

In practice, the best way to invalidate a cache file is to make sure the String returned from your DataFetcher's ``getId()`` method changes:

* Urls - For urls you can invalidate your cache keys by changing your url when you change the image, usually by adding some version or timestamp to the url that changes when the corresponding image changes.

* Media store content - For media store content, you can use Glide's [``loadFromMediaStore(...)``](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/RequestManager.html#loadFromMediaStore(android.net.Uri, java.lang.String, long, int)) method, which mixes in the date modified time, mime type, and orientation into the cache key to cache updates and edits.

* Other local content - For any non media store content, you can invalidate your cache keys in the same way Glide invalidates media store content by mixing in some data that identifies when the image changed, like the date modified time on the File, exif data etc. To do so, you will need to implement two simple interfaces as described below.

## Custom Cache Invalidation
Although ideally you can always invalidate your cached media by changing the identifies (urls, file paths etc) directly, you you can also relatively easily write your own [ModelLoader](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/model/ModelLoader.html) and [DataFetcher](http://bumptech.github.io/glide/javadocs/330/com/bumptech/glide/load/data/DataFetcher.html) implementations to mix in some additional data so you can have more fine grained control over invalidation.

Here's an example that invalidates local files by keeping a version number per file in [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html):

First define a POJO data model containing your local file and some key that can be used to retrieve the version from SharedPreferences:

```java
public final class FileDataModel {
    public final String versionKey;
    public final File file;
    ...
}
```

Then the DataFetcher:

```java
public class FileInvalidationDataFetcher implements DataFetcher<InputStream> {
    private final File file;
    private final int version;

    public FileInvalidationDataFetcher(File file, int version) {
        this.file = file;
        this.version = version;
    }

    @Override
    public InputStream loadData(Priority priority) throws Exception {
        return new FileInputStream(file);
    }

    @Override
    public String getId() {
        return file.toString() + version;
    }
}
```

Now your ModelLoader:

```java
public class FileDataModelLoader implements StreamModelLoader<FileDataModel> {
    private final Context context;

    public FileDataModelLoader(Context context) {
        this.context = context.getApplicationContext();
    }

    public DataFetcher<InputStream> getResourceFetcher(FileDataModel model, int width, int height) {
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(context);
        int version = prefs.getInt(model.versionKey, 0);
        File file = model.file;
        return new FileInvalidationDataFetcher(file, version);
    }
}
```

Finally you can use your custom model loader by passing it in to your load calls:

```java
Glide.with(yourFragment)
    .load(yourFileDataModel)
    .using(new FileDataModelLoader(yourFragment.getActivity())
    .into(yourImageView);
```
        
Whenever the data in the File changes, just increment the version number in shared preferences and all thumbnails will be regenerated, regardless of the sizes or transformations you may have applied.
        
    


