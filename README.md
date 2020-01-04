# IOS开发-事件--触摸事件

标签（空格分隔）： IOS

---

回顾IOS开发中的知识点，如有引用未注明出处，望告知；如有理解有错误的地方，望指正。感谢！！！

---

Demo：[https://github.com/cnwhao/TestHitTest.git](https://github.com/cnwhao/TestHitTest.git)

---

## Ios App开发中事件分为三类
 
- [x] 触摸事件
        如触摸点击手机屏幕。
- [ ] 加速计事件
        如摇一摇。
- [ ] 远程控制事件
        如耳机控制手机音量。

---
  

## UIResponder

只有继承了UIResponder的对象才能接收并处理事件，又称响应者对象。

---

### 1、UIResponder对事件提供的方法支持

- 触摸事件
```Objective-C
//一根或者多根手指开始触摸view
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
//一根或者多根手指在view上移动（随着手指的移动，会持续调用该方法）
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
//一根或者多根手指离开view
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
//触摸结束前，某个系统事件(例如电话呼入)会打断触摸过程
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```
 - 加速计事件
```Objective-C
- (void)motionBegan:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event;
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(UIEvent *)event;
```
 - 远程控制事件
```Objective-C
- (void)remoteControlReceivedWithEvent:(UIEvent *)event;
```

---


### 2、手势类UITouch

保存手指的相关信息，如触摸的位置、事件、阶段。
一个手指对应一个UITouch对象。
当手指触摸屏幕创建UITouch对象。对多个手指会相应创建多个UITouch。
手机离开屏幕销毁相应的UITouch对象。
当手指移动时，系统会更新对应的UITouch对象。


**重要属性**
> 
`//触摸产生时所处的窗口`  
@property(nonatomic,readonly,retain) UIWindow *window;  
`//触摸产生时所处的视图`  
@property(nonatomic,readonly,retain) UIView *view;  
`//短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多的点击`  
@property(nonatomic,readonly) NSUInteger tapCount;  
`//记录了触摸事件产生或变化时的时间，单位是秒`  
@property(nonatomic,readonly) NSTimeInterval timestamp;  
`//当前触摸事件所处的状态`  
@property(nonatomic,readonly) UITouchPhase phase;
>> UITouchPhase：是一个枚举类型，包含：  
UITouchPhaseBegan（触摸开始）  
UITouchPhaseMoved（接触点移动）  
UITouchPhaseStationary（接触点无移动）  
UITouchPhaseEnded（触摸结束）  
UITouchPhaseCancelled（触摸取消）

**重要方法**

>`//返回值表示触摸在view上的位置，这里返回的位置是针对view的坐标系的（以view的左上角为原点(0, 0)），调用时传入的view参数为nil的话，返回的是触摸点在UIWindow的位置`  
(CGPoint)locationInView:(UIView *)view;  
`//该方法记录了前一个触摸点的位置`  
(CGPoint)previousLocationInView:(UIView *)view;

---

### 3、事件类UIEvent
记录事件产生的时间和类型。
一个完整的触摸过程只会产生一个UIEvent事件。

---

## 事件传递

方向：父控件到子控件
UIApplication-> window->rootViewController->view->subviews

> 
1. 发生触摸事件。  
2. 系统将该事件加入到UIApplication管理的事件队列里。  
3. UIApplication从事件队列取出最前面的事件，首先将该事件发送给应用程序主窗口（keyWindow）。  
4. keyWindow会顺着视图层次结构中找到一个最合适的视图来处理触摸事件。  
5. 找到合适的视图控件，从该视图开始向反方向事件响应处理。  

**如何在视图层次中找到合适的视图处理触摸事件？**
>通过UIView(UIViewGeometry)
// recursively calls -pointInside:withEvent:. point is in the receiver's coordinate system
//当UIView中的isUserInteractionEnabled = NO、isHidden = YES、alpha <= 0.01时，hitTest方法不会被调用；
判断自己是否可以处理该事件  
`- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;`  
重写hitTest方法中使用super代表传递链中下一个UIView，可实现事件多响应。  
调用时机：当事件传递给当前UIView，就会调用。返回哪个UIView，哪个UIView就处理该事件。  
应用场景：view不可见区域的响应。  
判断触摸点是否在自己上面  
`- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent` *)event;  
调用时机：在hitTest方法中调用  
应用场景：扩大UIButton点击响应范围。

---

## 事件响应

响应者链：是由多个响应者对象连接起来的链条。
响应者对象：处理事件的对象。
方向：事件传递反方向，即子控件到父控件。
**响应流程**
> 
1. 事件传递流程中找到合适的处理视图控件（响应者对象）。
2. 如果响应者对象可以处理事件（如按钮点击、添加的手势、重写的touches方法等）则完成事件的响应，传递终止。
3. 如果响应者不能处理事件，则将事件响应传递给下一个响应者，重复步骤**2**。
4. 如果传递到subviews->rootViewContoller->UIWindow->UIApplication->AppDelegate事件依然没有被处理，则事件被丢弃，传递终止。

---
