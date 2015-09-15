## 静态资源

在项目的进程中，添加并且移除和处理那些在应用程序不是经常使用的图片是很常见的情况。为了解决这种情况，我们需要找到一个方法来静态地定位那些被用在应用程序里的图片。因此，我们使用了一个标记器。唯一允许的指向 bundle 里的图片的方法就是在源文件中遍历地搜索 `require('image!name-of-the-asset')` 。

```javascript
// GOOD
<Image source={require('image!my-icon')} />

// BAD
var icon = this.props.active ? 'my-icon-active' : 'my-icon-inactive';
<Image source={require('image!' + icon)} />

// GOOD
var icon = this.props.active ? require('image!my-icon-active') : require('image!my-icon-inactive');
<Image source={icon} />
```

当主要的代码执行到这里，你就可以做一些有趣的事情，比如自动将那些用于应用程序的 assets 打包。注意，这些代码不是强制实施的，但不代表将来也不会。

### 使用 Images.xcassets 将静态资源添加到你的 iOS 应用程序中

![](/react-native/img/StaticImageAssets.png)

> **NOTE**: 生成应用程序所需的新资源
>
> 无论在什么时候，您想把新的资源添加到 `Images.xcassets` 中，您都需要在使用它之前通过 Xcode 来重新构建您的应用程序 — — 在模拟器内重新加载它是不够的。

*这一进程正在被改进，不久就会提供更好的工作流程。*

### 将静态资源添加到您的 Android 应用程序中

将您的图像作为[位图画板](http://developer.android.com/guide/topics/resources/drawable-resource.html#Bitmap)添加到 android 项目中（`<yourapp>/android/app/src/main/res`）。 为了给您的 assets 文件提供不同的分辨率，使用 [配置限定符](http://developer.android.com/guide/practices/screens_support.html#qualifiers)进行检查。 通常情况下，您将想要把您的 assets 文件放在下列目录 (如果它们不存在，那么在 `res` 下创建它们)：

* `drawable-mdpi` (1x)
* `drawable-hdpi` (1.5x)
* `drawable-xhdpi` (2x)
* `drawable-xxhdpi` (3x)

如果您的 asset 文件丢失了一种分辨率，那么 Android 将采取下一个最好的分辨率并且为您调整它的大小。

> **NOTE**: 生成应用程序所需的新资源
>
> 无论在什么时候您把新的资源添加到您的画板中您都需要在您使用它之前通过运行 `react-native run-android` 重新构建您的应用程序 - 重新加载 JS 是不够的。

*这一进程正在被改进，不久就会提供更好的工作流程。*

## 网络资源

在您进行编译的时候，许多您的应用程序中需要展示的图片都不能使用，或者你会想要通过加载一些动态图片来保持二进制大小在较低的状态。不像静态资源那样，*您将需要手动指定图像的尺寸*。 

```javascript
// GOOD
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}} />
```

## 本地文件系统资源

请在 [CameraRoll](/react-native/docs/cameraroll.html) 中查看使用 `Images.xcassets` 之外的本地资源示例 。

### 最好的相片册图片

iOS 的相片册可以让你将同一张图片保存为不同的尺寸，对于选择那张接近你想要尺寸的图片来说，这很重要。你不会想用一张 3264x2448 分辨率的图片作为资源来显示一个 200x200 的缩略图。如果确实有符合你要求的尺寸， React Native 会自动选择它，否则它会使用第一张比特定尺寸大 50% 的图片来避免重新定义尺寸时带来的模糊失真。这些工作 React Native 自动帮你完成了，所以你不必再自己编写乏味和容易出错的代码。

## Why Not Automatically Size Everything?

*In the browser* if you don't give a size to an image, the browser is going to render a 0x0 element, download the image, and then render the image based with the correct size. The big issue with this behavior is that your UI is going to jump all around as images load, this makes for a very bad user experience.

*In React Native* this behavior is intentionally not implemented. It is more work for the developer to know the dimensions (or aspect ratio) of the remote image in advance, but we believe that it leads to a better user experience. Static images loaded from the app bundle via the `require('image!x')` syntax *can be automatically sized* because their dimensions are available immediately at the time of mounting.

For example, the result of `require('image!logo')` from the above screenshot:

```javascript
{"__packager_asset":true,"isStatic":true,"path":"/Users/react/HelloWorld/iOS/Images.xcassets/react.imageset/logo.png","uri":"logo","width":591,"height":573}
```

## Source as an object

In React Native, one interesting decision is that the `src` attribute is named `source` and doesn't take a string but an object with an `uri` attribute.

```javascript
<Image source={{uri: 'something.jpg'}} />
```

On the infrastructure side, the reason is that it allows us to attach metadata to this object. For example if you are using `require('image!icon')`, then we add an `isStatic` attribute to flag it as a local file (don't rely on this fact, it's likely to change in the future!). This is also future proofing, for example we may want to support sprites at some point, instead of outputting `{uri: ...}`, we can output `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}` and transparently support spriting on all the existing call sites.

On the user side, this lets you annotate the object with useful attributes such as the dimension of the image in order to compute the size it's going to be displayed in. Feel free to use it as your data structure to store more information about your image.

## Background Image via Nesting

A common feature request from developers familiar with the web is `background-image`. To handle this use case, simply create a normal `<Image>` component and add whatever children to it you would like to layer on top of it.

```javascript
return (
  <Image source={...}>
    <Text>Inside</Text>
  </Image>
);
```

## Off-thread Decoding

Image decoding can take more than a frame-worth of time. This is one of the major source of frame drops on the web because decoding is done in the main thread. In React Native, image decoding is done in a different thread. In practice, you already need to handle the case when the image is not downloaded yet, so displaying the placeholder for a few more frames while it is decoding does not require any code change.
