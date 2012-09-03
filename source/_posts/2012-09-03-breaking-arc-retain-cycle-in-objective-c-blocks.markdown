---
layout: post
title: "Breaking ARC retain cycle in Objective-C blocks"
date: 2012-09-03 17:50
comments: true
categories: 
- iOS
- Objective-C
- Memory-Management
---

In a recent client project, we noticed its iOS app often **received low memory
warning**. The iOS app is developed with ARC-enabled (Automatic Reference Counting).

When profiled the app using **Instruments > Allocations**, we found that 
a lot of unused ViewController objects were not released from memory. 

Retain Cycle
-----
The codebase uses a lot of **Objective-C blocks** as shown below, and 
`self` is often being called within the blocks:
```objective-c
@interface DetailPageViewController : UIViewController
@property(nonatomic, strong) UIButton *backButton;
...
@end

@implementation DetailPageViewController
@synthesize backButton;

- (void)loadView {
  ...
  [self.doneButton onTouch:^(id sender) {
    [self doSomething]; 
    self.isDone = YES;
  }];
}
...
@end
```

In this example code, the controller object holds a **strong reference to
`doneButton` object**. But because `doneButton onTouch:` block is calling
`self`, now the button object **holds a strong reference back to the controller
object**. 

When object A points strongly to object B, and B points strongly to A, a 
[**retain cycle**](http://cocoawithlove.com/2009/07/rules-to-avoid-retain-cycles.html) is created,
and both objects cannot be released from memory.

Use Lifetime Qualifiers
-----

[Apple Developer's guide on ARC transition](http://developer.apple.com/library/mac/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW9) suggested a few ways to **break the retain cycle using different lifetime
qualifiers**. It is definitely a **must read** for iOS development.

If your app is targeted for iOS 5, you can use `__weak` lifetime qualifier to break the cycle:
```
@interface DetailPageViewController : UIViewController
- (void)loadView {
  ...
  __weak DetailPageViewController controller = self;
  [self.doneButton onTouch:^(id sender) {
    [controller doSomething];
    controller.isDone = YES;
  }];
}
@end
```

Because we are using `__weak` qualifier, `doneButton onTouch:` block
only has **a weak reference to the controller object**. Now, the controller
object can be released from memory when its reference count drops to 0.

