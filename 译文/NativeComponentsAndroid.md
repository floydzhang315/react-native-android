

这里有很多原生的 UI 部件准备被用到最新的应用程序中 - 其中一些是平台的一部分，其他的部分可以作为第三方库来使用，而且仍然还有更多的部分可能是在你自己的投资组合中使用。React Native 已经将几个最关键的平台组件进行了打包，如同 `ScrollView` 和 `TextInput`，但是并不是所有都被打包了，所以当然也不可能是您以前写的应用程序。幸运的是，通过使用 React Native 应用程序可以很容易的将现有的组件进行无缝集成打包。

就如同原生模块指南，这是一个建立在假定你对 Android SDK 编程有些熟悉的基础上的更高级的指南。本指南将显示你该如何构建一个原生的 UI 组件， 帮助你遍历执行可用核心 `React Native` 库中可以使用的现有的 ImageViewcomponent 的一个子集。

## ImageView 示例

在本例中我们将要完全了解实施要求来实现在 JavaScript 中允许使用 ImageViews。

原生视图是由扩展 `ViewManage` 或者更普遍的 `SimpleViewManager` 所创建和操纵的。在这种情况下 `SimpleViewManager` 是很方便的，因为它适用于普遍的属性，比如背景颜色、 不透明度和 Flexbox 布局。当然也有其他例子,当您在使用 FrameLayout 进行包装组件的时候，那么这时候您需要使用 `ViewManage` ，比如 ProgressBar。

这些子类在本质上是很单一的 — — 每个子类之中只有一个实例是通过这个桥接器创建的。他们将原生视图传递到了 `NativeViewHierarchyManager` 之中，这代表回到了通过使用它们原始的方法来设置并更新这些必要的视图的属性。`ViewManagers` 通常也是这些视图的代表，它通过该桥接器将事件发送回 JavaScript。

传递一个视图很简单：

1.创建 ViewManager 子类  
2.使用  `@UIProp` 注释视图属性  
3.实现 `createViewInstance` 方法  
4.实现 `updateView` 方法  
5.在应用程序软件包中的 `createViewManagers` 中注册管理器  
6.执行 JavaScript 模块  

## 1. 创建 `ViewManager` 子类

在本示例中，我们通过继承 `ReactImageView` 类型的 `SimpleViewManager` 来创建的视图管理器类 `ReactImageManager`。它是由管理器管理的对象类型，这将成为一个原生视图。通过 `getName` 返回的名字将被用来从 Javascript 中引用原生视图类型。

```java
...

public class ReactImageManager extends SimpleViewManager<ReactImageView> {

  public static final String REACT_CLASS = "RCTImageView";

  @Override
  public String getName() {
    return REACT_CLASS;
  }
```

## 2. 注释视图属性

我们在 JavaScript 中使用 `@UIProp` 来注释需要被反映出来的属性。目前支持的类型有 `BOOLEAN`, `NUMBER`, `STRING`, `MAP` 和 `ARRAY`。每个属性都被声明为公共静态最终字符串常量，并且给它们分配的值在 JavaScript 中都会成为属性的名称。

```java
  @UIProp(UIProp.Type.STRING)
  public static final String PROP_SRC = "src";
  @UIProp(UIProp.Type.NUMBER)
  public static final String PROP_BORDER_RADIUS = "borderRadius";
  @UIProp(UIProp.Type.STRING)
  public static final String PROP_RESIZE_MODE = ViewProps.RESIZE_MODE;
```

## 3. 实现 `createViewInstance` 方法  

我们使用 `CreateViewInstance` 方法来创建视图，视图应将其自身初始化到默认状态，然后任何属性都会通过后续调用 `updateView` 来进行设置。

```java
  @Override
  public ReactImageView createViewInstance(ThemedReactContext context) {
    return new ReactImageView(context, Fresco.newDraweeControllerBuilder(), mCallerContext);
  }
```

## 4. 实现 `updateView` 方法 

和 iOS 中有些不同的是在 Android 中，不是通过自动调用 setter 方法来给一个视图的属性进行赋值； 对于 Android 而言，你需要通过您的 `ViewManager` 中的 `updateView` 方法手动调用 setter。从 `CatalystStylesDiffMap` 中提取出来值，并且传递给视图实例。它是通过 `updateView` 和视图类的组合来检查属性的有效性，并采取相应的行动。

```java
  @Override
  public void updateView(final ReactImageView view,
                         final CatalystStylesDiffMap props) {
    super.updateView(view, props);

    if (props.hasKey(PROP_RESIZE_MODE)) {
      view.setScaleType(
        ImageResizeMode.toScaleType(props.getString(PROP_RESIZE_MODE)));
    }
    if (props.hasKey(PROP_SRC)) {
       view.setSource(props.getString(PROP_SRC));
    }
    if (props.hasKey(PROP_BORDER_RADIUS)) {
      view.setBorderRadius(props.getFloat(PROP_BORDER_RADIUS, 0.0f));
    }
    view.maybeUpdateView();
  }
}
```

## 5. Register the `ViewManager`

The final Java step is to register the ViewManager to the application, this happens in a similar way to [Native Modules](NativeModulesAndroid.md), via the applications package member function `createViewManagers.`

```java
  @Override
  public List<ViewManager> createViewManagers(
                            ReactApplicationContext reactContext) {
    return Arrays.<ViewManager>asList(
      new ReactImageManager()
    );
  }
```

## 6. Implement the JavaScript module

The very final step is to create the JavaScript module that defines the interface layer between Java and JavaScript for the users of your new view. Much of the effort is handled by internal React code in Java and JavaScript and all that is left for you is to describe the `propTypes`.

```js
// ImageView.js

var { requireNativeComponent } = require('react-native');

var iface = {
  name: 'ImageView',
  propTypes: {
    src: PropTypes.string,
    borderRadius: PropTypes.number,
    resizeMode: PropTypes.oneOf(['cover', 'contain', 'stretch']),
  },
};

module.exports = requireNativeComponent('RCTImageView', iface);
```

`requireNativeComponent` commonly takes two parameters, the first is the name of the native view and the second is an object that describes the component interface. The component interface should declare a friendly `name` for use in debug messages and must declare the `propTypes` reflected by the Native View. The `propTypes` are used for checking the validity of a user's use of the native view.

# Events

So now we know how to expose native view components that we can control easily from JS, but how do we deal with events from the user, like pinch-zooms or panning? When a native event occurs the native code should issue an event to the JavaScript representation of the View, and the two views are linked with the value returned from the `getId()` method.

```java
class MyCustomView extends View {
   ...
   public void onReceiveNativeEvent() {
      WritableMap event = Arguments.createMap();
      event.putString("message", "MyMessage");
      ReactContext reactContext = (ReactContext)getContext();
      reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(
          getId(),
          "topChange",
          event);
    }
}
```

The event name `topChange` maps to the `onChange` callback prop in JavaScript (mappings are in `UIManagerModuleConstants.java`). This callback is invoked with the raw event, which we typically process in the wrapper component to make a simpler API:

```js
// MyCustomView.js

class MyCustomView extends React.Component {
  constructor() {
    this._onChange = this._onChange.bind(this);
  }
  _onChange(event: Event) {
    if (!this.props.onChange) {
      return;
    }
    this.props.onChange(event.nativeEvent.message);
  }
  render() {
    return <RCTMyCustomView {...this.props} onChange={this._onChange} />;
  }
}
MyCustomView.propTypes = {
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onChange: React.PropTypes.func,
  ...
};
```
