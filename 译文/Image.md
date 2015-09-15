## 静态资源

在项目的进程中，添加并且移除和处理那些在应用程序不是经常使用的图片是很常见的情况。为了处理这种情况，我们需要找到一个方法来静态地定位那些被用在应用程序里的图片。因此，我们使用了一个标记器。唯一允许的指向 bundle 里的图片的方法就是在源文件中遍历地搜索 `require('image!name-of-the-asset')` 。

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
> 无论在什么时候，您想把新的资源添加到 `Images.xcassets` 中，您都需要在使用它之前通过 Xcode 来重新构建您的应用程序 — — 仅在模拟器内重新加载它是不够的。

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
> 无论在什么时候您把新的资源添加到您的画板中您都需要在使用它之前通过运行 `react-native run-android` 重新构建您的应用程序 - 仅重新加载 JS 是不够的。

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

## 为什么不自动调整所有事物？

*在浏览器中* 如果你不给一个图像规定大小，那么浏览器会呈现一个 0x0 的元素、 随后下载图像，然后再以正确的尺寸呈现图像。这种行为最大的问题是，您的 UI 会在正在加载的图像四周跳动，这会造成一个非常糟糕的用户体验。

在 *React Native* 中故意不实施该行为。它的目的是让开发人员可以提前知道远程图像的尺寸 (或图形比例)，我们认为这样做的话可以实现更好的用户体验。通过使用 `require('image!x')` 语法从应用程序包中加载的静态图片可以自动的调整大小，因为它们的尺寸在安装时立即可用。

例如，上述使用 'require('image!logo')'  屏幕截图的结果：

```javascript
{"__packager_asset":true,"isStatic":true,"path":"/Users/react/HelloWorld/iOS/Images.xcassets/react.imageset/logo.png","uri":"logo","width":591,"height":573}
```

## Source 是一个对象类型

在 React Native 中，一个有趣的决定是 `src` 特性将会被命名为 `source`，并且不作为一个字符串而是一个 `uri` 特性的对象类型。

```javascript
<Image source={{uri: 'something.jpg'}} />
```

站在底层来看，这样做的原因是它允许将元数据依附到这个对象中。举个例子，你正在使用 `require('image!icon')`，我们将添加 `isStatic` 作为一个 flag 来标识本地文件（不要依赖这例子，将来这可能会改变！）。这在将来同时也会成为可能，比如我们可能会支持子画面，并用它来取代输出 `{uri: ...}`，我们可以输出 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}` 同时支持在所有已经存在的网站中透明地显示子画面。

在用户角度上，这会让你用有用的特性比如图片的几何尺寸来注释对象类型，从而计算出将要显示出来的尺寸。尽情地使用这种数据类型来储存你的图片吧。

## 背景图片叠加

一个对于 web 开发者们很常见的需求是 `background-image`。这种情况下，创建一个简单的 `<Image>` 组件然后将它作为子 layer 添加到你想要添加的 layer 上面。

```javascript
return (
  <Image source={...}>
    <Text>Inside</Text>
  </Image>
);
```

## 非主线程加载

图片的解析会花费很多的时间。这是导致网页的帧数下降的其中一个重要的原因，因为解析工作会被执行在主线程中。在 React Native 中，图片的解析会在不同的线程中执行。在实际操作中，你已经处理好这种情况，当图片还没有下载完成，因此需要将 placeholder 显示出来，这不用你写任何代码。
