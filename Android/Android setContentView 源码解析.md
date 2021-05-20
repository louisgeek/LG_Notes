> 源码基于 Android 11 API 30

# Activity#setContentView

- Activity 创建于 ActivityThread#performLaunchActivity 方法,会调用 Activity#attach 方法

```java
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback,
        AutofillManager.AutofillClient, ContentCaptureManager.ContentCaptureClient {
            
	//每个 Activity 都包含一个 Window 对象，而下文会提到 PhoneWindow 是 Window 抽象类的具体继承实现类
	private Window mWindow;

	public void setContentView(@LayoutRes int layoutResID) {
    	//getWindow 返回是 Window 的对象，Window 是一个抽象类
		//Window 的抽象方法 setContentView 由 PhoneWindow 实现
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
	}
            
	public void setContentView(View view) {
        getWindow().setContentView(view);
        initWindowDecorActionBar();
	}
            
	public void setContentView(View view, ViewGroup.LayoutParams params) {
        getWindow().setContentView(view, params);
        initWindowDecorActionBar();
	}
            
	public final boolean requestWindowFeature(int featureId) {
     // Window#requestFeature
      return getWindow().requestFeature(featureId);
	}
	//Activity#attach 实际是被 ActivityThread#performLaunchActivity 中被调用            
	final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
    
    mFragments.attachHost(null /*parent*/);
    //对 mWindow 进行初始化赋值
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    
	}
            
}
```

- Window 是一个抽象类，提供了一系列绘制窗口的方法

- 一个 Activity 对象对应一个 Window 对象

- 实际是交给 PhoneWindow#setContentView 来处理逻辑

- PhoneWindow 是 Window 的唯一具体继承实现类，Window 是 Activity 和 View 交互的桥梁

- PhoneWindow 对象在 Activity 的 attach() 方法中被初始化创建

  


## 1 PhoneWindow#setContentView

```java
    @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        //mContentParent 是一个 ViewGroup
        //installDecor() 方法里会进行赋值 mContentParent = generateLayout(mDecor);
        if (mContentParent == null) {
        	//为空，初始化 DecorView 、mContentParent
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            //mContentParent 不为空 
            //如果没有转场动画，就移除之前的所有子视图，所以 setContentView 方法可以多次调用 
            mContentParent.removeAllViews();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            //有转场动画
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
           //没有转场动画
           //解析 setContentView 传入的布局文件然后 addView 到 mContentParent 里
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        //直接调用了 View#requestFitSystemWindows(); 
        //应该处理是状态栏显示，ps:沉浸式状态？
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            //通知内容改变
            cb.onContentChanged();
        }
        //标记 mContentParent 已经设置，PhoneWindow#requestFeature 方法里会校验
        mContentParentExplicitlySet = true;
    }
	 @Override
    public void setContentView(View view) {
        setContentView(view, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
    }
	@Override
   public void setContentView(View view, ViewGroup.LayoutParams params) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            //
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            //
            view.setLayoutParams(params);
            final Scene newScene = new Scene(mContentParent, view);
            transitionTo(newScene);
        } else {
            //因为是 View 不需要 inflate 过程了，所以直接 addView
            mContentParent.addView(view, params);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```

1 PhoneWindow#installDecor 初始化 DecorView、mContentParent

2 如果有转场动画，就处理转场动画，否则直接将 setContentView 传入要显示的 layout 布局(布局加载器解析布局文件转换为 View 树)或者 View 添加到 mContentParent(ViewGroup) 中

3 通知 Callback(Activity) 调用 onContentChanged 方法



### 1.1 PhoneWindow#installDecor

