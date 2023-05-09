---
title: Android事件分发
date: 2021-06-25 21:25
tags: Android
description: 事件分发作为Android开发中最重要的知识体系之一，了解并熟悉整套的分发机制有助于更好的分析各种点击滑动失效问题，更好去扩展控件的事件功能和开发自定义控件，同时事件分发处理也作为Android面试必问考点之一，如果能有一个全面深入地了解肯定加分不少
---
# 引言

事件分发作为Android开发中最重要的知识体系之一，了解并熟悉整套的分发机制有助于更好的分析各种点击滑动失效问题，更好去扩展控件的事件功能和开发自定义控件，同时事件分发处理也作为Android面试必问考点之一，如果能有一个全面深入地了解肯定加分不少。总结：事件分发机制很重要



# 一 事件分发流程

{% asset_img 640.png 事件分发流程泳道图 %}

流程概述：事件从底层传递到Activity，Activity会通过dispatchTouchEvent将事件传递到窗口对象PhoneWindow中，窗口对象调用superDispatchTouchEvent将事件传递给视图顶层ViewGroup(DecorView对象),ViewGroup会调用dispatchTouchEvent.此处DecorView不会拦截事件，会将事件传递给下一层ViewGroup，下层ViewGroup会通过onInterceptTouchEvent来判断是否拦截事件，如果父布局不拦截此事件，事件将传递给View控件。View控件通过调用onTouchEvent来判断是否消费此事件，如果事件不被消费将交由父布局的onTouchEvent来处理。如果父布局仍然不消费此事件，事件将交由Activity的onTouchEvent来处理。



# 二 重要方法逻辑

以上是对事件分发流程大体上的一个概述，其中还有许多细节需要我们来掌握，其中有三个重要的方法在整个流程中反复调用：**dispatchTouchEvent**，**onInterceptTouchEvent**，**onTouchEvent**.

## 1.dispatchTouchEvent

方法原型：
``` java
/**
@param event The motion event to be dispatched
@return True if the event was handled by the view, false otherwise
*/
public boolean dispatchTouchEvent(MotionEvent event)
```

分发触摸事件的逻辑，方法内部会调用**onInterceptTouchEvent**决定是否拦截事件，调用**onTouchEvent**决定是否消费事件

## 2.onInterceptTouchEvent

方法原型：
``` java
/**
@param ev The motion event being dispatched down the hierarchy
@return Return true to steal motion events from the children and have
* them dispatched to this ViewGroup through onTouchEvent().
* The current target will receive an ACTION_CANCEL event, and no further
* messages will be delivered here.
*/
public boolean onInterceptTouchEvent(MotionEvent ev)
```
拦截事件逻辑。因为是ViewGroup对View的拦截。所以此方法只有ViewGroup才有，View是没有此事件的。同时通过注释可以知道，如果返回true的话，会走到自己的**onTouchEvent**方法。同时给子view发送一个**ACTION_CANCEL**取消事件。同时后面的事件消息将不会传递到这个方法了(一次拦截，一直拦截)。

## 3.onTouchEvent

方法原型：
``` java
    /*
     * @param event The motion event.
     * @return True if the event was handled, false otherwise.
     */
    public boolean onTouchEvent(MotionEvent event)
```
处理事件的逻辑。返回值代表是否消费此事件

**三个方法的总结：分发，拦截，处理(消费)**



# 三 结合源码

掌握了事件分发的基本逻辑后。不知道你是否思考过，

1.如果父布局不拦截ACTION_DOWN事件，只拦截ACTION_MOVE或截ACTION_UP事件会发生什么情况呢？
2.如果按下按钮后滑动离开按钮再松开会怎么样呢？
3.子父布局坐标系原点不一样的情况，事件坐标是如何处理的呢？
等等问题

要回答这些问题，只是以上的基本了解可能并不能很好地回答这些问题。想要了解这些问题，唯有深入源码。
``` java
//ViewGroup.java
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        ...
        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

           
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.
            final boolean intercepted;
            //1.判断是否需要调用拦截
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                    //2.调用过requestDisallowInterceptTouchEvent(true)方法才会有FLAG_DISALLOW_INTERCEPT标志，
                    //默认disallowIntercept始终为false
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                   //3.拦截事件逻辑
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
            ...
            // Check for cancelation.
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;
            ...
            //4.没有拦截
            if (!canceled && !intercepted) {
                // If the event is targeting accessibility focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x =
                                isMouseEvent ? ev.getXCursorPosition() : ev.getX(actionIndex);
                        final float y =
                                isMouseEvent ? ev.getYCursorPosition() : ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);
                            //5.可以接收事件，并且在点击范围
                            if (!child.canReceivePointerEvents()
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                continue;
                            }
                             ...
                             //6.符合条件的child进行事件分发
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                ...
                            }
                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }
                  ...
                }
            }

            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        //7.child事件需要取消
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        //8.回收touchTarget对象
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }
        ...
        return handled;
    }
    
    
    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
        //9.给子child传递一个ACTION_CANCEL事件
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }
        final MotionEvent transformedEvent;
        if (newPointerIdBits == oldPointerIdBits) {
            if (child == null || child.hasIdentityMatrix()) {
                if (child == null) {
                    handled = super.dispatchTouchEvent(event);
                } else {
                    final float offsetX = mScrollX - child.mLeft;
                    final float offsetY = mScrollY - child.mTop;
                    //10.转换坐标
                    event.offsetLocation(offsetX, offsetY);

                    handled = child.dispatchTouchEvent(event);

                    event.offsetLocation(-offsetX, -offsetY);
                }
                return handled;
            }
            transformedEvent = MotionEvent.obtain(event);
        } else {
            transformedEvent = event.split(newPointerIdBits);
        }
        ...
        }
```
**问题1：如果父布局不拦截ACTION_DOWN事件，只拦截ACTION_MOVE或截ACTION_UP事件会发生什么情况呢？**

