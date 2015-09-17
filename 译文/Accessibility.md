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

```android
<View accessible={true}>
  <Text>text one</Text>
  <Text >text two</Text>
</View>
```

在上面的示例中，我们不能分别在 'text one' 和 'text two' 中获得辅助焦点。相反我们可以在父元素上使用 'accessible'  属性获得焦点。

#### accessibilityLabel (iOS, Android)

将视图标记为具有辅助性，那么一个比较好的做法就是为这个视图设置一个 accessibilityLabel 标签以便使用 VoiceOver 的人知道他们选择了什么元素。当用户选择了一些元素，那么 VoiceOver 将会阅读响应的字符串文本。

若要使用它，在您的视图中将 accessibilityLabel 属性设置为一个自定义的字符串：

```android
<TouchableOpacity accessible={true} accessibilityLabel={'Tap me!'} onPress={this._onPress}>
  <View style={styles.button}>
    <Text style={styles.buttonText}>Press me!</Text>
  </View>
</TouchableOpacity>
```