```java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
   //DecorView 作为整个应用所有窗口(Activity界面)的根 View 
   // This is the top-level view of the window, containing the window decor.
    private DecorView mDecor;
    // This is the view in which the window contents are placed. It is either
    // mDecor itself, or a child of mDecor where the contents go.
    ViewGroup mContentParent;
    
    //
	private void installDecor() {
        mForceDecorInstall = false;
        //
        if (mDecor == null) {
            //初始化创建 DecorView
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            //与 PhoneWindow 进行关联
            mDecor.setWindow(this);
        }
        //
        if (mContentParent == null) {
            //generateLayout 方法里有 ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT)，而 ID_ANDROID_CONTENT = com.android.internal.R.id.content
            mContentParent = generateLayout(mDecor);

            // Set up decor part of UI to ignore fitsSystemWindows if appropriate.
            mDecor.makeFrameworkOptionalFitsSystemWindows();
			//
            final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(R.id.decor_content_parent);
            if (decorContentParent != null) {
                mDecorContentParent = decorContentParent;
                mDecorContentParent.setWindowCallback(getCallback());
                if (mDecorContentParent.getTitle() == null) {
                    mDecorContentParent.setWindowTitle(mTitle);
                }
				//getLocalFeatures 得到的是前面 generateLayout 里 requestFeature 设置进去的相关参数，结合上文得出Activity#requestWindowFeature 调用进行一些配置必须要在 Activity#setContentView 方法之前  
                final int localFeatures = getLocalFeatures();
                //FEATURE_MAX = FEATURE_ACTIVITY_TRANSITIONS
                for (int i = 0; i < FEATURE_MAX; i++) {
                    if ((localFeatures & (1 << i)) != 0) {
                        mDecorContentParent.initFeature(i);
                    }
                }

                mDecorContentParent.setUiOptions(mUiOptions);

                if ((mResourcesSetFlags & FLAG_RESOURCE_SET_ICON) != 0 ||
                        (mIconRes != 0 && !mDecorContentParent.hasIcon())) {
                    mDecorContentParent.setIcon(mIconRes);
                } else if ((mResourcesSetFlags & FLAG_RESOURCE_SET_ICON) == 0 &&
                        mIconRes == 0 && !mDecorContentParent.hasIcon()) {
                    mDecorContentParent.setIcon(
                            getContext().getPackageManager().getDefaultActivityIcon());
                    mResourcesSetFlags |= FLAG_RESOURCE_SET_ICON_FALLBACK;
                }
                if ((mResourcesSetFlags & FLAG_RESOURCE_SET_LOGO) != 0 ||
                        (mLogoRes != 0 && !mDecorContentParent.hasLogo())) {
                    mDecorContentParent.setLogo(mLogoRes);
                }

                // Invalidate if the panel menu hasn't been created before this.
                // Panel menu invalidation is deferred avoiding application onCreateOptionsMenu
                // being called in the middle of onCreate or similar.
                // A pending invalidation will typically be resolved before the posted message
                // would run normally in order to satisfy instance state restoration.
                PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
                if (!isDestroyed() && (st == null || st.menu == null) && !mIsStartingWindow) {
                    invalidatePanelMenu(FEATURE_ACTION_BAR);
                }
            } else {
                //如果 decorContentParent == null 走这里 
                //就是 getDecorView().findViewById()
                mTitleView = findViewById(R.id.title);
                if (mTitleView != null) {
                    //和上文提到类似，requestWindowFeature(Window.FEATURE_NO_TITLE) 方法需要在 setContentView 之前调用才能生效
                    if ((getLocalFeatures() & (1 << FEATURE_NO_TITLE)) != 0) {
                        //有些布局里标题栏比较复杂，如 screen_title_icons.xml，title_container 是 RelativeLayout
                        //如果设置了 FEATURE_NO_TITLE 参数就隐藏标题栏
                        final View titleContainer = findViewById(R.id.title_container);
                        if (titleContainer != null) {
                            titleContainer.setVisibility(View.GONE);
                        } else {
                            mTitleView.setVisibility(View.GONE);
                        }
                        mContentParent.setForeground(null);
                    } else {
                        //Activity#setTitle -> Activity#onTitleChanged -> Window#setTitle 会设置一次标题的文字，同时也给 mTitle 赋值
                        mTitleView.setText(mTitle);
                    }
                }
            }

            if (mDecor.getBackground() == null && mBackgroundFallbackDrawable != null) {
                mDecor.setBackgroundFallback(mBackgroundFallbackDrawable);
            }

            // Only inflate or create a new TransitionManager if the caller hasn't
            // already set a custom one.
            if (hasFeature(FEATURE_ACTIVITY_TRANSITIONS)) {
                //加载动画资源
                if (mTransitionManager == null) {
                    final int transitionRes = getWindowStyle().getResourceId(
                            R.styleable.Window_windowContentTransitionManager,
                            0);
                    if (transitionRes != 0) {
                        final TransitionInflater inflater = TransitionInflater.from(getContext());
                        mTransitionManager = inflater.inflateTransitionManager(transitionRes,
                                mContentParent);
                    } else {
                        mTransitionManager = new TransitionManager();
                    }
                }

                mEnterTransition = getTransition(mEnterTransition, null,
                        R.styleable.Window_windowEnterTransition);
                mReturnTransition = getTransition(mReturnTransition, USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowReturnTransition);
                mExitTransition = getTransition(mExitTransition, null,
                        R.styleable.Window_windowExitTransition);
                mReenterTransition = getTransition(mReenterTransition, USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowReenterTransition);
                mSharedElementEnterTransition = getTransition(mSharedElementEnterTransition, null,
                        R.styleable.Window_windowSharedElementEnterTransition);
                mSharedElementReturnTransition = getTransition(mSharedElementReturnTransition,
                        USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowSharedElementReturnTransition);
                mSharedElementExitTransition = getTransition(mSharedElementExitTransition, null,
                        R.styleable.Window_windowSharedElementExitTransition);
                mSharedElementReenterTransition = getTransition(mSharedElementReenterTransition,
                        USE_DEFAULT_TRANSITION,
                        R.styleable.Window_windowSharedElementReenterTransition);
                if (mAllowEnterTransitionOverlap == null) {
                    mAllowEnterTransitionOverlap = getWindowStyle().getBoolean(
                            R.styleable.Window_windowAllowEnterTransitionOverlap, true);
                }
                if (mAllowReturnTransitionOverlap == null) {
                    mAllowReturnTransitionOverlap = getWindowStyle().getBoolean(
                            R.styleable.Window_windowAllowReturnTransitionOverlap, true);
                }
                if (mBackgroundFadeDurationMillis < 0) {
                    mBackgroundFadeDurationMillis = getWindowStyle().getInteger(
                            R.styleable.Window_windowTransitionBackgroundFadeDuration,
                            DEFAULT_BACKGROUND_FADE_DURATION_MS);
                }
                if (mSharedElementsUseOverlay == null) {
                    mSharedElementsUseOverlay = getWindowStyle().getBoolean(
                            R.styleable.Window_windowSharedElementsUseOverlay, true);
                }
            }
        }
    }
}
```

- 代码中显示了为什么 Activity#requestWindowFeature 需要在 setContentView 之前调用
- mContentParent 里的子 View 就是 setContentView 时候传入的

1 创建 DecorView ，mDecor = generateDecor(-1)  

