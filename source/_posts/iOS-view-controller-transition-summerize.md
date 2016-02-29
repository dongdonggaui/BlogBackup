---
title: iOS 中 viewController 的自定义 transition 动画效果总结
date: 2016-02-26 17:27:31
tags:
		- iOS
		- 动画
		- 总结
---

从 iOS7 开始，Apple 提供了一组 API 来帮助我们实现自定义的转场动画。如系统自带的“照片” App 中照片列表到照片详情的转场切换动画、“微信”中聊天界面里点击图片的转场切换动画效果、“支付宝”中付款时底部弹出的确认效果等等，都可以用这一组 API 来实现。这里总结一下完成一个自定义转场动画的主要步骤。

<!--more-->

## 主要 API

这些 API 主要包括三个协议：

- *UIViewControllerAnimatedTransitioning*：用来自定义转场动画；
- *UINavigationControllerDelegate*：用来告诉 viewController 转场需要的动画对象，并对转场过程做必要的控制，viewController 的转场通过 push 方式组织；
- *UIViewControllerTransitioningDelegate*：同上，只是 viewController 转场的组织方式是通过 present 的方式。

## 需要做的工作
自定义转场动画效果主要分为两个步骤

- 创建自定义动画
- 指定 viewController 转场需要的动画


### 创建自定义动画

通过创建自定义动画类，实现 *UIViewControllerAnimatedTransitioning* 协议来定义和管理转场动画。实现`transitionDuration:`和`animateTransition:`方法，在`animateTransition:`方法中通过`transitionContext`参数获取动画所需要的上下文信息，如`containView`、`fromVC`、`toVC`等，然后进行自定义动画的实现。

### 指定 viewController 转场需要的动画
在这一步，按 viewController 转场的组织方式分为两种类型：push 和 present。

#### 基于 push 的转场动画
此种转场需要实现 *UINavigationControllerDelegate* 协议的相关方法来指定和管理前面定义好的动画对象，大致步骤为：

- 实现 `navigationController:interactionControllerForAnimationController:` 指定可交互的 transition 动画控制器，若不需要可交互返回`nil`即可；
- 实现 `navigationController:didShowViewController:animated:` 做一些必要的额外工作，如更新 statusBar 样式等；
- 若支持交互，建议在此类中实现`UIScreenEdgePanGestureRecognizer`手势回调
    - 在识别手势 `.Began` 之后将 `transition` 标识为 `interacting`
    - 为 `transition` 创建 `UIPercentDrivenInteractiveTransition` 对象，并 `startInteractiveTransition`
    - 在 `.Change` 中 `updateInteractiveTransition`
    - 手势结束时（ `.Ended` 和 `.Cancel` ）中根据 `percent` 判断是 `finishInteractiveTransition` 还是 `cancelInteractiveTransition`
    - 一些收尾工作，如回收`UIPercentDrivenInteractiveTransition`对象
- 将 `UINavigationController` 的 `delegate` 设置为此类的实例

#### 基于 present 的转场动画
此种转场需要实现 *UIViewControllerTransitioningDelegate* 协议相关的方法来指定和管理前面定义好的动画对象，大致步骤为：

- 实现 `animationControllerForPresentedController:presentingController:sourceController:` 指定 present 动画对象
- 实现 `animationControllerForDismissedController:` 指定 dismiss 动画对象
- 实现 `interactionControllerForDismissal:` 指定可交互的 dismiss 动画对象
- 实现 `interactionControllerForPresentation:` 指定可交互的  present  动画对象
- 将想要 present / dismiss 的 viewController 的 `transitioningDelegate` 设置为此类的实例

#### iOS8 更新
从 iOS8 开始，Apple 提供了 *UIPresentationController* 来辅助实现自定义的 view transition，所有 `UIViewControllerAnimatedTransitioning` 和 `UINavigationControllerDelegate ` 或 `UIViewControllerTransitioningDelegate` 的相关方法可以在 *UIPresentationController* 的子类中组织，这样让我们的自定义转场效果相关的代码更好组织与管理。可以参考 Apple 的官方例程：[Custom View Controller Presentations and Transitions](https://developer.apple.com/library/ios/samplecode/CustomTransitions/Introduction/Intro.html#//apple_ref/doc/uid/TP40015158)。

#### 第三方框架
开源社区有大神将市场上常见的 App 转场效果实现封装并开源了——[TransitionTreasury](https://github.com/DianQK/TransitionTreasury)。通过这个框架，我们只需要几行代码就可以实现各式各样的转场效果。如果需要基于 push 的转场动画，我们只需要写以下代码：

~~~swift
    func push() {
        let vc = SecondViewController()
        navigationController?.tr_pushViewController(vc, method: TRPushTransitionMethod.Fade, completion: {
                print("Push finish")
            })
    }
~~~

如果需要基于 present 的转场动画，我们只需要：

~~~swift
    func present() {
        let vc = ModalViewController()
        vc.modalDelegate = self // Don't forget to set modalDelegate
        tr_presentViewController(vc, method: TRPresentTransitionMethod.Fade, completion: {
                print("Present finished.")
            })
    }
~~~

## 总结
自定义 viewController 的转场动画主要分两步走，第一步，通过实现 *UIViewControllerAnimatedTransitioning* 协议来自定义装自己想要的动画效果；第二步，根据转场动画的组织方式实现 *UINavigationControllerDelegate* 或 *UIViewControllerTransitioningDelegate* 来管理动画。从 iOS8 开始，我们可以使用 *UIPresentationController* 来管理和组织自定义转场相关的代码。通过使用合适的第三方框架可以大大地简化我们的开发工作。