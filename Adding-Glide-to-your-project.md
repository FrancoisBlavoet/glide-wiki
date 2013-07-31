### Command line

1. In your project directory, either:
  1. run: git clone git@github.com:bumptech/glide.git
  2. or, add Glide as a submodule: git submodule add git@github.com:bumptech/glide.git glide
2. cd glide/library && make setup

### Intellij

1. Follow the steps for the command line build first.
2. Open Intellij, right click on your root module, and click "Open Module Settings"
3. Click on the "+" symbol above the list of modules and select "Import Module"
4. Select glide/library and click "OK"
5. Make sure "Create module from existing sources" is selected and click "OK"
6. Click "Next"
7. Make sure libraries is checked and Click "Next", then "Next" twice more, and then "Finish"
8. In Module Settings, click on your main project module, and then click on the "+" button below the dependencies list, select "Module Dependency", and choose the new module (probably called "library").

  