## What are integration libraries?
Glide includes a number of small and optional integration libraries that allow Glide to interface with external libraries. Currently Glide includes integration libraries for performing all http/https operations with [Volley][1] and [OkHttp][2].

## Why include integration libraries?
We strongly believe that your choice of a client side media library should neither dictate the networking library you use in your app, nor require you to include an additional networking library used only to load images. The integration libraries and Glide's [ModelLoader][4] system allow developers to use a consistent platform for all networking operations across their app.

## Why isn't xxx library included?
Because you haven't written the integration library for it yet! OkHttp and Volley are popular libraries that many developers will find useful, but we certainly don't mean to exclude any other library. If you've written a [ModelLoader][4] for another library for your app and want to open source it, we'd love to see pull requests to support additional libraries.

## How do I depend on an integration library?
There are two parts to depending on any integration library. First you need to include the corresponding Maven, Gradle, or jar dependency because the optional integration libraries are not included by default.

Second, you need to make sure your app includes the integration library's [GlideModule][5]. For more information on GlideModules, see the [Configuration][6] wiki page. For specific instructions for each of Glide's current integration libraries, see below.

## Using Volley
1. To use Volley, you need to first include the corresponding dependencies.

    With Gradle:
    ```groovy
    dependencies {
        compile 'com.github.bumptech.glide:volley-integration:1.2.2'
        compile 'com.mcxiaoke.volley:library:1.0.5'
    }
    ```

    With Maven:
    ```xml
    <dependency>
        <groupId>com.github.bumptech.glide</groupId>
        <artifactId>volley-integration</artifactId>
        <version>1.2.2</version>
        <type>aar</type>
    </dependency>
    <dependency>
        <groupId>com.mcxiaoke.volley</groupId>
        <artifactId>library</artifactId>
        <version>1.0.5</version>
        <type>aar</type>
    </dependency>
    ```

    Or download a jar from the [releases page][7].

2. Include the Volley integration library's GlideModule.
    1. Include the GlideModule metadata tag in your ``AndroidManifest.xml``.
        
        If you build with **Gradle** or any other build system that supports manifest merging and depend on the aar version of the Volley integration library, the integration library's GlideModule will be merged into your application's manifest automatically.

        If you build with Maven, Ant, or any build system that does not support manifest merging, you must manually add a metadata tag to your manifest:
        ```xml
        <meta-data
            android:name="com.bumptech.glide.integration.volley.VolleyGlideModule"
            android:value="GlideModule" />
        ```
    2. Add a proguard keep for the VolleyGlideModule class.
        
        Regardless of the build system you use, you need to make sure proguard doesn't obfuscate or strip the VolleyGlideModule class so it can be instantiated using reflection. Add one of the following to your proguard.cfg file:
        ```
        -keep class com.bumptech.glide.integration.volley.VolleyGlideModule
        #or
        -keep public class * implements com.bumptech.glide.module.GlideModule
        ```

## OkHttp

1. To use OkHttp, you need to first include the corresponding dependencies.

    With Gradle:
    ```groovy
    dependencies {
        compile 'com.github.bumptech.glide:okhttp-integration:1.2.2'
        compile 'com.squareup.okhttp:okhttp:2.0.0'
    }
    ```

    With Maven:
    ```xml
    <dependency>
        <groupId>com.github.bumptech.glide</groupId>
        <artifactId>okhttp-integration</artifactId>
        <version>1.2.2</version>
        <type>aar</type>
    </dependency>
    <dependency>
        <groupId>com.squareup.okhttp</groupId>
        <artifactId>okhttp</artifactId>
        <version>2.0.0</version>
        <type>jar</type>
    </dependency>
    ```

    Or download a jar from the [releases page][7].

2. Include the OkHttp integration library's GlideModule.
    1. Include the GlideModule metadata tag in your ``AndroidManifest.xml``.
        
        If you build with **Gradle** or any other build system that supports manifest merging and depend on the aar version of the OkHttp integration library, the integration library's GlideModule will be merged into your application's manifest automatically.

        If you build with Maven, Ant, or any build system that does not support manifest merging, you must manually add a metadata tag to your manifest:
        ```xml
        <meta-data
            android:name="com.bumptech.glide.integration.okhttp.OkHttpGlideModule"
            android:value="GlideModule" />
        ```
    2. Add a proguard keep for the OkHttpGlideModule class.
        
        Regardless of the build system you use, you need to make sure proguard doesn't obfuscate or strip the VolleyGlideModule class so it can be instantiated using reflection. Add one the following to your proguard.cfg file:
        ```
        -keep class com.bumptech.glide.integration.okhttp.OkHttpGlideModule
        #or
        -keep public class * implements com.bumptech.glide.module.GlideModule
        ```

[1]: http://developer.android.com/training/volley/index.html
[2]: http://square.github.io/okhttp/
[3]: http://developer.android.com/reference/java/net/HttpURLConnection.html
[4]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/model/ModelLoader.html
[5]: https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/module/GlideModule.java
[6]: https://github.com/bumptech/glide/wiki/Configuration
[7]: https://github.com/bumptech/glide/releases