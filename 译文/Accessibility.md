# 辅助功能

## iOS

在 iOS 系统上辅助功能涵盖许多话题，但对许多人来说辅助功能是  VoiceOver 的代名词，即 iOS 3.0 版本以后的一种技术。它充当屏幕阅读器的角色，允许有视觉障碍的人使用 iOS 设备。点击[这里](https://developer.apple.com/accessibility/ios/)了解更多。

## Android 

对 Android 系统而言，辅助功能涉及到了许多不同的话题，其中之一是让丧失视力的人能够使用您的应用程序。对于当前社会，谷歌提供了一个名叫 TalkBack 的内置屏幕读者服务机器人。使用该机器人，你可以使用触摸勘探和手势来使用移动设备和应用程序。TalkBack 将会使用文本语音转换器来阅读屏幕上的内容并且可以发出警报来通知用户有关于应用程序中的重要信息。点击[这里](https://support.google.com/accessibility/android)来了解更多关于 Android 的辅助功能的特征以及点击[这里](https://developer.android.com/guide/topics/ui/accessibility)来了解更多关于使您的本地应用程序的辅助功能。

## 创建辅助性应用程序

### 辅助功能的性质

#### 辅助性（iOS, Android）

如果为 `true`，表示该视图是一个辅助功能元素。当视图死辅助功能元素时，它把它的子元素分组成一个单一的可选组件。默认情况下，可触摸的所有元素都具有辅助性。

在 Android 系统中，在 react-native  视图中 ' accessible={true}' 属性将被翻译成本地命令 ' focusable={true}'。

```java
<View accessible={true}>
  <Text>text one</Text>
  <Text >text two</Text>
</View>
```

在上面的示例中，我们不能分别在 'text one' 和 'text two' 中获得辅助焦点。相反我们可以在父元素上使用 'accessible'  属性获得焦点。

#### accessibilityLabel (iOS, Android)

将视图标记为具有辅助性，那么一个比较好的做法就是为这个视图设置一个 accessibilityLabel 标签以便使用 VoiceOver 的人知道他们选择了什么元素。当用户选择了一些元素，那么 VoiceOver 将会阅读响应的字符串文本。

若要使用它，在您的视图中将 accessibilityLabel 属性设置为一个自定义的字符串：

```java
<TouchableOpacity accessible={true} accessibilityLabel={'Tap me!'} onPress={this._onPress}>
  <View style={styles.button}>
    <Text style={styles.buttonText}>Press me!</Text>
  </View>
</TouchableOpacity>
```
在上面的示例中，TouchableOpacity 元素中的 `accessibilityLabel` 会被默认的设置为 "点击我！"。 该标签是通过使用空格符来串联所有文本节点子元素构造的。

#### accessibilityTraits (iOS) 

辅助功能特征告诉人们他们在使用 VoiceOver 的时候选择了什么元素。此元素是一个标签？一个按钮？还是标头？ `accessibilityTraits` 将会回答这些问题。

如果要使用它，请把 accessibilityTraits 属性设置为 accessibilityTraits 辅助功能字符串(或数组)之一：

- **none** 单元素没有特征的时候使用。
- **button** 当元素需要被当做一个按钮的时候使用。
- **link** 当元素需要被当做链接的时候使用。
- **header** 当元素作为内容部分的标题 (如导航栏中的标题) 的时候使用。
- **search** 当文本字段元素也被视为一个搜索字段的时候使用。
- **image** 当元素需要被作为图像，比如和按钮和链接结合的时候使用。
- **selected** 当该元素被选中时使用。例如，表中被选中的行或者分段控件中选中的按钮。
- **plays** 当元素被激活的并且播放自己的声音的时候使用。
- **key** 当元素充当键盘按键的时候使用。
- **text** 当元素应该被视为不能更改的静态文本的时候使用。
- **summary** 当在应用程序首次启动的时候，该元素可以提供应用程序的实时状况的摘要的时候使用。例如，当关于天气的应用程序首次启动的时候，带有当天天气信息的元素将被该特征所标记。
- **disabled** 当控件未启动并且对用户的输入无响应的时候使用。
- **frequentUpdates** Used when the element frequently updates its label or value, but too often to send notifications. Allows an accessibility client to poll for changes. A stopwatch would be an example.
- **startsMedia** Used when activating an element starts a media session (e.g. playing a movie, recording audio) that should not be interrupted by output from an assistive technology, like VoiceOver.
- **adjustable** Used when an element can be "adjusted" (e.g. a slider).
- **allowsDirectInteraction** Used when an element allows direct touch interaction for VoiceOver users (for example, a view representing a piano keyboard).
- **pageTurn** Informs VoiceOver that it should scroll to the next page when it finishes reading the contents of the element.
