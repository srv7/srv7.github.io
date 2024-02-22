---
layout: post
title: UITextView 添加链接点击并自定义事件  
subtitle: UITextView
date: 2018-04-16 09:58:26  
categories: iOS
---


![](http://ww1.sinaimg.cn/large/b92f96b9gy1fqeae0ygvnj20a90go0ul.jpg)

<!-- more -->

实现上图（iOS Setting App）中部分文字的效果。

## 使用 UITextView 实现

```
let plainText = "同意《用户协议》与《隐私保护》"
let attributedText = NSMutableAttributedString(string: plainText)
/// 添加蓝色可点击部分
attributedText.addAttributes([NSAttributedStringKey.link: Link.agreement.rawValue], range: NSMakeRange(2, 6))
attributedText.addAttributes([NSAttributedStringKey.link: URL(string: Link.private.rawValue)!], range: NSMakeRange(9, 6))
// attributedText.addAttributes([NSAttributedStringKey.link: Link.private.rawValue], range: NSMakeRange(9, 6))


textView.attributedText = attributedText
textView.isEditable = false
textView.isScrollEnabled = false
textView.showsHorizontalScrollIndicator = false
textView.showsVerticalScrollIndicator = false

/// 检测链接 !important
textView.dataDetectorTypes = .link

```

此时点击`《隐私保护》`会自动打开 Safari 加载`https://www.baidu.com`，点击`《用户协议》`控制台会输出:

```
Could not find any actions for URL agreement without any result.
```
长按链接部分则会弹出 ActionSheet，items 随 link 类型变化。

![](http://ww1.sinaimg.cn/large/b92f96b9ly1fqeb0cwifyj20aa0970t1.jpg)
![](http://ww1.sinaimg.cn/large/b92f96b9gy1fqeb0tjedcj20aa0ahq3d.jpg)

## 禁用 ActionSheet
禁用 ActionSheet 需要实现 `UITextViewDelegate` 的协议方法:

```
textView.delegate = self

@available(iOS 10.0, *)
optional public func textView(_ textView: UITextView, shouldInteractWith URL: URL, in characterRange: NSRange, interaction: UITextItemInteraction) -> Bool

@available(iOS, introduced: 7.0, deprecated: 10.0, message: "Use textView:shouldInteractWithURL:inRange:forInteractionType: instead")
optional public func textView(_ textView: UITextView, shouldInteractWith URL: URL, in characterRange: NSRange) -> Bool
```
返回 `false` 禁用 ActionSheet

## 自定义响应事件

对于可点击部分进行不同的操作，在代理方法`shouldInteractWith`中 `interaction` 的值是不同的。单击可以得到 `.invokeDefaultAction`,长按可以得到`.presentActions`。其类型都是`UITextItemInteraction`枚举，而`.preview`我没找到触发的方法。

现在我想达到下面的效果：
- 点击或长按`《隐私保护》`都打开 Safari 加载相应内容
- 点击`《用户协议》`跳转到其他 ViewController
- 长按`《用户协议》`在 `UIAlertController`中展示用户协议

代码如下：

```
func textView(_ textView: UITextView, shouldInteractWith URL: URL, in characterRange: NSRange, interaction: UITextItemInteraction) -> Bool {
        print(interaction.rawValue)
        
        switch URL.absoluteString {
        case Link.agreement.rawValue:
            let agreement = UIStoryboard.init(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "AgreementViewController")
            switch interaction {
            case .invokeDefaultAction:
                navigationController?.pushViewController(agreement, animated: true)
            case .presentActions:
                let alert = UIAlertController(title: "Agreement", message: nil, preferredStyle: .alert)
                let ok = UIAlertAction(title: "OK", style: .default, handler: nil)
                alert.addAction(ok)
                alert.set(contentVc: agreement)
                navigationController?.present(alert, animated: true, completion: nil)
            case .preview:
                break
            }
        case Link.private.rawValue:
            return true
        default:
            break
        }
        return false
    }
```
![](https://ws1.sinaimg.cn/large/b92f96b9gy1fqegp9pp30j20v91vo47r.jpg)
![](https://ws1.sinaimg.cn/large/b92f96b9ly1fqegttqcw3j20v91vodri.jpg)

以上就完成了不同响应事件的自定义。完整代码见 [github](https://github.com/srv7/TextViewLink)