2 创建 mContentParent ，mContentParent = generateLayout(mDecor) 



#### 1.1.1 PhoneWindow#generateDecor

```java
protected DecorView generateDecor(int featureId) {
        // System process doesn't have application context and in that case we need to directly use
        // the context we have. Otherwise we want the application context, so we don't cling to the
        // activity.
        Context context;
    	//默认 false 
        if (mUseDecorContext) {
            Context applicationContext = getContext().getApplicationContext();
            if (applicationContext == null) {
                context = getContext();
            } else {
                //这个 this 是传 PhoneWindow
                context = new DecorContext(applicationContext, this);
                if (mTheme != -1) {
                    context.setTheme(mTheme);
                }
            }
        } else {
            // Window 的 mContext;
            context = getContext();
        }
       //这个 this 是传 Window
        return new DecorView(context, featureId, this, getAttributes());
    }
```

- 得到不同的 context 用于创建 DecorView 实例



##### DecorView

- 将要显示的具体内容呈现在 PhoneWindow 上
- 继承自 FrameLayout，就是对 FrameLayout 进行功能的修饰
- 作为整个应用窗口(Activity界面)的根 View，可以说是所有 View 的 parent view ，是 Android 最基本的窗口系统

```java
public class DecorView extends FrameLayout implements RootViewSurfaceTaker, WindowCallbacks {

	private PhoneWindow mWindow;
	//安卓早期源码里 mContentRoot 定义在 PhoneWindow 里
    ViewGroup mContentRoot;
    


}
```



#### 1.1.2 PhoneWindow#generateLayout

