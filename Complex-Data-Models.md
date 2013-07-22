#### Complex data models

Using Glide with a data model more complex than a simple File or URL requires implementing the [ModelLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/model/ModelLoader.java) interface.

The purpose of the ModelLoader interface is to translate an arbitrarily complex data model into a concrete data type that we can then pass to a [StreamLoader](https://github.com/bumptech/glide/tree/master/library/src/com/bumptech/glide/loader/stream) to actually fetch

#### Http using [Volley](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CCoQtwIwAA&url=http%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dyhv8l9F44qo&ei=IWjtUerhNoW_qgGYhoHwBA&usg=AFQjCNEKoQ4Fs-VJ-0VJuP4LFA1s5tUpgw)

If you simply need to construct a url to fetch a given model using http, the easiest thing to do is subclass [VolleyModelLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/model/VolleyModelLoader.java). Volley is a fast parallelized networking library for Android that we've integrated with Glide to make fetching remote images easy and fast. 

VolleyModelLoader requires you to implement two simple methods, `getId(Model model)` and `getUrl(Model mode, int width, int height)`. 

`getId()` just needs to return some String unique to the particular model that can be used as a key in a cache. An id field in a database is an obvious example, though other things like a path or file name can work as well. 

`getUrl()` is where you construct your URL from the given model and dimensions. This URL may then later be passed to Volley to fetch the image if it is not cached.

For a simple example of a VolleyModelLoader, see [FlickrModelLoader](https://github.com/bumptech/glide/blob/master/samples/flickr/src/com/bumptech/flickr/FlickrModelLoader.java) in the Flickr sample project.

#### Generic remote or local assets

If you don't use http, are loading images from local files, or don't want to use Volley, you can either use or subclass one of the other included [ModelLoaders](https://github.com/bumptech/glide/tree/master/library/src/com/bumptech/glide/loader/model), or you can extend 
[BaseModelLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/model/BaseModelLoader.java). 

BaseModelLoader is a simple abstract implementation of the ModelLoader interface that includes some generic housekeeping. BaseModelLoader also requires you to implement two methods, `getId(Model model)` and `buildStreamLoader(Model mode, int width, int height)`.

`getId()` requires you to return a unique String identifying this particular model, the same as with VolleyModelLoader.

`buildStreamLoader()` is slightly more complex and requires you to return a [StreamLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/stream/StreamLoader.java). Ideally you can simply translate your model into one of the types for which Glide already includes a [StreamLoader implementation](https://github.com/bumptech/glide/tree/master/library/src/com/bumptech/glide/loader/stream).

A simple implementation of BaseModelLoader for files is something like this:

    public class FileLoader extends BaseModelLoader<YourModel> {
        @Override
        public String getId(YourModel model) {
            return model.getId(); 
        }
 
        public StreamLoader buildStreamLoader(YourModel model, int width, int height) {
            File file = getFileFor(model);
            return new FileStreamLoader(file);
        }
    }
   
Where FileStreamLoader is an included StreamLoader for Files. 

#### Generic assets with external libraries

Although Glide includes an implementation of ModelLoader and StreamLoader based on Volley, you can also easily produce a similar implementation for any third party library or method of fetching assets. To do so, you will need to implement BaseModelLoader as above, but you will also need to define a new [StreamLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/stream/StreamLoader.java). 

StreamLoaders are the objects that actually fetch an image, or call in to a third party library, from a specific type (URL, file etc). 

For example, Bump uses some shared code written in C to download images from a UUID that identifies an image and call a callback with a path to the image when the download completes. Therefore, we've implemented a BaseModelLoader and StreamLoader that look something like this:

    public class BumpImageLoader extends BaseModelLoader<BumpImage> {
        private final BumpImageDownloader imageDownloader;
    
        public BumpImageLoader(BumpImageDownloader imageDownloader) {
            this.imageDownloader = imageDownloader;
        }
  
        @Override
        public String getId(BumpImage image) {
            return image.getUuid().toString();
        }

        @Override
        public StreamLoader buildStreamLoader(BumpImage image, int width, int height) {
            final UUID uuid = image.getUuid();
            return new StreamLoader() {
                @Override
                public void loadStream(StreamReadyCallback cb) {
                     imageDownloader.downloadImage(uuid, width, height, new ImageDownloader.Callback() {
                         @Override
                         public void onDownloadComplete(String path) {
                             try {
                                 cb.onStreamReady(new FileInputStream(path));
                             } catch (FileNotFoundException e) {
                                 cb.onException(e);
                             }
                      }
                 }
                 
                 @Override
                 public void cancel() { 
                     imageDownloader.cancel(uuid);
                 }
             }
         }
    }

    For other examples take a look at the [StreamLoader implementations](https://github.com/bumptech/glide/tree/master/library/src/com/bumptech/glide/loader/stream) and [ModelLoader implementations](https://github.com/bumptech/glide/tree/master/library/src/com/bumptech/glide/loader/model)


