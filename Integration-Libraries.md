## What are integration libraries?
Glide includes a number of small and optional integration libraries that allow Glide to interface with external libraries. Currently Glide includes integration libraries for performing all http/https operations with [Volley][1] and [OkHttp][2].

## Why include integration libraries?
We strongly believe that your choice of a client side media library should neither dictate the networking library you use in your app, nor require you to include an additional networking library used only to load images. The integration libraries and Glide's [ModelLoader][4] system allow developers to use a consistent platform for all networking operations across their app.

## Why isn't xxx library included?
Because you haven't written the integration library for it yet! OkHttp and Volley are popular libraries that many developers will find useful, but we certainly don't mean to exclude any other library. If you've written a [ModelLoader][4] for another library for your app and want to open source it, we'd love to see pull requests to support additional libraries.

## How do I depend on an integration library?
There are two parts to depending on any integration library. First you need to include the corresponding Maven, Gradle, or jar dependency because the optional integration libraries are not included by default.

Second, you need to make sure your app includes the integration library's [GlideModule][5]. For more information on GlideModules, see the [Configuration][6] wiki page.

## OkHttp
Instructions coming soon...

## Volley
Instructions coming soon...

[1]: http://developer.android.com/training/volley/index.html
[2]: http://square.github.io/okhttp/
[3]: http://developer.android.com/reference/java/net/HttpURLConnection.html
[4]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/model/ModelLoader.html
[5]: https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/module/GlideModule.java
[6]: https://github.com/bumptech/glide/wiki/Configuration