```java
 protected ViewGroup generateLayout(DecorView decor) {
        // Apply data from current theme.
        //获取 AndroidManifest.xml 中指定的 themes 主题，如 android:theme="@style/AppTheme"
		//com.android.internal.R.styleable.Window 具体的属性定义在 attrs.xml
        //比如是否有 title、是否是悬浮 window，是否透明等
        TypedArray a = getWindowStyle();
		//写代码时候的临时日志，又不舍得删，变相注释？
        if (false) {
            System.out.println("From style:");
            String s = "Attrs:";
            for (int i = 0; i < R.styleable.Window.length; i++) {
                s = s + " " + Integer.toHexString(R.styleable.Window[i]) + "="
                        + a.getString(i);
            }
            System.out.println(s);
        }
		//是否悬浮
        mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                & (~getForcedWindowFlags());
        if (mIsFloating) {
            setLayout(WRAP_CONTENT, WRAP_CONTENT);
            setFlags(0, flagsToUpdate);
        } else {
            setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
            getAttributes().setFitInsetsSides(0);
            getAttributes().setFitInsetsTypes(0);
        }
        if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {
            //是否没有标题栏，常用来去除标题栏的功能
            requestFeature(FEATURE_NO_TITLE);
        } else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {
            // Don't allow an action bar if there is no title.
            requestFeature(FEATURE_ACTION_BAR);
        }
        if (a.getBoolean(R.styleable.Window_windowActionBarOverlay, false)) {
            requestFeature(FEATURE_ACTION_BAR_OVERLAY);
        }
        if (a.getBoolean(R.styleable.Window_windowActionModeOverlay, false)) {
            requestFeature(FEATURE_ACTION_MODE_OVERLAY);
        }
		//全屏
        if (a.getBoolean(R.styleable.Window_windowFullscreen, false)) {
            setFlags(FLAG_FULLSCREEN, FLAG_FULLSCREEN & (~getForcedWindowFlags()));
        }
        if (a.getBoolean(R.styleable.Window_windowTranslucentStatus,
                false)) {
            setFlags(FLAG_TRANSLUCENT_STATUS, FLAG_TRANSLUCENT_STATUS
                    & (~getForcedWindowFlags()));
        }
        if (a.getBoolean(R.styleable.Window_windowTranslucentNavigation,
                false)) {
            setFlags(FLAG_TRANSLUCENT_NAVIGATION, FLAG_TRANSLUCENT_NAVIGATION
                    & (~getForcedWindowFlags()));
        }
        if (a.getBoolean(R.styleable.Window_windowShowWallpaper, false)) {
            setFlags(FLAG_SHOW_WALLPAPER, FLAG_SHOW_WALLPAPER&(~getForcedWindowFlags()));
        }
        if (a.getBoolean(R.styleable.Window_windowEnableSplitTouch,
                getContext().getApplicationInfo().targetSdkVersion
                        >= android.os.Build.VERSION_CODES.HONEYCOMB)) {
            setFlags(FLAG_SPLIT_TOUCH, FLAG_SPLIT_TOUCH&(~getForcedWindowFlags()));
        }
        a.getValue(R.styleable.Window_windowMinWidthMajor, mMinWidthMajor);
        a.getValue(R.styleable.Window_windowMinWidthMinor, mMinWidthMinor);
        if (DEBUG) Log.d(TAG, "Min width minor: " + mMinWidthMinor.coerceToString()
                + ", major: " + mMinWidthMajor.coerceToString());
        if (a.hasValue(R.styleable.Window_windowFixedWidthMajor)) {
            if (mFixedWidthMajor == null) mFixedWidthMajor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedWidthMajor,
                    mFixedWidthMajor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedWidthMinor)) {
            if (mFixedWidthMinor == null) mFixedWidthMinor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedWidthMinor,
                    mFixedWidthMinor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedHeightMajor)) {
            if (mFixedHeightMajor == null) mFixedHeightMajor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedHeightMajor,
                    mFixedHeightMajor);
        }
        if (a.hasValue(R.styleable.Window_windowFixedHeightMinor)) {
            if (mFixedHeightMinor == null) mFixedHeightMinor = new TypedValue();
            a.getValue(R.styleable.Window_windowFixedHeightMinor,
                    mFixedHeightMinor);
        }
        if (a.getBoolean(R.styleable.Window_windowContentTransitions, false)) {
            requestFeature(FEATURE_CONTENT_TRANSITIONS);
        }
        if (a.getBoolean(R.styleable.Window_windowActivityTransitions, false)) {
            requestFeature(FEATURE_ACTIVITY_TRANSITIONS);
        }
		//是否是半透明的
        mIsTranslucent = a.getBoolean(R.styleable.Window_windowIsTranslucent, false);
		//
        final Context context = getContext();
        final int targetSdk = context.getApplicationInfo().targetSdkVersion;
     	//版本兼容 声明变量，方便配合 if 使用
        final boolean targetPreL = targetSdk < android.os.Build.VERSION_CODES.LOLLIPOP;
        final boolean targetPreQ = targetSdk < Build.VERSION_CODES.Q;

        if (!mForcedStatusBarColor) {
            mStatusBarColor = a.getColor(R.styleable.Window_statusBarColor, 0xFF000000);
        }
        if (!mForcedNavigationBarColor) {
            mNavigationBarColor = a.getColor(R.styleable.Window_navigationBarColor, 0xFF000000);
            mNavigationBarDividerColor = a.getColor(R.styleable.Window_navigationBarDividerColor,0x00000000);
        }
        if (!targetPreQ) {
            mEnsureStatusBarContrastWhenTransparent = a.getBoolean(
                    R.styleable.Window_enforceStatusBarContrast, false);
            mEnsureNavigationBarContrastWhenTransparent = a.getBoolean(
                    R.styleable.Window_enforceNavigationBarContrast, true);
        }

        WindowManager.LayoutParams params = getAttributes();

        // Non-floating windows on high end devices must put up decor beneath the system bars and
        // therefore must know about visibility changes of those.
        if (!mIsFloating) {
            if (!targetPreL && a.getBoolean(
                    R.styleable.Window_windowDrawsSystemBarBackgrounds,
                    false)) {
                setFlags(FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS,
                        FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS & ~getForcedWindowFlags());
            }
            if (mDecor.mForceWindowDrawsBarBackgrounds) {
                params.privateFlags |= PRIVATE_FLAG_FORCE_DRAW_BAR_BACKGROUNDS;
            }
        }
        if (a.getBoolean(R.styleable.Window_windowLightStatusBar, false)) {
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
        }
        if (a.getBoolean(R.styleable.Window_windowLightNavigationBar, false)) {
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR);
        }
        if (a.hasValue(R.styleable.Window_windowLayoutInDisplayCutoutMode)) {
            int mode = a.getInt(R.styleable.Window_windowLayoutInDisplayCutoutMode, -1);
            if (mode < LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT
                    || mode > LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS) {
                throw new UnsupportedOperationException("Unknown windowLayoutInDisplayCutoutMode: "
                        + a.getString(R.styleable.Window_windowLayoutInDisplayCutoutMode));
            }
            params.layoutInDisplayCutoutMode = mode;
        }

        if (mAlwaysReadCloseOnTouchAttr || getContext().getApplicationInfo().targetSdkVersion
                >= android.os.Build.VERSION_CODES.HONEYCOMB) {
            if (a.getBoolean(
                    R.styleable.Window_windowCloseOnTouchOutside,
                    false)) {
                setCloseOnTouchOutsideIfNotSet(true);
            }
        }

        if (!hasSoftInputMode()) {
            params.softInputMode = a.getInt(
                    R.styleable.Window_windowSoftInputMode,
                    params.softInputMode);
        }

        if (a.getBoolean(R.styleable.Window_backgroundDimEnabled,
                mIsFloating)) {
            /* All dialogs should have the window dimmed */
            if ((getForcedWindowFlags()&WindowManager.LayoutParams.FLAG_DIM_BEHIND) == 0) {
                params.flags |= WindowManager.LayoutParams.FLAG_DIM_BEHIND;
            }
            if (!haveDimAmount()) {
                params.dimAmount = a.getFloat(
                        android.R.styleable.Window_backgroundDimAmount, 0.5f);
            }
        }

        if (params.windowAnimations == 0) {
            params.windowAnimations = a.getResourceId(
                    R.styleable.Window_windowAnimationStyle, 0);
        }

        // The rest are only done if this window is not embedded; otherwise,
        // the values are inherited from our container.
        //private Window mContainer;
        if (getContainer() == null) {
            if (mBackgroundDrawable == null) {

                if (mFrameResource == 0) {
                    mFrameResource = a.getResourceId(R.styleable.Window_windowFrame, 0);
                }

                if (a.hasValue(R.styleable.Window_windowBackground)) {
                    mBackgroundDrawable = a.getDrawable(R.styleable.Window_windowBackground);
                }
            }
            if (a.hasValue(R.styleable.Window_windowBackgroundFallback)) {
                mBackgroundFallbackDrawable =
                        a.getDrawable(R.styleable.Window_windowBackgroundFallback);
            }
            if (mLoadElevation) {
                mElevation = a.getDimension(R.styleable.Window_windowElevation, 0);
            }
            mClipToOutline = a.getBoolean(R.styleable.Window_windowClipToOutline, false);
            mTextColor = a.getColor(R.styleable.Window_textColor, Color.TRANSPARENT);
        }

        // Inflate the window decor.

        int layoutResource;
        int features = getLocalFeatures();
		//省略部分代码：通过 features 判断加载不同的【标题+内容】内置系统布局文件
        //比如 layoutResource = R.layout.screen_simple_overlay_action_mode, 这些布局里都有一个 android:id="@android:id/content" 的 FrameLayout，用于添加自己的布局，默认加载 R.layout.screen_simple 布局
		//标记 mChanging = true;
        mDecor.startChanging();
        //解析 screen_simple.xml 等内置系统布局
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
		//ID_ANDROID_CONTENT 的对应的 View，赋值给 contentParent
     	//Window#findViewById() 里面就是调用 Window#getDecorView().findViewById()
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        if (contentParent == null) {
            throw new RuntimeException("Window couldn't find content container view");
        }
        if ((features & (1 << FEATURE_INDETERMINATE_PROGRESS)) != 0) {
            ProgressBar progress = getCircularProgressBar(false);
            if (progress != null) {
                progress.setIndeterminate(true);
            }
        }
        // Remaining setup -- of background and title -- that only applies
        // to top-level windows.
        if (getContainer() == null) {
            mDecor.setWindowBackground(mBackgroundDrawable);

            final Drawable frame;
            if (mFrameResource != 0) {
                frame = getContext().getDrawable(mFrameResource);
            } else {
                frame = null;
            }
            mDecor.setWindowFrame(frame);

            mDecor.setElevation(mElevation);
            mDecor.setClipToOutline(mClipToOutline);

            if (mTitle != null) {
                setTitle(mTitle);
            }

            if (mTitleColor == 0) {
                mTitleColor = mTextColor;
            }
            setTitleColor(mTitleColor);
        }
		//标记 mChanging = false;
        //里面调用了 drawableChanged();
        mDecor.finishChanging();

        return contentParent;
    }

```
- 通过 WindowStyle 中设置的各种属性对 Window 进行 requestFeature 、setFlags 、赋值等一系列操作

  

