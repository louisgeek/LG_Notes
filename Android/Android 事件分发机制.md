

## 事件分发

- 责任链模式

四个重要方法
dispatchTouchEvent 事件分发
onInterceptTouchEvent 事件拦截(ViewGroup 才有)
dispatchTransformedTouchEvent
onTouchEvent 事件消费



ViewGroup

- 包含子 View，侧重是否需要拦截事件，是否需要把事件继续分发给子 View，如果把事件分发给子 View

  



View

- 侧重如何去处理事件





### Activity

```java
  /**
     * Called to process touch screen events.  You can override this to
     * intercept all touch screen events before they are dispatched to the
     * window.  Be sure to call this implementation for touch screen events
     * that should be handled normally.
     *
     * @param ev The touch screen event.
     *
     * @return boolean Return true if this event was consumed.
     */
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            //空方法，可覆写
            onUserInteraction();
        }
        //就是 PhoneWindow#superDispatchTouchEvent 
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
   /**
     * Called when a touch screen event was not handled by any of the views
     * under it.  This is most useful to process touch events that happen
     * outside of your window bounds, where there is no view to receive it.
     *
     * @param event The touch screen event being processed.
     *
     * @return Return true if you have consumed the event, false if you haven't.
     * The default implementation always returns false.
     */
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }

        return false;
    }
```



### Window

```java
   /**
     * Used by custom windows, such as Dialog, to pass the touch screen event
     * further down the view hierarchy. Application developers should
     * not need to implement or call this.
     *
     */
    public abstract boolean superDispatchTouchEvent(MotionEvent event);
    
   public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
        final boolean isOutside =
                event.getAction() == MotionEvent.ACTION_UP && isOutOfBounds(context, event)
                || event.getAction() == MotionEvent.ACTION_OUTSIDE;
        if (mCloseOnTouchOutside && peekDecorView() != null && isOutside) {
            return true;
        }
        return false;
    }
```



### PhoneWindow#superDispatchTouchEvent

```java
 @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        //DecorView
        return mDecor.superDispatchTouchEvent(event);
    }
```



### DecorView#superDispatchTouchEvent

```java
public boolean superDispatchTouchEvent(MotionEvent event) {
     //ViewGroup
     return super.dispatchTouchEvent(event);
}
```



### ViewGroup

