# Introduction

## What are integration libraries?
Glide includes a number of small and optional integration libraries that allow Glide to interface with external libraries. Currently Glide includes integration libraries for performing all http/https operations with [Volley][1] and [OkHttp][2].

## Why include integration libraries?
We strongly believe that your choice of a client side media library should neither dictate the networking library you use in your app, nor require you to include an additional networking library used only to load images. The integration libraries and Glide's [`ModelLoader`][4] system allow developers to use a consistent platform for all networking operations across their app.

## Why isn't xxx library implemented?
Because you haven't written the integration library for it yet! OkHttp and Volley are popular libraries that many developers will find useful, but we certainly don't mean to exclude any other library. If you've written a [`ModelLoader`][4] for another library for your app and want to open source it, we'd love to see pull requests to support additional libraries.

## How do I depend on an integration library?
There are two parts to depending on any integration library:

 1. Include the corresponding Maven, Gradle, or jar dependency, because the optional integration libraries are not included in the `glide` jar dependency.
 2. Make sure the app includes the integration library's [GlideModule][5].
 For more information on GlideModules, see the [Configuration][6] wiki page.
 For specific instructions for each of Glide's current integration libraries, see below.

## Which version should I pick?
The integration libraries are versioned together with the Glide releases, but follow different numbering. Make sure you pick the version that corresponds to your Glide dependency version. Check the Downloads on [releases page][7] for versions.

The networking libraries have yet another versioning and the integration libraries depend on those versions via transitive Maven/Gradle dependencies when used through Maven Central/JCenter, you can override the version by explicitly including the networking library dependency.

# Volley
[Volley][1] is an HTTP library that makes networking for Android apps easier and most importantly, faster.

## Volley with Gradle

```gradle
dependencies {
    compile 'com.github.bumptech.glide:volley-integration:1.3.1@aar'
    //compile 'com.mcxiaoke.volley:library:1.0.8'
}
```

The integration library's `GlideModule` will be merged into your application's manifest automatically.

## Volley with Maven

```xml
<dependency>
    <groupId>com.github.bumptech.glide</groupId>
    <artifactId>volley-integration</artifactId>
    <version>1.3.1</version>
    <type>aar</type>
</dependency>
<dependency>
    <groupId>com.mcxiaoke.volley</groupId>
    <artifactId>library</artifactId>
    <version>1.0.8</version>
    <type>aar</type>
</dependency>
```
See the [corresponding manifest section](#volley-manifest) to include the integration library's `GlideModule`.

## Volley Manually
Download the [glide-volley-integration-\<version\>.jar][8] from the [releases page][7] and add it to your Android app's compile classpath.

See the [corresponding manifest section](#volley-manifest) to include the integration library's `GlideModule`.

## Volley Manifest
If you build with Maven, Ant, or any build system that does not support manifest merging, you must manually add the `GlideModule` metadata tag in your `AndroidManifest.xml`:

```xml
<meta-data
    android:name="com.bumptech.glide.integration.volley.VolleyGlideModule"
    android:value="GlideModule" />
```

## Volley Proguard

Regardless of the build system you use, you need to make sure proguard doesn't obfuscate or strip the `VolleyGlideModule` class so it can be instantiated using reflection. Add the following to your `proguard.cfg` file (or see the [generic section](#generic-proguard)):

```pro
-keep class com.bumptech.glide.integration.volley.VolleyGlideModule
```

# OkHttp
[OkHttp][2] is an HTTP client that’s efficient by default, perseveres when the network is troublesome and is easy to use.

## OkHttp with Gradle

```gradle
dependencies {
    compile 'com.github.bumptech.glide:okhttp-integration:1.3.1@aar'
    //compile 'com.squareup.okhttp:okhttp:2.2.0'
}
```

The integration library's `GlideModule` will be merged into your application's manifest automatically.

## OkHttp with Maven

```xml
<dependency>
    <groupId>com.github.bumptech.glide</groupId>
    <artifactId>okhttp-integration</artifactId>
    <version>1.3.1</version>
    <type>aar</type>
</dependency>
<!--
<dependency>
    <groupId>com.squareup.okhttp</groupId>
    <artifactId>okhttp</artifactId>
    <version>2.2.0</version>
    <type>jar</type>
</dependency>
-->
```
See the [corresponding manifest section](#okhttp-manifest) to include the integration library's `GlideModule`.

## OkHttp Manually
Download the [glide-okhttp-integration-\<version\>.jar][9] from the [releases page][7] and add it to your Android app's compile classpath.

See the [corresponding manifest section](#okhttp-manifest) to include the integration library's `GlideModule`.

## OkHttp Manifest
If you build with Maven, Ant, or any build system that does not support manifest merging, you must manually add the `GlideModule` metadata tag in your `AndroidManifest.xml`:

```xml
<meta-data
    android:name="com.bumptech.glide.integration.okhttp.OkHttpGlideModule"
    android:value="GlideModule" />
```

## OkHttp Proguard

Regardless of the build system you use, you need to make sure proguard doesn't obfuscate or strip the `VolleyGlideModule` class so it can be instantiated using reflection. Add the following to your `proguard.cfg` file (or see the [generic section](#generic-proguard)):

```pro
-keep class com.bumptech.glide.integration.okhttp.OkHttpGlideModule
```

# More Options

## Generic Proguard
You have an alternative to keep all possible `GlideModule` modules:

```pro
-keep public class * implements com.bumptech.glide.module.GlideModule
```

This has the added benefit that it doesn't need to change if/when you change integration libraries, or customize the integration library's behavior. It also includes any additional modules which you may add later and portable between projects.

## Overriding default behaviour
All integration libraries have additional options if you're not satisfied with the defaults. For example adding retry-behaviour. See the source code of the integration libraries' `GlideModule`s at [`/integration/<lib>/src/main/java/<package>`][10] for how the default registration is done. You can then modify the behavior by changing the arguments for the `UrlLoader.Factory` class in your own custom `GlideModule`.

When overriding default behaviour make sure your custom `GlideModule` is registered in the manifest and the default one is excluded. Exclusion may mean removing the corresponding metadata from the manifest or using the jar dependency instead of the aar one. For more info on `GlideModule`s see the [Configuration][6] wiki page.

[1]: http://developer.android.com/training/volley/index.html
[2]: http://square.github.io/okhttp/
[3]: http://developer.android.com/reference/java/net/HttpURLConnection.html
[4]: http://bumptech.github.io/glide/javadocs/latest/com/bumptech/glide/load/model/ModelLoader.html
[5]: https://github.com/bumptech/glide/blob/master/library/src/main/java/com/bumptech/glide/module/GlideModule.java
[6]: https://github.com/bumptech/glide/wiki/Configuration
[7]: https://github.com/bumptech/glide/releases
[8]: https://github.com/bumptech/glide/releases/download/v3.6.1/glide-volley-integration-1.3.1.jar
[9]: https://github.com/bumptech/glide/releases/download/v3.6.1/glide-okhttp-integration-1.3.1.jar
[10]: https://github.com/bumptech/glide/tree/3.0/integration