1. 从 com.android.internal.R.styleable.Window(android.R.styleable#Window) 获取到对应的 attributes 属性，就是获取 AndroidManifest.xml 中设置的主题样式属性，window支持的属性写在  attrs.xml  里

2. Window#setLayout 、Window#setFlags 、PhoneWindow#requestFeature 、params(=getAttributes()) 各个参数赋值 、decor.setSystemUiVisibility 等配置

3. 对 PhoneWindow 里的 mIsFloating 、mIsTranslucent 、mStatusBarColor 、mNavigationBarColor 、mNavigationBarDividerColor 、mFrameResource 、mBackgroundDrawable 、mBackgroundFallbackDrawable 、mElevation 、mClipToOutline 、mTextColor 等变量进行赋值

4. 通过 getLocalFeatures 得到 requestFeature 设置等相关参数然后进行判断后面需要使用哪个布局文件赋值给 layoutResource 进行处理， 添加到 DecorView 中

5. 用 mDecor.startChanging() 打上开始 mChanging 的标记 ，然后通过 mDecor.onResourcesLoaded(mLayoutInflater, layoutResource) 进行加载

```java
void onResourcesLoaded(LayoutInflater inflater, int layoutResource) {
        if (mBackdropFrameRenderer != null) {
            loadBackgroundDrawablesIfNeeded();
            mBackdropFrameRenderer.onResourcesLoaded(
                    this, mResizingBackgroundDrawable, mCaptionBackgroundDrawable,
                    mUserCaptionBackgroundDrawable, getCurrentColor(mStatusColorViewState),
                    getCurrentColor(mNavigationColorViewState));
        }
		//是一个 ViewGroup，可以最大化、关闭窗口
        mDecorCaptionView = createDecorCaptionView(inflater);
        //LayoutInflater 解析 layoutResource 对应的【标题+内容】内置系统布局文件
        final View root = inflater.inflate(layoutResource, null);
        if (mDecorCaptionView != null) {
            //如果配置有 DecorCaptionView
            if (mDecorCaptionView.getParent() == null) {
                //mDecorCaptionView 还没被 parent.addView 的话
                //那么需要进行 addView, 将 mDecorCaptionView 放入 DecorView
                addView(mDecorCaptionView,
                        new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
            }
            //把 layoutResource 布局文件 inflate 得到的 root 放入 mDecorCaptionView 
            mDecorCaptionView.addView(root,
                    new ViewGroup.MarginLayoutParams(MATCH_PARENT, MATCH_PARENT));
        } else {
            // Put it below the color views.
            //如果没有 DecorCaptionView 那就把 layoutResource 布局文件 inflate 得到的 root 直接放入 DecorView 中
            addView(root, 0, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
        }
    	//赋值给 mContentRoot
        mContentRoot = (ViewGroup) root;
    	//走初始化阴影逻辑
        initializeElevation();
    }
```



6. 再通过 findViewById 找到里面的 com.android.internal.R.id.content 的 View，作为 mContentParent
7. if (getContainer() == null) 情况下对 mDecor.setWindowBackground 、mDecor.setWindowFrame 、mDecor.setElevation 、mDecor.setClipToOutline 等处理，以及  PhoneWindow#setTitle 、PhoneWindow#setTitleColor
8. 用 mDecor.finishChanging()  打上完成 mChanging 的标记，然后立即调用里面的 drawableChanged() 方法，该方法  

```java
  private void drawableChanged() {
       //校验 mChanging 
        if (mChanging) {
            return;
        }
        // Fields can be null if super constructor calls setBackgroundDrawable.
        Rect framePadding = mFramePadding != null ? mFramePadding : new Rect();
        Rect backgroundPadding = mBackgroundPadding != null ? mBackgroundPadding : new Rect();

        setPadding(framePadding.left + backgroundPadding.left,
                framePadding.top + backgroundPadding.top,
                framePadding.right + backgroundPadding.right,
                framePadding.bottom + backgroundPadding.bottom);
        requestLayout();
        invalidate();
//省略通过判断获取 opacity 值的代码
//使用 opacity
   mDefaultOpacity = opacity;
        if (mFeatureId < 0) {
            mWindow.setDefaultWindowFormat(opacity);
    }
}
```

9. 最后返回 contentParent



## 总结

- Window 是一个抽象类，提供了一系列绘制窗口的方法
- 一个 Activity 对象对应一个 Window 对象
- Activity 的展示界面的特性是通过 Window 对象来处理的
- setContentView 实际是交给 PhoneWindow#setContentView 来处理逻辑
- PhoneWindow 是 Window 的唯一具体继承实现类，所以 Window 是 Activity 和 View 关联的桥梁
- PhoneWindow 对象在 Activity 启动后的 attach() 方法中被初始化创建，在 onCreate 方法之前
- PhoneWindow 对象内部 mDectorView 对象，是 Acitivty 界面的根 view 
- mContentParent 里的子 View 就是 setContentView 时候传入的
- PhoneWindow#installDecor 方法中显示了为什么 Activity#requestWidowFeature 需要在 setContentView 之前调用



## 流程

Activity#setContentView

PhoneWindow#setContentView

- 首先判断 mContentParent == null，则走 PhoneWindow#installDecor 初始化 DecorView、mContentParent
  - 判断 mDecor == null ，通过 mDecor = generateDecor(-1)  创建 DecorView ，否则直接 mDecor.setWindow(this)
  - 判断 mContentParent == null，则通过 mContentParent = generateLayout(mDecor)  创建 mContentParent 
    - 获取 AndroidManifest.xml 设置的主题，通过 WindowStyle 中设置的各种属性对 Window 进行 requestFeature 、setFlags 、赋值等一系列操作
    - 通过各个属性确定加个对应的系统布局文件，然后通过 mDecor.onResourcesLoaded(mLayoutInflater, layoutResource) 加载成 mContentRoot 并放入 mDecorView 中，并且找到 android:id="@android:id/content" 作为 mContentParent 进行返回
- 如果有转场动画，就处理转场动画，否则进行 mContentParent.removeAllViews() 后就将 setContentView 传入要显示的 layout 布局(布局加载器解析布局文件转换为 View 树)或者 View 添加到 mContentParent(ViewGroup) 中
- 通知 Callback(Activity) 调用 onContentChanged 方法

 

## 视图层级

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032322270172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdWlzZ2Vlaw==,size_16,color_FFFFFF,t_70#pic_center)

 Activity -> mPhoneWindow -> mDectorView -> mContentRoot -> mContentParent ->  Custom Layout View



- screen_simple.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
         android:id="@android:id/content"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:foregroundInsidePadding="false"
         android:foregroundGravity="fill_horizontal|top"
         android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```





> 源码基于 androidx.appcompat:appcompat:1.2.0

# AppCompatActivity#setContentView

- 实现setFactory把TextView转为AppCompatTextView，ImageView转AppCompatImageView等

```java
    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        getDelegate().setContentView(layoutResID);
    }

    @Override
    public void setContentView(View view) {
        getDelegate().setContentView(view);
    }

    @Override
    public void setContentView(View view, ViewGroup.LayoutParams params) {
        getDelegate().setContentView(view, params);
    }
   /**
     * @return The {@link AppCompatDelegate} being used by this Activity.
     */
    @NonNull
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }
```

AppCompatDelegate#create

```java
  /**
     * Create an {@link androidx.appcompat.app.AppCompatDelegate} to use with {@code activity}.
     *
     * @param callback An optional callback for AppCompat specific events
     */
    @NonNull
    public static AppCompatDelegate create(@NonNull Activity activity,
            @Nullable AppCompatCallback callback) {
        return new AppCompatDelegateImpl(activity, callback);
    }


    AppCompatDelegateImpl(Activity activity, AppCompatCallback callback) {
        this(activity, null, callback, activity);
    }

    AppCompatDelegateImpl(Dialog dialog, AppCompatCallback callback) {
        this(dialog.getContext(), dialog.getWindow(), callback, dialog);
    }

    AppCompatDelegateImpl(Context context, Window window, AppCompatCallback callback) {
        this(context, window, callback, context);
    }

    AppCompatDelegateImpl(Context context, Activity activity, AppCompatCallback callback) {
        this(context, null, callback, activity);
    }

    private AppCompatDelegateImpl(Context context, Window window, AppCompatCallback callback,
            Object host) {
        mContext = context;
        mAppCompatCallback = callback;
        mHost = host;

        if (mLocalNightMode == MODE_NIGHT_UNSPECIFIED && mHost instanceof Dialog) {
            final AppCompatActivity activity = tryUnwrapContext();
            if (activity != null) {
                // This code path is used to detect when this Delegate is a child Delegate from
                // an Activity, primarily for Dialogs. Dialogs use the Activity as it's Context,
                // so we want to make sure that the this 'child' delegate does not interfere
                // with the Activity config. The simplest way to do that is to match the
                // outer Activity's local night mode
                mLocalNightMode = activity.getDelegate().getLocalNightMode();
            }
        }
        if (mLocalNightMode == MODE_NIGHT_UNSPECIFIED) {
            // Try and read the current night mode from our static store
            final Integer value = sLocalNightModes.get(mHost.getClass().getName());
            if (value != null) {
                mLocalNightMode = value;
                // Finally remove the value
                sLocalNightModes.remove(mHost.getClass().getName());
            }
        }

        if (window != null) {
            attachToWindow(window);
        }

        // Preload appcompat-specific handling of drawables that should be handled in a special
        // way (for tinting etc). After the following line completes, calls from AppCompatResources
        // to ResourceManagerInternal (in appcompat-resources) will handle those internal drawable
        // paths correctly without having to go through AppCompatDrawableManager APIs.
        AppCompatDrawableManager.preload();
    }

