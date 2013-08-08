### Download
You can simply download the jar for the latest release on the [releases page](https://github.com/bumptech/glide/releases).

### Command line

1. Either:
  1. run: git clone git@github.com:bumptech/glide.git
  2. or, add Glide as a submodule to your project: git submodule add git@github.com:bumptech/glide.git glide
2. cd glide/library && make jar (or glide-minus-volley for a jar that doesn't include volley)
3. Copy the jar from from library/bin to your project's libs folder

### Library project

Although neither Glide nor Volley is currently a library project, both can easily be converted if you can simply add android.library=true in both project's project.properties files and then use [the command line tools](http://developer.android.com/tools/projects/projects-cmdline.html#UpdatingAProject) to add both projects as library projects. 
  