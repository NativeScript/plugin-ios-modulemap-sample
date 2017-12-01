# Overview
This sample demonstrates how to use a manually generated `module.modulemap` file for the consumption of a native library within a plugin.

# Motivation
When you design a NativeScript plugin consuming a native iOS library for which you must create a `module.modulemap` you may encounter problems with consuming the native iOS types with JavaScript code. These problems boil down to several factors:

- you are using a version of NPM greater or equal to 5
- you are installing the plugin from a local folder, i.e. a cloned Github repository

When installing a plugin from a local folder with NPM 5 or later, the plugin source is added in the NativeScript app's `node_modules` folder as a symlink. Because of the symlink, upon build, when the NativeScript metadata generator iterates over the headers of the native libraries, the paths within your `module.modulemap` may not be resolved correctly thus preventing NativeScript from generating metadata for your native library. To prevent this from hapenning you need to move the affected `module.modulemap` outside of the symlinked folder and update the paths in it correspondingly. You will also need to update the `build.xcconfig` file describing the `HEADER_SEARCH_PATHS` for the native library to point to the new `module.modulemap` location.

This sample implements an automated approach for doing this by integrating NativeScript hooks into your plugin. These hooks will copy the `module.modulemap` file into a dedicated folder in the `platforms/ios` directory of your NativeScript app when the `tns prepare ios` command is executed (directly or indirectly).

# Dependencies
To work with NativeScript hooks you will need to install the `nativescript-hook` plugin and integrate it into your plugin as instructed here: https://github.com/NativeScript/nativescript-hook

# Implementation
The sample implements a scenario in which a NativeScript app references two plugins that use static native libraries coming from Pods. Both plugins define `module.modulemap` files to expose the native APIs of the corresponding libraries. A NativeScript hook is used to copy the module maps after the plugin is prepared to a location within the app's `platforms/ios` folder. 

## Sample source

```
const sourceMapLocation = '/node_modules/plugin1/platforms/ios/module.modulemap';
const targetMapLocation = '/platforms/ios/GVRSDK/module.modulemap';
const projectMapFolder = '/platforms/ios/GVRSDK/';
module.exports = function (logger, platformsData, projectData, hookArgs, $usbLiveSyncService) {
    var fs = require('fs');
    let targetMapFolder = projectData.projectDir + projectMapFolder;
    if (!fs.existsSync(targetMapFolder)) {
        fs.mkdirSync(targetMapFolder);
    }
    fs.copyFileSync(projectData.projectDir + sourceMapLocation, projectData.projectDir + targetMapLocation);
}
```

And here's a sample of how the `module.modulemap` will look like:

```
module GVRSDK {
    umbrella "../Pods/GVRSDK/Sources"
    export *
}
```