```

AppCompatDelegateImpl#attachToWindow

```java
    private void attachToWindow(@NonNull Window window) {
        if (mWindow != null) {
            throw new IllegalStateException(
                    "AppCompat has already installed itself into the Window");
        }

        final Window.Callback callback = window.getCallback();
        if (callback instanceof AppCompatWindowCallback) {
            throw new IllegalStateException(
                    "AppCompat has already installed itself into the Window");
        }
        mAppCompatWindowCallback = new AppCompatWindowCallback(callback);
        // Now install the new callback
        window.setCallback(mAppCompatWindowCallback);

        final TintTypedArray a = TintTypedArray.obtainStyledAttributes(
                mContext, null, sWindowBackgroundStyleable);
        final Drawable winBg = a.getDrawableIfKnown(0);
        if (winBg != null) {
            // Now set the background drawable
            window.setBackgroundDrawable(winBg);
        }
        a.recycle();

        mWindow = window;
    }
```



## 1 AppCompatDelegateImpl#setContentView

```java
    @Override
    public void setContentView(int resId) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        LayoutInflater.from(mContext).inflate(resId, contentParent);
        mAppCompatWindowCallback.getWrapped().onContentChanged();
    }

   @Override
    public void setContentView(View v) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        contentParent.addView(v);
        mAppCompatWindowCallback.getWrapped().onContentChanged();
    }
    
    @Override
    public void setContentView(View v, ViewGroup.LayoutParams lp) {
        ensureSubDecor();
        ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
        contentParent.removeAllViews();
        contentParent.addView(v, lp);
        mAppCompatWindowCallback.getWrapped().onContentChanged();
    }