```java
 @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        //过滤安全的事件
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;
			//MotionEvent.ACTION_DOWN 是作为一个事件的开始
            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                //如果是 ACTION_DOWN 事件或者存在可以消费事件的对象，就准备拦截
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    //如果没有不允许拦截的条件，就交由 onInterceptTouchEvent 去处理是否拦截
                    //关键方法
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                   //如果有不允许拦截的条件，那么就标记未拦截
                    intercepted = false;
                }
            } else {
                //如果不是 ACTION_DOWN 事件且没找到可以消费事件的对象，就直接标记拦截
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

            // Update list of touch targets for pointer down, if needed.
            final boolean isMouseEvent = ev.getSource() == InputDevice.SOURCE_MOUSE;
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0
                    && !isMouseEvent;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
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
                            if (!child.canReceivePointerEvents()
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            //
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                //
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                //如果没找到可以消费事件的对象
                // No touch targets so treat this as an ordinary view.
                //mFirstTouchTarget 为空时候，内部就是调用 super.dispatchTouchEvent 的方法，super 就是指父类，它的 View#dispatchTouchEvent 方法
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
                        //
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
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

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }


    public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
                && ev.getAction() == MotionEvent.ACTION_DOWN
                && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
                && isOnScrollbarThumb(ev.getX(), ev.getY())) {
            return true;
        }
        return false;
    }

    /**
     * Transforms a motion event into the coordinate space of a particular child view,
     * filters out irrelevant pointer ids, and overrides its action if necessary.
     * If child is null, assumes the MotionEvent will be sent to this ViewGroup instead.
     */
    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;

        // Canceling motions is a special case.  We don't need to perform any transformations
        // or filtering.  The important part is the action, not the contents.
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }

        // Calculate the number of pointers to deliver.
        final int oldPointerIdBits = event.getPointerIdBits();
        final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;

        // If for some reason we ended up in an inconsistent state where it looks like we
        // might produce a motion event with no pointers in it, then drop the event.
        if (newPointerIdBits == 0) {
            return false;
        }

        // If the number of pointers is the same and we don't need to perform any fancy
        // irreversible transformations, then we can reuse the motion event for this
        // dispatch as long as we are careful to revert any changes we make.
        // Otherwise we need to make a copy.
        final MotionEvent transformedEvent;
        if (newPointerIdBits == oldPointerIdBits) {
            if (child == null || child.hasIdentityMatrix()) {
                if (child == null) {
                    //由 ViewGroup 的父类方法 View#dispatchTouchEvent 进行分发
                    handled = super.dispatchTouchEvent(event);
                } else {
                    final float offsetX = mScrollX - child.mLeft;
                    final float offsetY = mScrollY - child.mTop;
                    event.offsetLocation(offsetX, offsetY);
					//由子 View 自己的 View#dispatchTouchEvent 进行分发
                    handled = child.dispatchTouchEvent(event);

                    event.offsetLocation(-offsetX, -offsetY);
                }
                return handled;
            }
            transformedEvent = MotionEvent.obtain(event);
        } else {
            transformedEvent = event.split(newPointerIdBits);
        }

        // Perform any necessary transformations and dispatch.
        if (child == null) {
            handled = super.dispatchTouchEvent(transformedEvent);
        } else {
            final float offsetX = mScrollX - child.mLeft;
            final float offsetY = mScrollY - child.mTop;
            transformedEvent.offsetLocation(offsetX, offsetY);
            if (! child.hasIdentityMatrix()) {
                transformedEvent.transform(child.getInverseMatrix());
            }

            handled = child.dispatchTouchEvent(transformedEvent);
        }

        // Done.
        transformedEvent.recycle();
        return handled;
    }

```



### View

