

## Toast#makeText

```java
/**
     * Make a standard toast to display using the specified looper.
     * If looper is null, Looper.myLooper() is used.
     *
     * @hide
     */
public static Toast makeText(@NonNull Context context, @Nullable Looper looper,
            @NonNull CharSequence text, @Duration int duration) {
        if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
            Toast result = new Toast(context, looper);
            result.mText = text;
            result.mDuration = duration;
            return result;
        } else {
            Toast result = new Toast(context, looper);
            //加载布局 com.android.internal.R.layout.transient_notification
            View v = ToastPresenter.getTextToastView(context, text);
            result.mNextView = v;
            result.mDuration = duration;

            return result;
        }
}
```

### ToastPresenter#getTextToastView

```java
/**
  *Returns the default text toast view for message {@code text}.
  */
public static View getTextToastView(Context context, CharSequence text) {
        View view = LayoutInflater.from(context).inflate(TEXT_TOAST_LAYOUT, null);
    	//局部变量
        TextView textView = view.findViewById(com.android.internal.R.id.message);
        textView.setText(text);
        return view;
}
```



### Toast#setView

- 可选

```java
 /**
     * Set the view to show.
     *
     * @see #getView
     * @deprecated Custom toast views are deprecated. Apps can create a standard text toast with the
     *      {@link #makeText(Context, CharSequence, int)} method, or use a
     *      <a href="{@docRoot}reference/com/google/android/material/snackbar/Snackbar">Snackbar</a>
     *      when in the foreground. Starting from Android {@link Build.VERSION_CODES#R}, apps
     *      targeting API level {@link Build.VERSION_CODES#R} or higher that are in the background
     *      will not have custom toast views displayed.
     */
    @Deprecated
    public void setView(View view) {
        mNextView = view;
    }
```



## Toast#show

```java
 /**
     * Show the view for the specified duration.
     */
    public void show() {
        if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
            checkState(mNextView != null || mText != null, "You must either set a text or a view");
        } else {
            if (mNextView == null) {
                throw new RuntimeException("setView must have been called");
            }
        }
		//可以考虑动态代理
        INotificationManager service = getService();
        String pkg = mContext.getOpPackageName();
        TN tn = mTN;
        tn.mNextView = mNextView;
        final int displayId = mContext.getDisplayId();

        try {
            if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
                if (mNextView != null) {
                    // It's a custom toast
                    service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
                } else {
                    // It's a text toast
                    ITransientNotificationCallback callback =
                            new CallbackBinder(mCallbacks, mHandler);
                    service.enqueueTextToast(pkg, mToken, mText, mDuration, displayId, callback);
                }
            } else {
                service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
            }
        } catch (RemoteException e) {
            // Empty
        }
    }

```

### Toast#getService

```java
//Toast#sService
private static INotificationManager sService;
//懒汉式单例模式
static private INotificationManager getService() {
        if (sService != null) {
            return sService;
        }
        sService = INotificationManager.Stub.asInterface(
                ServiceManager.getService(Context.NOTIFICATION_SERVICE));
        return sService;
}
```