父布局不拦截**ACTION_DOWN**事件，那么**ACTION_DOWN**事件会子view消费。同时子view会保存在**mFirstTouchTarget**中。第二次**ACTION_MOVE**事件来了之后**onInterceptTouchEvent(ev)**拦截了事件，返回**true**。注释4处判断不会进入，直接走到注释7处。同时传递了**cancelChild**为true。到注释9可以看到，会给子child传递一个**ACTION_CANCEL**事件。然后，子child在**ACTION_CANCEL**中做各种取消和重置处理。<br>
``` java
//View.java
public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();

        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
        ...
        if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    if ((viewFlags & TOOLTIP) == TOOLTIP) {
                    //隐藏tooltip，延时1500ms
                        handleTooltipUp();
                    }
                    if (!clickable) {
                        removeTapCallback();
                        removeLongPressCallback();
                        mInContextButtonPress = false;
                        mHasPerformedLongPress = false;
                        mIgnoreNextUpEvent = false;
                        break;
                    }
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                        // take focus if we don't have it already and we should in
                        // touch mode.
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                            //如果有预按下标记，但是抬起时(由于抬起过早)还没清除此标记，需要设置为按压态
                            setPressed(true, x, y);
                        }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                //点击事件处理
                                if (!post(mPerformClick)) {
                                    performClickInternal();
                                }
                            }
                        }

                        if (mUnsetPressedState == null) {
                            mUnsetPressedState = new UnsetPressedState();
                        }

                        if (prepressed) {
                        //64ms延时，按下状态置为false
                            postDelayed(mUnsetPressedState,
                                    ViewConfiguration.getPressedStateDuration());
                        } else if (!post(mUnsetPressedState)) {
                            // If the post failed, unpress right now
                            mUnsetPressedState.run();
                        }
                        //清除预按下flag
                        removeTapCallback();
                    }
                    mIgnoreNextUpEvent = false;
                    break;

                case MotionEvent.ACTION_DOWN:
                    if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
                        mPrivateFlags3 |= PFLAG3_FINGER_DOWN; 
                    }
                    mHasPerformedLongPress = false;

                    if (!clickable) {
                    //长按runnable
                        checkForLongClick(
                                ViewConfiguration.getLongPressTimeout(),
                                x,
                                y,
                                TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                        break;
                    }

                    if (performButtonActionOnTouchDown(event)) {
                        break;
                    }

                    // Walk up the hierarchy to determine if we're inside a scrolling container.
                    boolean isInScrollingContainer = isInScrollingContainer();

                    // For views inside a scrolling container, delay the pressed feedback for
                    // a short period in case this is a scroll.
                    if (isInScrollingContainer) {
                    //如果在滑动控件里面，设置预按下flag
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        mPendingCheckForTap.x = event.getX();
                        mPendingCheckForTap.y = event.getY();
                        //预按下flag标记清除
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    } else {
                        // Not inside a scrolling container, so show the feedback right away
                        setPressed(true, x, y);
                        checkForLongClick(
                                ViewConfiguration.getLongPressTimeout(),
                                x,
                                y,
                                TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                    }
                    break;

                case MotionEvent.ACTION_CANCEL:
                //重置各种状态和移除runnable
                    if (clickable) {
                        setPressed(false);
                    }
                    removeTapCallback();
                    removeLongPressCallback();
                    mInContextButtonPress = false;
                    mHasPerformedLongPress = false;
                    mIgnoreNextUpEvent = false;
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    break;

                case MotionEvent.ACTION_MOVE:
                    ...
                    //11.离开区域
                    if (!pointInView(x, y, touchSlop)) {
                        // Outside button
                        // Remove any future long press/tap checks
                        removeTapCallback();
                        removeLongPressCallback();
                        if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                            setPressed(false);
                        }
                        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    }

                    final boolean deepPress =
                            motionClassification == MotionEvent.CLASSIFICATION_DEEP_PRESS;
                    if (deepPress && hasPendingLongPressCallback()) {
                        // process the long click action immediately
                        removeLongPressCallback();
                        checkForLongClick(
                                0 /* send immediately */,
                                x,
                                y,
                                TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__DEEP_PRESS);
                    }

                    break;
            }

            return true;
        }

        return false;
    }
```

**问题2:如果按下按钮后滑动离开按钮再松开会怎么样呢？**

注释11处**pointInView(x, y, touchSlop)**会判断点击区域是否在响应区域。当手划出点击view区域时会移除掉各种callback,并且接下来的ACTION_UP事件在注释5处就会过滤掉，无法传递给子view。相应的按钮的点击事件(ACTION_UP中)也无法响应。但是各种状已经在**ACTION_MOVE**中重置了。

**问题3:子父布局坐标系原点不一样的情况，事件坐标是如何处理的呢？**

注释11处通过**mScrollX - child.mLeft**得到在child中的相对x坐标，通过**mScrollY - child.mTop**得到在child中的相对y坐标。然后通过**event.offsetLocation(offsetX, offsetY)**赋值给event。然后分发给子view。



# 四 总结

Android事件分发流程 = Activity ->Window-> ViewGroup -> View，即：1个点击事件发生后，事件先传到Activity、再传到Window、再传到ViewGroup、最终再传到View。
事件分发的方法主要包括：**dispatchTouchEvent()**、**dispatchTransformedTouchEvent**、**onInterceptTouchEvent()**和**onTouchEvent()**。