```java
    /**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.
     *
     * @param event The motion event to be dispatched.
     * @return True if the event was handled by the view, false otherwise.
     */
    public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }
        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            //处理 OnTouchListener#onTouch 如果返回 true 代表消费了
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
			//如果 OnTouchListener#onTouch 返回 false, 则继续走 onTouchEvent
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }

   /**
     * Implement this method to handle touch screen motion events.
     * <p>
     * If this method is used to detect click actions, it is recommended that
     * the actions be performed by implementing and calling
     * {@link #performClick()}. This will ensure consistent system behavior,
     * including:
     * <ul>
     * <li>obeying click sound preferences
     * <li>dispatching OnClickListener calls
     * <li>handling {@link AccessibilityNodeInfo#ACTION_CLICK ACTION_CLICK} when
     * accessibility features are enabled
     * </ul>
     *
     * @param event The motion event.
     * @return True if the event was handled, false otherwise.
     */
    public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();

        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return clickable;
        }
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }

        if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                    if ((viewFlags & TOOLTIP) == TOOLTIP) {
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
                                if (!post(mPerformClick)) {
                                    //
                                    performClickInternal();
                                }
                            }
                        }

                        if (mUnsetPressedState == null) {
                            mUnsetPressedState = new UnsetPressedState();
                        }

                        if (prepressed) {
                            postDelayed(mUnsetPressedState,
                                    ViewConfiguration.getPressedStateDuration());
                        } else if (!post(mUnsetPressedState)) {
                            // If the post failed, unpress right now
                            mUnsetPressedState.run();
                        }

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
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        mPendingCheckForTap.x = event.getX();
                        mPendingCheckForTap.y = event.getY();
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
                    if (clickable) {
                        drawableHotspotChanged(x, y);
                    }

                    final int motionClassification = event.getClassification();
                    final boolean ambiguousGesture =
                            motionClassification == MotionEvent.CLASSIFICATION_AMBIGUOUS_GESTURE;
                    int touchSlop = mTouchSlop;
                    if (ambiguousGesture && hasPendingLongPressCallback()) {
                        if (!pointInView(x, y, touchSlop)) {
                            // The default action here is to cancel long press. But instead, we
                            // just extend the timeout here, in case the classification
                            // stays ambiguous.
                            removeLongPressCallback();
                            long delay = (long) (ViewConfiguration.getLongPressTimeout()
                                    * mAmbiguousGestureMultiplier);
                            // Subtract the time already spent
                            delay -= event.getEventTime() - event.getDownTime();
                            checkForLongClick(
                                    delay,
                                    x,
                                    y,
                                    TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                        }
                        touchSlop *= mAmbiguousGestureMultiplier;
                    }

                    // Be lenient about moving outside of buttons
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
    /**
     * Entry point for {@link #performClick()} - other methods on View should call it instead of
     * {@code performClick()} directly to make sure the autofill manager is notified when
     * necessary (as subclasses could extend {@code performClick()} without calling the parent's
     * method).
     */
    private boolean performClickInternal() {
        // Must notify autofill manager before performing the click actions to avoid scenarios where
        // the app has a click listener that changes the state of views the autofill service might
        // be interested on.
        notifyAutofillManagerOnClick();
		
        return performClick();
    }

    /**
     * Call this view's OnClickListener, if it is defined.  Performs all normal
     * actions associated with clicking: reporting accessibility event, playing
     * a sound, etc.
     *
     * @return True there was an assigned OnClickListener that was called, false
     *         otherwise is returned.
     */
    // NOTE: other methods on View should not call this method directly, but performClickInternal()
    // instead, to guarantee that the autofill manager is notified when necessary (as subclasses
    // could extend this method without calling super.performClick()).
    public boolean performClick() {
        // We still need to call this method to handle the cases where performClick() was called
        // externally, instead of through performClickInternal()
        notifyAutofillManagerOnClick();

        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            //OnClickListener#onClick
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        notifyEnterOrExitForAutoFillIfNeeded(true);

        return result;
    }
```






Activity#dispatchTouchEvent
- 先调用 Window#superDispatchTouchEvent 就是判断 PhoneWindow#superDispatchTouchEvent
  如果 superDispatchTouchEvent 返回 true 就是意味着 Window 消费了，就不继续执行了
  -- 而他就是直接调用 DecorView#superDispatchTouchEvent，api23 有 Callback 相关代码 cb.dispatchTouchEvent 
  --- ViewGroup#dispatchTouchEvent
  ---- ViewGroup#onInterceptTouchEvent
  ---- ViewGroup#dispatchTransformedTouchEvent
  ----- View#dispatchTouchEvent

  ------ View.OnTouchListener#onTouch

  ------ View#onTouchEvent

  ------- View#performClickInternal

  -------- View#performClick

  --------- View.OnClickListener#onClick

- Activity#onTouchEvent
-- 判断 Window#shouldCloseOnTouch







## 滑动冲突

- 通过调用`父View`的`requestDisallowInterceptTouchEvent`来解决滑动冲突









1 dispatchTouchEvent（）
2 onInterceptTouchEvent（）
3 onTouch（）
4 onTouchEvent（）
5 onClick（）


## aty

### 单击抬起【默认返回】


```
I/MainActivity: dispatchTouchEvent DOWN
I/MainActivity: onTouchEvent DOWN
I/MainActivity: dispatchTouchEvent UP
I/MainActivity: onTouchEvent UP

```



```
graph TB

A[aty dispatchTouchEvent DOWN]-->B[aty onTouchEvent DOWN]

C[aty dispatchTouchEvent UP]-->D[aty onTouchEvent UP]


```





### 单击滑动抬起【默认返回】


