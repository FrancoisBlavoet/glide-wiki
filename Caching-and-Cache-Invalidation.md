Cache invalidation is a relatively complex topic and ideally you have to think about it as little as possible. The goal of this page is to give you a rough idea of how cache keys are generated in Glide and some hints as to how to make the cache work best for you.

## Cache Keys
Cache keys used for [``DiskCacheStrategy.RESULT``][15] in Glide are comprised of four main parts:

* The String returned from the [``getId()``][1] method in DataFetcher. Typically DataFetchers simply return the result of calling ``toString()`` on your data model, so the string version of your ``URL`` if you provide a ``URL``, or the file path if you provide a ``File`` etc.

* The width and height passed in to the [``override(int, int)``][2] method if called, or by default the width and height provided by your [Target's ``getSize()`` method.][3]

* The Strings returned from the ``getId()`` methods of the various decoders and encoders used to load and cache your image. Only the decoders and encoders that affect the retrieved bytes include ids. For example if you have an encoder that simply writes a byte array to disk, that encoder will not have an id because it does not affect the data in any way.

* An optional signature you may apply to each load (see Custom cache invalidation below). 

All of these keys are hashed in a particular order to create a unique and safe File name to save a particular image on disk.

## Cache Invalidation
Because File names are hashed keys, there is no good way to simply delete all of the cached files on disk that correspond to a particular url or file path. The problem would be simpler if you were only ever allowed to load or cache the original image, but since Glide also caches thumbnails and provides various transformations, each of which will result in a new File in the cache, tracking down and deleting every cached version of an image is difficult.

In practice, the best way to invalidate a cache file is to change your identifier when the content changes (url, uri, file path etc).

## Custom cache invalidation
Since it's often difficult or impossible to change identifiers, Glide also offers the [``signature()``][9] API to mix in additional data that you control into your cache key. Signatures work well for media store content, as well as any content you can maintain some versioning metadata for.

* Media store content - For media store content, you can use Glide's [``MediaStoreSignature``][12] class as your signature. MediaStoreSignature allows you to mix the date modified time, mime type, and orientation of a media store item into the cache key. These three attributes reliably catch edits and updates allowing you to cache media store thumbs.

* Files - You can use [``StringSignature``][11] to mix in the File's date modified time.

* Urls - Although the best way to invalidate urls is to make sure the server changes the url and updates the client when the content at the url changes, you can also use [``StringSignature``][11] to mix in arbitrary metadata (such as a version number) instead.

Passing in string signatures to loads is simple:

```java
Glide.with(yourFragment)
    .load(yourFileDataModel)
    .signature(new StringSignature(yourVersionMetadata))
    .into(yourImageView);
```

The media store signature is also straightforward [data from the MediaStore][12]:

```java
Glide.with(fragment)
    .load(mediaStoreUri)
    .signature(new MediaStoreSignature(mimeType, dateModified, orientation))
    .into(view);
```

You can also define your own signature by implementing the [``Key``][10] interface. Be sure to implement ``equals()``, ``hashCode()`` and the ``updateDiskCacheKey()`` method:

```java
public class IntegerVersionSignature implements Key {
    private int currentVersion;

    public IntegerVersionSignature(int currentVersion) {
         this.currentVersion = currentVersion;
    }
   
    @Override
    public boolean equals(Object o) {
        if (o instanceof IntegerVersionSignature) {
            IntegerVersionSignature other = (IntegerVersionSignature) o;
            return currentVersion = other.currentVersion;
        }
        return false;
    }
 
    @Override
    public int hashCode() {
        return currentVersion;
    }

    @Override
    public void updateDiskCacheKey(MessageDigest md) {
        messageDigest.update(ByteBuffer.allocate(Integer.SIZE).putInt(signature).array());
    }
}
```
   
Keep in mind that to avoid degrading performance, you will want to batch load any versioning metadata in the background so that it is available when you want to load your image.

If all else fails and you can neither change your identifier nor keep track of any reasonable version metadata, you can also disable disk caching entirely using [``diskCacheStrategy()``][13] and [``DiskCacheStrategy.NONE``][14].


## Dialog with another persistence layer

In some cases, you might want to integrate Glide with another persistence module.  
One example would be an app allowing its users to pin some content in order to make it available any time, even while offline.  

Glide does not make any special treatment for a specific key in its LRU cache, so any medium can be evicted at any time. This goes contrary to the pin feature. In order to implement it, it is necessary for Glide to interact with the module responsible for offline content (it's implementation is outside of Glide's scope).  

One way to accomplish this is to use a [``MediaLoader``][16] which provides 2 [``DataFetchers``][17] : 

* One that can interrogate the application's persistence module, in charge of handling pinned content.  
* One's that will make a network call if the first one fails.  

That way we implement the following request path :  

* Check for the resource in the Memory LRU cache.
* Check for the resource in the Disk LRU cache.
* Check for the resource in the persistence module
* If all else has failed, make a network to try to fetch the resource.

The [``MediaLoader``][16] implementation is pretty straightforward, its only goal here is to provide the custom DataFetcher class:  
```java 
public class SynchronizableImageLoader<Model> implements StreamModelLoader<Model> {
    
    /**
    * we wrap our implementation around Glide's own ModelLoader. 
    */
    private final ModelLoader<GlideUrl, InputStream> mBaseLoader;
 
    public SynchronizableImageLoader(Context context) {
        mBaseLoader = Glide.buildModelLoader(GlideUrl.class, InputStream.class, context);
    }
 
    @Override
    @Nullable
    public final DataFetcher<InputStream> getResourceFetcher(@Nullable Model model,
                                                             int width,
                                                             int height) {
                                                             
       [...]
       // We use mBaseLoader in order to provide a NetworkFetcher to our SynchronizableDataFetcher :
       DataFetcher<InputStream> networkFetcher = mBaseLoader.getResourceFetcher(glideUrl, width, height);
        return new SynchronizableDataFetcher<Model>(networkFetcher, model, width, height);
    }
 
}
```

The  [``DataFetcher``][17] implementation is where you will dialog with the persistence module :  
```java
public class SynchronizableDataFetcher<Model> implements DataFetcher<InputStream> {
 
    @Nullable private final DataFetcher<InputStream> networkFetcher;
    @NonNull private final Model model;
    private final int width;
    private final int height;
 
    public SynchronizableDataFetcher(@Nullable DataFetcher<InputStream> networkFetcher,
                                     @NonNull Model model,
                                     int width,
                                     int height) {
        this.networkFetcher = networkFetcher;
        this.model = model;
        this.width = width;
        this.height = height;
    }
 
    @Override
    public InputStream loadData(Priority priority) throws Exception {
        // first, let's try to query our persistence module where the pinned content might be :
        InputStream result = loadFromPersistenceModule(model, width, height);
        
        // if the content is not available there, fall back on a traditional network fetch :
        if (result == null && networkFetcher != null) {
                result = networkFetcher.loadData(priority);
        }
        
        return result;
    }
 
    @Override
    public void cleanup() {
        if (networkFetcher != null) networkFetcher.cleanup();
    }
 
    @Override
    public void cancel() {
        if (networkFetcher != null) networkFetcher.cancel();
    }
```

You can find a full gist [here][18]



[1]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/data/DataFetcher.html#getId()
[2]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#override(int,%20int)
[3]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/request/target/Target.html#getSize(com.bumptech.glide.request.target.SizeReadyCallback)
[6]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/model/ModelLoader.html
[7]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/data/DataFetcher.html
[8]: http://developer.android.com/reference/android/content/SharedPreferences.html
[9]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#signature(com.bumptech.glide.load.Key)
[10]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/Key.html
[11]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/signature/StringSignature.html
[12]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/signature/MediaStoreSignature.html
[13]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/DrawableRequestBuilder.html#diskCacheStrategy(com.bumptech.glide.load.engine.DiskCacheStrategy)
[14]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/engine/DiskCacheStrategy.html#NONE
[15]: http://bumptech.github.io/glide/javadocs/350/com/bumptech/glide/load/engine/DiskCacheStrategy.html#RESULT
[16]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/model/ModelLoader.html
[17]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/data/DataFetcher.html
[18]: https://gist.github.com/Teovald/bfddf6cbddc8e61ade80