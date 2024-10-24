api 30



# Touch and input

## Input events

## EventListener 事件监听器

### View.OnClickListener

```java
    /**
     * Interface definition for a callback to be invoked when a view is clicked.
     */
    public interface OnClickListener {
        /**
         * Called when a view has been clicked.
         *
         * @param v The view that was clicked.
         */
        void onClick(View v);
    }
```



### View.OnLongClickListener

```java
    /**
     * Interface definition for a callback to be invoked when a view has been clicked and held.
     */
    public interface OnLongClickListener {
        /**
         * Called when a view has been clicked and held.
         *
         * @param v The view that was clicked and held.
         *
         * @return true if the callback consumed the long click, false otherwise.
         */
        boolean onLongClick(View v);
    }
```







### View.OnFocusChangeListener

```java
    /**
     * Interface definition for a callback to be invoked when the focus state of
     * a view changed.
     */
    public interface OnFocusChangeListener {
        /**
         * Called when the focus state of a view has changed.
         *
         * @param v The view whose state has changed.
         * @param hasFocus The new focus state of v.
         */
        void onFocusChange(View v, boolean hasFocus);
    }
```





### View.OnKeyListener

- 硬件按键

```java
    /**
     * Interface definition for a callback to be invoked when a hardware key event is
     * dispatched to this view. The callback will be invoked before the key event is
     * given to the view. This is only useful for hardware keyboards; a software input
     * method has no obligation to trigger this listener.
     */
    public interface OnKeyListener {
        /**
         * Called when a hardware key is dispatched to a view. This allows listeners to
         * get a chance to respond before the target view.
         * <p>Key presses in software keyboards will generally NOT trigger this method,
         * although some may elect to do so in some situations. Do not assume a
         * software input method has to be key-based; even if it is, it may use key presses
         * in a different way than you expect, so there is no way to reliably catch soft
         * input key presses.
         *
         * @param v The view the key has been dispatched to.
         * @param keyCode The code for the physical key that was pressed
         * @param event The KeyEvent object containing full information about
         *        the event.
         * @return True if the listener has consumed the event, false otherwise.
         */
        boolean onKey(View v, int keyCode, KeyEvent event);
    }
```



### View.OnTouchListener

```java
    /**
     * Interface definition for a callback to be invoked when a touch event is
     * dispatched to this view. The callback will be invoked before the touch
     * event is given to the view.
     */
    public interface OnTouchListener {
        /**
         * Called when a touch event is dispatched to a view. This allows listeners to
         * get a chance to respond before the target view.
         *
         * @param v The view the touch event has been dispatched to.
         * @param event The MotionEvent object containing full information about
         *        the event.
         * @return True if the listener has consumed the event, false otherwise.
         */
        boolean onTouch(View v, MotionEvent event);
    }
```





## EventHandler 事件处理器



### View.onKeyDown

public boolean onKeyDown(int keyCode, KeyEvent event)

### View.onKeyUp

public boolean onKeyUp(int keyCode, KeyEvent event)

### View.onTouchEvent

public boolean onTouchEvent(MotionEvent event)

### View.onFocusChanged

- 在 View 对象获得或失去焦点时调用

protected void onFocusChanged(boolean gainFocus, @FocusDirection int direction,
            @Nullable Rect previouslyFocusedRect)





## Gesture 触摸手势



###  GestureDetector 手势检测器



#### GestureDetector.OnGestureListener

```java
    /**
     * The listener that is used to notify when gestures occur.
     * If you want to listen for all the different gestures then implement
     * this interface. If you only want to listen for a subset it might
     * be easier to extend {@link SimpleOnGestureListener}.
     */
    public interface OnGestureListener {

        /**
         * Notified when a tap occurs with the down {@link MotionEvent}
         * that triggered it. This will be triggered immediately for
         * every down event. All other events should be preceded by this.
         *
         * @param e The down motion event.
         */
        boolean onDown(MotionEvent e);

        /**
         * The user has performed a down {@link MotionEvent} and not performed
         * a move or up yet. This event is commonly used to provide visual
         * feedback to the user to let them know that their action has been
         * recognized i.e. highlight an element.
         *
         * @param e The down motion event
         */
        void onShowPress(MotionEvent e);

        /**
         * Notified when a tap occurs with the up {@link MotionEvent}
         * that triggered it.
         *
         * @param e The up motion event that completed the first tap
         * @return true if the event is consumed, else false
         */
        boolean onSingleTapUp(MotionEvent e);

        /**
         * Notified when a scroll occurs with the initial on down {@link MotionEvent} and the
         * current move {@link MotionEvent}. The distance in x and y is also supplied for
         * convenience.
         *
         * @param e1 The first down motion event that started the scrolling.
         * @param e2 The move motion event that triggered the current onScroll.
         * @param distanceX The distance along the X axis that has been scrolled since the last
         *              call to onScroll. This is NOT the distance between {@code e1}
         *              and {@code e2}.
         * @param distanceY The distance along the Y axis that has been scrolled since the last
         *              call to onScroll. This is NOT the distance between {@code e1}
         *              and {@code e2}.
         * @return true if the event is consumed, else false
         */
        boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY);

        /**
         * Notified when a long press occurs with the initial on down {@link MotionEvent}
         * that trigged it.
         *
         * @param e The initial on down motion event that started the longpress.
         */
        void onLongPress(MotionEvent e);

        /**
         * Notified of a fling event when it occurs with the initial on down {@link MotionEvent}
         * and the matching up {@link MotionEvent}. The calculated velocity is supplied along
         * the x and y axis in pixels per second.
         *
         * @param e1 The first down motion event that started the fling.
         * @param e2 The move motion event that triggered the current onFling.
         * @param velocityX The velocity of this fling measured in pixels per second
         *              along the x axis.
         * @param velocityY The velocity of this fling measured in pixels per second
         *              along the y axis.
         * @return true if the event is consumed, else false
         */
        boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY);
    }
```



