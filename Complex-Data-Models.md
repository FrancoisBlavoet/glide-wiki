#### Complex data models

Using Glide with a data model more complex than a simple File or URL requires implementing the [ModelStreamLoader](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/model/ModelStreamLoader.java) interface.

#### DirectStreamLoader (synchronous)

Assuming you don't need to perform a long running task to prepare your model, the simplest thing to do is extend [DirectModelStreamLoader] (https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/model/DirectModelStreamLoader.java). Doing so requires you to implement two methods, `getStreamOpener()` and `getId()`. 

`getId()` just requires you to return a String that is unique to the image the model represents irrespective of size. An ID field in a database, a file name, or a url would all work well. 

`getStreamOpener()` is a little more complex and requires you to return a [StreamOpener](https://github.com/bumptech/glide/blob/master/library/src/com/bumptech/glide/loader/opener/StreamOpener.java). Essentially a StreamOpener is a way of lazily opening an InputStream to the image represented by the model so that if the image is cached, we don't do something like make an http call unnecessarily.

Although it is not safe to perform long running tasks in either `getId()` or `getStreamOpener()` it is safe to do so inside the StreamOpener you return from `getStreamOpener()` since that will always be called on a background thread.

Here is a simple example DirectStreamLoader that returns a different path depending on the size of the view the image will be loaded into:

    public class SizedFileStreamLoader extends DirectModelStreamLoader<String> {  
        private final File imagesDir;
        public SizedFileStreamLoader(File imagesDir) {
            this.imagesDir = imagesDir;
        }

        @Override
        protected String getId(String fileName) {
            return fileName; //Assumes fileName is unique in the data set
        }

        @Override
        protected StreamOpener getStreamOpener(String fileName, int width, int height) {
            int determiningDimen = Math.max(width, height);
            final String subDir;
            if (determiningDimen > 1000) {
                subDir = "large";
            } else if (determiningDimen > 500) {
                subDir = "medium";
            } else {
                subDir = "small";
            }
            return new FileInputStreamOpener(new File(imagesDir, subDir + File.separator + fileName));
        }
    }
    
For a slightly more complex example, see the [DirectFlickrStreamLoader](https://github.com/bumptech/glide/blob/master/samples/flickr/src/com/bumptech/flickr/DirectFlickrStreamLoader.java) in the Flickr sample.