```

I/MainActivity: dispatchTouchEvent DOWN
I/MainActivity: onTouchEvent DOWN
I/MainActivity: dispatchTouchEvent MOVE
I/MainActivity: onTouchEvent MOVE
...
I/MainActivity: dispatchTouchEvent MOVE
I/MainActivity: onTouchEvent MOVE
I/MainActivity: dispatchTouchEvent UP
I/MainActivity: onTouchEvent UP

```


```
graph TB

A[aty dispatchTouchEvent DOWN]-->B[aty onTouchEvent DOWN]
B-->C

C[aty dispatchTouchEvent MOVE]-->D[aty onTouchEvent MOVE]

E[aty dispatchTouchEvent UP]-->F[aty onTouchEvent UP]
```


## aty包含viewgroup

### 单击抬起【默认返回】

```
I/MainActivity: dispatchTouchEvent DOWN
I/MyViewGroup: dispatchTouchEvent DOWN
I/MyViewGroup: onInterceptTouchEvent DOWN
I/MyViewGroup: onTouchEvent DOWN
I/MainActivity: onTouchEvent DOWN
I/MainActivity: dispatchTouchEvent UP
I/MainActivity: onTouchEvent UP

MyViewGroup 后续就得不到 UP事件了

```

```
graph TB

A[aty dispatchTouchEvent DOWN]-->B[vp dispatchTouchEvent DOWN]

B-->C[vp onInterceptTouchEvent DOWN]

C-->D[vp onTouchEvent DOWN]

D-->E[aty onTouchEvent DOWN]

F[aty dispatchTouchEvent UP]-->G[aty onTouchEvent UP]

```


### 单击滑动抬起【默认返回】


```
I/MainActivity: dispatchTouchEvent DOWN
I/MyViewGroup: dispatchTouchEvent DOWN
I/MyViewGroup: onInterceptTouchEvent DOWN
I/MyViewGroup: onTouchEvent DOWN
I/MainActivity: onTouchEvent DOWN
I/MainActivity: dispatchTouchEvent MOVE
I/MainActivity: onTouchEvent MOVE
......
I/MainActivity: dispatchTouchEvent MOVE
I/MainActivity: onTouchEvent MOVE
I/MainActivity: dispatchTouchEvent UP
I/MainActivity: onTouchEvent UP

MyViewGroup  后续就得不到MOVE、 UP事件了
 
```


```
graph TB

A[aty dispatchTouchEvent DOWN]-->B[vp dispatchTouchEvent DOWN]

B-->C[vp onInterceptTouchEvent DOWN]

C-->D[vp onTouchEvent DOWN]

D-->E[aty onTouchEvent DOWN]

E-->H[aty dispatchTouchEvent MOVE]

H-->I[aty onTouchEvent MOVE]

F[aty dispatchTouchEvent UP]-->G[aty onTouchEvent UP]

```


## aty包含viewgroup包含view

### 单击抬起【默认返回】



```
I/MainActivity: dispatchTouchEvent DOWN
I/MyViewGroup: dispatchTouchEvent DOWN
I/MyViewGroup: onInterceptTouchEvent DOWN
I/MyView: dispatchTouchEvent DOWN
I/MyView: onTouchEvent DOWN
I/MyViewGroup: onTouchEvent DOWN
I/MainActivity: onTouchEvent DOWN
I/MainActivity: dispatchTouchEvent UP
I/MainActivity: onTouchEvent UP

MyViewGroup、MyView 后续就得不到 UP事件了

```

```
graph TB

A[aty dispatchTouchEvent DOWN]-->B[vp dispatchTouchEvent DOWN]

B-->C[vp onInterceptTouchEvent DOWN]

C-->H[view dispatchTouchEvent DOWN]

H-->I[view onTouchEvent DOWN]

I-->D[vp onTouchEvent DOWN]

D-->E[aty onTouchEvent DOWN]

F[aty dispatchTouchEvent UP]-->G[aty onTouchEvent UP]


```



