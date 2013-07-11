There are three basic things you need to do to use Glide in your project.

1. Create an ImageManager in your Activity
2. Implement the ModelStreamLoader interface to translate your data model into InputStreams for the image you want to show
3. Create an ImagePresenter with your ModelStreamLoader interface a predefined ImageLoader subclass that matches how you want your images resized

#### Create an ImageManager in your Activity

The ImageManager is the backbone of Glide. It starts a background thread and a thread pool to load and resize images from disk in the background, manage bitmap recycling, and track or cancel scheduled jobs. The ImageManager is not usually used directly, but rather is passed to an ImageLoader implementation you select as part of creating your ImagePresenter.

The code is fairly straight forward. In your Activity:

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        this.imageManager = new ImageManager.Builder(MyActivity.this).build(); 
    }

    @Override
    protected void onDestroy() {
        this.imageManager.shutdown(); //shutdown our thread pool and background thread
    }

You will need to be able to reference your ImageManager wherever you are creating your ImagePresenter. At Bump, we've found it helpful to add the above code to a superclass of all of our activities along with a `getImageManager()` method so it is always easily accessible and we don't have to add code to every activity class individually.

The flickr sample's [FlickrSearchActivity](https://github.com/sjudd/glide/blob/master/samples/flickr/src/com/bumptech/flickr/FlickrSearchActivity.java) includes creating and destroying an ImageManager.

#### Implement the ModelStreamLoader interface

The ModelStreamLoader interface is a generic interface that translates between your data model and an InputStream that we can actually use to get the bytes of a bitmap representing the image you want to show. 

The interface uses a callback to allow you to call in to some third party library if you use one to download your images (see [FlickrStreamLoader](https://github.com/sjudd/glide/blob/master/samples/flickr/src/com/bumptech/flickr/FlickrStreamLoader.java)). Most people probably don't need anything that complex and instead should use the [DirectModelStreamLoader](https://github.com/sjudd/glide/blob/master/library/src/com/bumptech/glide/loader/model/DirectModelStreamLoader.java) class. 

The DirectModelStreamLoader requires you to implement two methods:

`getId()` needs to return some unique string id that uniquely identifies the image the model represents. An Iid from a database works, but often so does the string version of a URI, path, or url. 

`getStreamOpener()` is essentially just a way of lazily opening InputStreams so that we avoid doing http calls or disk io if the image is already cached. Two StreamOpener implementations are provided: [FileInputStreamsOpener](https://github.com/sjudd/glide/blob/master/library/src/com/bumptech/glide/loader/opener/FileInputStreamsOpener.java), and [HttpInputStreamsOpener](https://github.com/sjudd/glide/blob/master/library/src/com/bumptech/glide/loader/opener/HttpInputStreamsOpener.java). You can also implement your own.

The exact implementation of the ModelStreamLoader interface obviously depends on your data model and how you're planning on retrieving images, but since most people will probably be using http, something like this would probably work:

    public class UrlStreamLoader extends DirectModelStreamLoader<URL> {

        @Override
        protected String getId(URL yourUrl) { //The specific model is provided by the ImagePresenter
            return yourUrl.toString();
        }
       
        @Override
        protected StreamOpener getStreamOpener(URL yourUrl, int width, int height) {
            return new HttpInputStreamsOpener(yourUrl); //We could also use the width and height of the view    
                                                        //to download a particular size of the image
        }
    }

For a complete example see the [DirectFlickrStreamLoader](e/blob/master/samples/flickr/src/com/bumptech/flickr/DirectFlickrStreamLoader.java) in the sample project. 

The width and height are provided by the ImagePresenter. The url is also passed through by the ImagePresenter based on whatever its current model is (see the next section).

#### Create an ImagePresenter

Though Glide will work pretty much anywhere, it is optimized for use in ListViews or GridViews. Here is a simple example that ignores view recycling for simplicity's sake:

    @Override
    public View getView(int position, View recycled, ViewGroup container) { 
        ImageView imageView = new ImageView(MyActivity.this); 
        URL url = myUrls.get(position);

        ImagePresenter<URL> imagePresenter = new ImagePresenter.Builder<URL>()
            .setImageView(imageView)
            .setModelStreamLoader(new UrlStreamLoader()) //here is our ModelStreamLoader from step 2
            .setImageLoader(new CenterCrop(MyActivity.this.getImageManager())) //and our ImageManager from step 1
            .build();
        
        imageView.setTag(imagePresenter);
        imagePresenter.setModel(url); //here is where our model is passed to our ModelStreamLoader
    
        return imageView;
    } 

CenterCrop is one of a number of [ImageLoaders](https://github.com/sjudd/glide/tree/master/library/src/com/bumptech/glide/resize/loader) defined in the library. You can also implement the interface and define your own, with or without the ImageManager class.
        
If you were using view recycling, instead of a creating a new ImagePresenter, you would retrieve the ImagePresenter from the ImageView's tag and call setModel again with whatever your current URL is. Each time you call setModel, any in process loads are cancelled and a new one is started for the new model. 

You can call `setPlaceholderDrawable` or `setPlaceholderResourceId` on the ImagePresenter.Builder to set a drawable to show while a load is in progress. If you want to fade in your images or perform some other animation when a load finishes, you can call `setImageSetCallback` and provide a callback that will be called each time a load finishes.

For concrete examples of creating and using ImagePresenters see [FlickrPhotoList](https://github.com/sjudd/glide/blob/master/samples/flickr/src/com/bumptech/flickr/FlickrPhotoList.java) or [FlickrPhotoGrid](https://github.com/sjudd/glide/blob/master/samples/flickr/src/com/bumptech/flickr/FlickrPhotoGrid.java)

