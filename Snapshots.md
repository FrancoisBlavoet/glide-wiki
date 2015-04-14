## About Snapshots
For users who don't want to wait for the next version of Glide and are willing to live on the bleeding edge, we deploy snapshot versions of the library to [Sonatype's snapshot repo][2]. 

After each push to the master branch on GitHub, Glide is built by [travis-ci][1]. If the build succeeds, we automatically deploy the latest version of the library to Sonatype.

Each integration library will have its own snapshot, as will the main Glide library. If you use a snapshot version of the Glide library you must also use the snapshot versions of any integration libraries you use as well, and vice versa. 

## Obtaining Snapshots
Sonatype's snapshot repo functions as any other maven repo would, so snapshots are accessible as a jar, in maven, or in gradle.

### Jar
Jars can be downloaded [directly from Sonatype][3]. Double check the date to make sure you're getting the latest version. 

### Gradle

Add the snapshot repo to your list of repositories:

```groovy
repositories {
  jcenter()
  maven {
    url 'http://oss.sonatype.org/content/repositories/snapshots'
  }
}
```

And then change your dependencies to the snapshot version:

```groovy
dependencies {
  compile "com.github.bumptech.glide:glide:3.6.0-SNAPSHOT"
  compile "com.github.bumptech.glide:okhttp-integration:1.3.0-SNAPSHOT"
}
```

### Maven

Add the following to your ``~/.m2/settings.xml``:

```xml
<profiles>
  <profile>
     <id>allow-snapshots</id>
     <activation><activeByDefault>true</activeByDefault></activation>
     <repositories>
       <repository>
         <id>snapshots-repo</id>
         <url>https://oss.sonatype.org/content/repositories/snapshots</url>
         <releases><enabled>false</enabled></releases>
         <snapshots><enabled>true</enabled></snapshots>
       </repository>
     </repositories>
   </profile>
</profiles>
```

Then change your dependencies to the snapshot version:

```xml
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>glide</artifactId>
  <version>3.6.0-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.github.bumptech.glide</groupId>
  <artifactId>okhttp-integration</artifactId>
  <version>1.3.0-SNAPSHOT</version>
</dependency>
```

[1]: https://oss.sonatype.org/content/repositories/snapshots/
[2]: https://travis-ci.org/bumptech/glide
[3]: https://oss.sonatype.org/content/repositories/snapshots/com/github/bumptech/glide/