#### GestureDetector.SimpleOnGestureListener

- 针对所有的事件都返回 false ,所以可以继承它，复写需要用到的方法即可

```java
    /**
     * A convenience class to extend when you only want to listen for a subset
     * of all the gestures. This implements all methods in the
     * {@link OnGestureListener}, {@link OnDoubleTapListener}, and {@link OnContextClickListener}
     * but does nothing and return {@code false} for all applicable methods.
     */
    public static class SimpleOnGestureListener implements OnGestureListener, OnDoubleTapListener,
            OnContextClickListener {

        public boolean onSingleTapUp(MotionEvent e) {
            return false;
        }

        public void onLongPress(MotionEvent e) {
        }

        public boolean onScroll(MotionEvent e1, MotionEvent e2,
                float distanceX, float distanceY) {
            return false;
        }

        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX,
                float velocityY) {
            return false;
        }

        public void onShowPress(MotionEvent e) {
        }

        public boolean onDown(MotionEvent e) {
            //onDown 是所有事件的开始，如果返回 false 就代表忽略了该后续其他的事件，所以一般都需要 onDown 复写返回 true
            return false;
        }

        public boolean onDoubleTap(MotionEvent e) {
            return false;
        }

        public boolean onDoubleTapEvent(MotionEvent e) {
            return false;
        }

        public boolean onSingleTapConfirmed(MotionEvent e) {
            return false;
        }

        public boolean onContextClick(MotionEvent e) {
            return false;
        }
    }
```





Track touch and pointer movements 跟踪轻触和指针移动





## Track velocity



android.view.VelocityTracker





## scroll gesture



## gesture navigation





## Handle multi-touch gestures



## Drag and scale



#### ScaleGestureDetector



#### ScaleGestureDetector.OnScaleGestureListener

```java
    /**
     * The listener for receiving notifications when gestures occur.
     * If you want to listen for all the different gestures then implement
     * this interface. If you only want to listen for a subset it might
     * be easier to extend {@link SimpleOnScaleGestureListener}.
     *
     * An application will receive events in the following order:
     * <ul>
     *  <li>One {@link OnScaleGestureListener#onScaleBegin(ScaleGestureDetector)}
     *  <li>Zero or more {@link OnScaleGestureListener#onScale(ScaleGestureDetector)}
     *  <li>One {@link OnScaleGestureListener#onScaleEnd(ScaleGestureDetector)}
     * </ul>
     */
    public interface OnScaleGestureListener {
        /**
         * Responds to scaling events for a gesture in progress.
         * Reported by pointer motion.
         *
         * @param detector The detector reporting the event - use this to
         *          retrieve extended info about event state.
         * @return Whether or not the detector should consider this event
         *          as handled. If an event was not handled, the detector
         *          will continue to accumulate movement until an event is
         *          handled. This can be useful if an application, for example,
         *          only wants to update scaling factors if the change is
         *          greater than 0.01.
         */
        public boolean onScale(ScaleGestureDetector detector);

        /**
         * Responds to the beginning of a scaling gesture. Reported by
         * new pointers going down.
         *
         * @param detector The detector reporting the event - use this to
         *          retrieve extended info about event state.
         * @return Whether or not the detector should continue recognizing
         *          this gesture. For example, if a gesture is beginning
         *          with a focal point outside of a region where it makes
         *          sense, onScaleBegin() may return false to ignore the
         *          rest of the gesture.
         */
        public boolean onScaleBegin(ScaleGestureDetector detector);

        /**
         * Responds to the end of a scale gesture. Reported by existing
         * pointers going up.
         *
         * Once a scale has ended, {@link ScaleGestureDetector#getFocusX()}
         * and {@link ScaleGestureDetector#getFocusY()} will return focal point
         * of the pointers remaining on the screen.
         *
         * @param detector The detector reporting the event - use this to
         *          retrieve extended info about event state.
         */
        public void onScaleEnd(ScaleGestureDetector detector);
    }
```



#### ScaleGestureDetector.SimpleOnScaleGestureListener

```java
    /**
     * A convenience class to extend when you only want to listen for a subset
     * of scaling-related events. This implements all methods in
     * {@link OnScaleGestureListener} but does nothing.
     * {@link OnScaleGestureListener#onScale(ScaleGestureDetector)} returns
     * {@code false} so that a subclass can retrieve the accumulated scale
     * factor in an overridden onScaleEnd.
     * {@link OnScaleGestureListener#onScaleBegin(ScaleGestureDetector)} returns
     * {@code true}.
     */
    public static class SimpleOnScaleGestureListener implements OnScaleGestureListener {

        public boolean onScale(ScaleGestureDetector detector) {
            return false;
        }

        public boolean onScaleBegin(ScaleGestureDetector detector) {
            return true;
        }

        public void onScaleEnd(ScaleGestureDetector detector) {
            // Intentionally empty
        }
    }

```



## Manage touch events in a ViewGroup