```



### 1.1 AppCompatDelegateImpl#ensureSubDecor

```java
private void ensureSubDecor() {
        if (!mSubDecorInstalled) {
            mSubDecor = createSubDecor();

            // If a title was set before we installed the decor, propagate it now
            CharSequence title = getTitle();
            if (!TextUtils.isEmpty(title)) {
                if (mDecorContentParent != null) {
                    mDecorContentParent.setWindowTitle(title);
                } else if (peekSupportActionBar() != null) {
                    peekSupportActionBar().setWindowTitle(title);
                } else if (mTitleView != null) {
                    mTitleView.setText(title);
                }
            }

            applyFixedSizeWindow();

            onSubDecorInstalled(mSubDecor);

            mSubDecorInstalled = true;

            // Invalidate if the panel menu hasn't been created before this.
            // Panel menu invalidation is deferred avoiding application onCreateOptionsMenu
            // being called in the middle of onCreate or similar.
            // A pending invalidation will typically be resolved before the posted message
            // would run normally in order to satisfy instance state restoration.
            PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
            if (!mIsDestroyed && (st == null || st.menu == null)) {
                invalidatePanelMenu(FEATURE_SUPPORT_ACTION_BAR);
            }
        }
    }
```



#### 1.1.1 AppCompatDelegateImpl#createSubDecor

```java
private ViewGroup createSubDecor() {
        TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);

        if (!a.hasValue(R.styleable.AppCompatTheme_windowActionBar)) {
            a.recycle();
            throw new IllegalStateException(
                    "You need to use a Theme.AppCompat theme (or descendant) with this activity.");
        }

        if (a.getBoolean(R.styleable.AppCompatTheme_windowNoTitle, false)) {
            requestWindowFeature(Window.FEATURE_NO_TITLE);
        } else if (a.getBoolean(R.styleable.AppCompatTheme_windowActionBar, false)) {
            // Don't allow an action bar if there is no title.
            requestWindowFeature(FEATURE_SUPPORT_ACTION_BAR);
        }
        if (a.getBoolean(R.styleable.AppCompatTheme_windowActionBarOverlay, false)) {
            requestWindowFeature(FEATURE_SUPPORT_ACTION_BAR_OVERLAY);
        }
        if (a.getBoolean(R.styleable.AppCompatTheme_windowActionModeOverlay, false)) {
            requestWindowFeature(FEATURE_ACTION_MODE_OVERLAY);
        }
        mIsFloating = a.getBoolean(R.styleable.AppCompatTheme_android_windowIsFloating, false);
        a.recycle();

        // Now let's make sure that the Window has installed its decor by retrieving it
        ensureWindow();
        mWindow.getDecorView();

        final LayoutInflater inflater = LayoutInflater.from(mContext);
        ViewGroup subDecor = null;


        if (!mWindowNoTitle) {
            if (mIsFloating) {
                // If we're floating, inflate the dialog title decor
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_dialog_title_material, null);

                // Floating windows can never have an action bar, reset the flags
                mHasActionBar = mOverlayActionBar = false;
            } else if (mHasActionBar) {
                /**
                 * This needs some explanation. As we can not use the android:theme attribute
                 * pre-L, we emulate it by manually creating a LayoutInflater using a
                 * ContextThemeWrapper pointing to actionBarTheme.
                 */
                TypedValue outValue = new TypedValue();
                mContext.getTheme().resolveAttribute(R.attr.actionBarTheme, outValue, true);

                Context themedContext;
                if (outValue.resourceId != 0) {
                    themedContext = new ContextThemeWrapper(mContext, outValue.resourceId);
                } else {
                    themedContext = mContext;
                }

                // Now inflate the view using the themed context and set it as the content view
                subDecor = (ViewGroup) LayoutInflater.from(themedContext)
                        .inflate(R.layout.abc_screen_toolbar, null);

                mDecorContentParent = (DecorContentParent) subDecor
                        .findViewById(R.id.decor_content_parent);
                mDecorContentParent.setWindowCallback(getWindowCallback());

                /**
                 * Propagate features to DecorContentParent
                 */
                if (mOverlayActionBar) {
                    mDecorContentParent.initFeature(FEATURE_SUPPORT_ACTION_BAR_OVERLAY);
                }
                if (mFeatureProgress) {
                    mDecorContentParent.initFeature(Window.FEATURE_PROGRESS);
                }
                if (mFeatureIndeterminateProgress) {
                    mDecorContentParent.initFeature(Window.FEATURE_INDETERMINATE_PROGRESS);
                }
            }
        } else {
            if (mOverlayActionMode) {
                subDecor = (ViewGroup) inflater.inflate(
                        R.layout.abc_screen_simple_overlay_action_mode, null);
            } else {
                subDecor = (ViewGroup) inflater.inflate(R.layout.abc_screen_simple, null);
            }
        }

        if (subDecor == null) {
            throw new IllegalArgumentException(
                    "AppCompat does not support the current theme features: { "
                            + "windowActionBar: " + mHasActionBar
                            + ", windowActionBarOverlay: "+ mOverlayActionBar
                            + ", android:windowIsFloating: " + mIsFloating
                            + ", windowActionModeOverlay: " + mOverlayActionMode
                            + ", windowNoTitle: " + mWindowNoTitle
                            + " }");
        }

        if (Build.VERSION.SDK_INT >= 21) {
            // If we're running on L or above, we can rely on ViewCompat's
            // setOnApplyWindowInsetsListener
            ViewCompat.setOnApplyWindowInsetsListener(subDecor, new OnApplyWindowInsetsListener() {
                        @Override
                        public WindowInsetsCompat onApplyWindowInsets(View v,
                                WindowInsetsCompat insets) {
                            final int top = insets.getSystemWindowInsetTop();
                            final int newTop = updateStatusGuard(insets, null);

                            if (top != newTop) {
                                insets = insets.replaceSystemWindowInsets(
                                        insets.getSystemWindowInsetLeft(),
                                        newTop,
                                        insets.getSystemWindowInsetRight(),
                                        insets.getSystemWindowInsetBottom());
                            }

                            // Now apply the insets on our view
                            return ViewCompat.onApplyWindowInsets(v, insets);
                        }
                    });
        } else if (subDecor instanceof FitWindowsViewGroup) {
            // Else, we need to use our own FitWindowsViewGroup handling
            ((FitWindowsViewGroup) subDecor).setOnFitSystemWindowsListener(
                    new FitWindowsViewGroup.OnFitSystemWindowsListener() {
                        @Override
                        public void onFitSystemWindows(Rect insets) {
                            insets.top = updateStatusGuard(null, insets);
                        }
                    });
        }

        if (mDecorContentParent == null) {
            mTitleView = (TextView) subDecor.findViewById(R.id.title);
        }

        // Make the decor optionally fit system windows, like the window's decor
        ViewUtils.makeOptionalFitsSystemWindows(subDecor);

        final ContentFrameLayout contentView = (ContentFrameLayout) subDecor.findViewById(
                R.id.action_bar_activity_content);

        final ViewGroup windowContentView = (ViewGroup) mWindow.findViewById(android.R.id.content);
        if (windowContentView != null) {
            // There might be Views already added to the Window's content view so we need to
            // migrate them to our content view
            while (windowContentView.getChildCount() > 0) {
                final View child = windowContentView.getChildAt(0);
                windowContentView.removeViewAt(0);
                contentView.addView(child);
            }

            // Change our content FrameLayout to use the android.R.id.content id.
            // Useful for fragments.
            windowContentView.setId(View.NO_ID);
            contentView.setId(android.R.id.content);

            // The decorContent may have a foreground drawable set (windowContentOverlay).
            // Remove this as we handle it ourselves
            if (windowContentView instanceof FrameLayout) {
                ((FrameLayout) windowContentView).setForeground(null);
            }
        }

        // Now set the Window's content view with the decor
        mWindow.setContentView(subDecor);

        contentView.setAttachListener(new ContentFrameLayout.OnAttachListener() {
            @Override
            public void onAttachedFromWindow() {}

            @Override
            public void onDetachedFromWindow() {
                dismissPopups();
            }
        });

        return subDecor;
    }
```





## 总结

通过XML的Pull解析方式获取View的标签（这个在主线程是耗时的）

通过标签以反射的方式来创建View对象（反射是比new对象是耗性能的）

如果是ViewGroup的话则会对子View遍历并重复以上步骤，然后add到父View中











动态换肤离不开LayoutInflater与Factory（Factory2）