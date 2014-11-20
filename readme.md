# Lets Convert Some Images!
It is very common for modern web applications to handle image storage, conversion, and hosting. To reduce bandwidth use, applcations convert uploaded images into various sizes, quality levels, and formats. This can be very processor intensive and limiting as the applications are only able to get the image formats that they have already generated (and genarating new images can take forever for large image sets).

### This Solution!
This tool is designed to sit between an application's file store and its content distribution network (CDN). It works by setting your CDN's origin server to this tool. So when an image is requested from your CDN it request the image from this tool, this tool then grabs the original image from your file store, processes it (resize, quality, format conversion, etc...), and send it back to the CDN. Now whenever that image is requested, it will simply respond with the image stored in the CDN (and won't re-process it).

### Cool! What all can this tool do?
This tool is built on top of ImageMagick and RMagick. So pretty much everything ImageMagick can do, this tool can do to (see a [full list here](https://github.com/KenStipek/image-converter/blob/readme/methods.md), and the [RMagick API docs here](http://www.imagemagick.org/RMagick/doc/))!

### What does a request look like?
Lets say you have an images stored at http://example.com/images/ and you wanted to get an image called be-cool.jpg, remove some of the colors, make it look like an oil painting, and convert it to a PNG. Assuming you have your CDN set up at cdn.example.com and that CDN's origin server is pointing at this tool all you would need to do is include the ImageMagick methods and attributes in the path before the path split character.
```
http://cdn.example.com/quantize/70/oil_paint/~/_/images/be-cool.png
```
The tool ignores the file extension and converts whatever image it finds with the given name to the requested extension.

You don't have to put the methods in path if you don't want to. You can set up defaults and templates using your server's environmental variables.

###### Original Image
![original image](https://limitless-island-6276.herokuapp.com/_/be-cool.jpg)

###### Converted Image
![converted image](https://limitless-island-6276.herokuapp.com/quantize/70/oil_paint/~/_/be-cool.png)

### Why use the path and not GET params for the methods and attributes?
Because some CDN's ignore everything after the '?' in a request and wont include them in the request to the origin server.

# Getting Started
1. Fork this Repo
2. Download and set up the Sinatra app (see this if need help with this)
3. Install ImageMagick ```brew install imagemagick``` on OSX using Homebrew, more info here
4. Install the Gems using ```bundle install```
5. Start playing with it on your local system and figure out what settings you need
6. Deploy and start rocking (don't forget to change your CDN's origin server)

If you do not provide an origin server the app will default to pulling images from the /images folder.

## Set Up

All the configuration for the app takes place in the servers environmental variables (for simplity).

###### Available Settings
```FILE_DOMAIN```
Defaults to /images folder on the local server.

```USE_PARAMS```
Whether or not to allow paramater defined methods to be used. Default is true.

```DEFAULTS```
Default methods to used. They are defined the same way params in your env string (e.g. ```DEFAULTS='resize_to_fit/100/oil_paint/~'```). Defualts override the whitelist.

```USE_DEFAULTS```
Whether or not to use any set DEFAULTS.

```TEMPLATES```
You can use templates like defaults, but you can have multiple of them. They are set the same way as params, but split up using '&'. Here is an example of setting up two templates named 'th' and 'oil' respectfully:
```
TEMPLATES='th&thumbnail/50/100&oil&oil_paint/~'
```
Templates do not override the whitelist like defaults.

You use a template by including its name in the path. Example:
```
http://cdn.example.com/template/th/_/images/be-cool.png
```
Would load the image using the 'th' template.

```USE_TEMPLATES```
Whether or not to use templates, defaults is true.

```TEMPLATE_MARK```
You can set your own template mark, by default this is 'template' but can be switched to anything you want. So lets say you set this to 't' the the above path would be:
```
http://cdn.example.com/t/th/_/images/be-cool.png
```
Rather then
```
http://cdn.example.com/template/th/_/images/be-cool.png
````

```SPLIT_CHAR```
The path is split between the methods (or templates) used and the actual path of the image. By default this is split using '_' but you can set what ever you want. Lets say you set SPLIT_CHAR to 'icons' then the path would look like this:
```
http://cdn.example.com/t/th/icons/images/be-cool.png
```

```WHITELIST```
You can set a whitelist of methods that are allowed to by run against your server. They are simply comma seperated.

```DEBUG```
The server prints image info to std_out by default, you can turn this off by setting this to false.

## Sandbox
You can experiment with using paramaters using my sandbox server at https://limitless-island-6276.herokuapp.com. Here are the settings on it:

```
DEFAULTS:  resize_to_fit/300
TEMPLATES: simple&gaussian_blur/~/quantize/60&flow&wave/~/oil_paint/~
WHITELIST: gaussian_blur,wave,sharpen,oil_paint,quantize
```
https://limitless-island-6276.herokuapp.com/be-cool.jpg

https://limitless-island-6276.herokuapp.com/template/simple/_/be-cool.jpg

https://limitless-island-6276.herokuapp.com/template/flow/_/be-cool.jpg

https://limitless-island-6276.herokuapp.com/sharpen/~/_/be-cool.jpg

https://limitless-island-6276.herokuapp.com/gaussian_blur/~/quantize/100/_/be-cool.jpg

