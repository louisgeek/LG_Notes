> 源码基于 Android 11 API 30

## LayoutInflater 简介

- android.view.LayoutInflater （Inflater 充气者）
- 布局解析器，把 xml 布局文件解析后动态生成 View
- 是一个抽象类
- 是一个系统服务，服务名称 Context.LAYOUT_INFLATER_SERVICE = "layout_inflater"



## 获取 LayoutInflater 实例方法

### 1 Context#getSystemService

```java
LayoutInflater LayoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

### 2 LayoutInflater#from

```java
public static LayoutInflater from(Context context) {
//就是调用了 Context#getSystemService
LayoutInflater LayoutInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    	//做了判空
        if (LayoutInflater == null) {
            throw new AssertionError("LayoutInflater not found.");
        }
        return LayoutInflater;
}
```

### 3 Activity#getLayoutInflater

```java
private Window mWindow;
public Window getWindow() {
        return mWindow;
    }
@NonNull
public LayoutInflater getLayoutInflater() {
    //PhoneWindow 是 mWindow 的唯一继承实现类 
   return getWindow().getLayoutInflater();
}
```

- PhoneWindow#getLayoutInflater

```java
private LayoutInflater mLayoutInflater;
public PhoneWindow(Context context) {
        super(context);
        //就是调用了 LayoutInflater#from
        mLayoutInflater = LayoutInflater.from(context);
        mRenderShadowsInCompositor = Settings.Global.getInt(context.getContentResolver(),
                DEVELOPMENT_RENDER_SHADOWS_IN_COMPOSITOR, 1) != 0;
}
@Override
public LayoutInflater getLayoutInflater() {
    //PhoneWindow 初始化时候赋值
    return mLayoutInflater;
}
```



## inflate 解析方法

### 1 View#inflate

```java
public static View inflate(Context context, @LayoutRes int resource, ViewGroup root) {
        LayoutInflater factory = LayoutInflater.from(context);
        return factory.inflate(resource, root);
}
```

- 其实就是调用了 LayoutInflater.from(context).inflate(resource, root) 方法
- 解析布局生成 View 加入 ViewGroup 里面

##### View.inflate 方式的注意事项

- root == null 成立，那么 attachToRoot = false
- root  != null 成立，那么 attachToRoot = true

所以不适合在列表的 Adapter 里使用，因为它会自己添加 child view ，做不到 root !=null 同时 attachToRoot = false 的情况

所以也不合适在 Fragment#onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)  方法里使用，因为 FragmentManager 里会执行一次 container.addView



### 2 LayoutInflater#inflate

- Android 内置了 org.xmlpull 组件，用 pull 方式处理 xml

```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
        return inflate(resource, root, root != null);
}
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                  + Integer.toHexString(resource) + ")");
        }
    	//尝试从预编译缓存中解析
        View view = tryInflatePrecompiled(resource, res, root, attachToRoot);
        if (view != null) {
            return view;
        }
    	//XmlBlock#newParser 创建 XmlBlock$Parser 返回
        XmlResourceParser parser = res.getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
 }
//org.xmlpull.v1.XmlPullParser
public View inflate(XmlPullParser parser, @Nullable ViewGroup root) {
        return inflate(parser, root, root != null);
}
//org.xmlpull.v1.XmlPullParser
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            //默认是传入的 root
            View result = root;
    
            try {
                //移动到开始标签
                advanceToRootNode(parser);
                //比如 androidx.constraintlayout.widget.ConstraintLayout
                final String name = parser.getName();
    
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }
    			//根布局是 merge ，意味着必须要有父容器，所以下面判断不满足条件就抛异常
                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }
    				//
                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    //创建 xml 的根 View
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);
    
                    ViewGroup.LayoutParams params = null;
    
                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        // new LayoutParams(getContext(), attrs)
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }
                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }
    				//解析 xml 布局中的子 View
                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);
                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }
                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }
                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }
            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(
                        getParserStateDescription(inflaterContext, attrs)
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;
    
                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }
            //
            return result;
        }
    }
```

- root == null || attachToRoot == false 返回 xml 解析出的 temp 视图
- root != null && attachToRoot == true 返回传入的 root 视图，进行了 root.addView(temp, params) ，addView 方法里进行了 temp .setLayoutParams(params) 
- root != null && attachToRoot == false 时 xml 解析出的 temp 视图 ，此时需要 temp .setLayoutParams(params)

由此可以得出当 root 为空时，xml 最外层布局的属性会失效，因为压根没有设置参数嘛！



#### tryInflatePrecompiled 

- Android 10 API 29 中新增特征，暂不深究
- 采用预编译方式减少 xml 解析时间

```java
 private @Nullable
    View tryInflatePrecompiled(@LayoutRes int resource, Resources res, @Nullable ViewGroup root,
        boolean attachToRoot) {
        if (!mUseCompiledView) {
            //不使用预编译，直接返回 null
            return null;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate (precompiled)");
        // Try to inflate using a precompiled layout.
     	//包名，比如 com.louisgeek.xxx
        String pkg = res.getResourcePackageName(resource);
     	//layout名称，比如 activity_main
        String layout = res.getResourceEntryName(resource);
		//通过反射创建对应的 View
        try {
            Class clazz = Class.forName("" + pkg + ".CompiledView", false, mPrecompiledClassLoader);
            Method inflater = clazz.getMethod(layout, Context.class, int.class);
            View view = (View) inflater.invoke(null, mContext, resource);

            if (view != null && root != null) {
                //解析属性信息设置到上面生成的 view 上
                // We were able to use the precompiled inflater, but now we need to do some work to
                // attach the view to the root correctly.
                XmlResourceParser parser = res.getLayout(resource);
                try {
                    AttributeSet attrs = Xml.asAttributeSet(parser);
                    advanceToRootNode(parser);
                    ViewGroup.LayoutParams params = root.generateLayoutParams(attrs);
					//是否添加到 root，以及设置布局参数
                    if (attachToRoot) {
                        root.addView(view, params);
                    } else {
                        view.setLayoutParams(params);
                    }
                } finally {
                    parser.close();
                }
            }

            return view;
        } catch (Throwable e) {
            if (DEBUG) {
                Log.e(TAG, "Failed to use precompiled view", e);
            }
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
        return null;
    }

```

- 通过反射来生成对应 View
- mUseCompiledView = true 的情况，现仅供内部测试使用，其他情况默认 false

```java
  protected LayoutInflater(Context context) {
          mContext = context;
      	//无参方法默认 mUseCompiledView = false
          initPrecompiledViews();
  }
  
  protected LayoutInflater(LayoutInflater original, Context newContext) {
          mContext = newContext;
          mFactory = original.mFactory;
          mFactory2 = original.mFactory2;
          mPrivateFactory = original.mPrivateFactory;
          setFilter(original.mFilter);
     		 //无参方法默认 mUseCompiledView = false
          initPrecompiledViews();
  }  
   /**
     * @hide for use by CTS tests
     */
  @TestApi
  public void setPrecompiledLayoutsEnabledForTesting(boolean enablePrecompiledLayouts) {
          initPrecompiledViews(enablePrecompiledLayouts);
  }
  //无参方法默认 mUseCompiledView = false
   private void initPrecompiledViews() {
       // Precompiled layouts are not supported in this release.
       boolean enabled = false;
      initPrecompiledViews(enabled);
  }
  private void initPrecompiledViews(boolean enablePrecompiledViews) {
          mUseCompiledView = enablePrecompiledViews;
  
          if (!mUseCompiledView) {
              mPrecompiledClassLoader = null;
              return;
          }
          // Make sure the application allows code generation
          ApplicationInfo appInfo = mContext.getApplicationInfo();
          if (appInfo.isEmbeddedDexUsed() || appInfo.isPrivilegedApp()) {
              mUseCompiledView = false;
              return;
          }
          // Try to load the precompiled layout file.
          try {
              mPrecompiledClassLoader = mContext.getClassLoader();
              String dexFile = mContext.getCodeCacheDir() + COMPILED_VIEW_DEX_FILE_NAME;
              if (new File(dexFile).exists()) {
                  mPrecompiledClassLoader = new PathClassLoader(dexFile, mPrecompiledClassLoader);
              } else {
                  // If the precompiled layout file doesn't exist, then disable precompiled
                  // layouts.
                  mUseCompiledView = false;
              }
          } catch (Throwable e) {
              if (DEBUG) {
                  Log.e(TAG, "Failed to initialized precompiled views layouts", e);
              }
              mUseCompiledView = false;
          }
          if (!mUseCompiledView) {
              mPrecompiledClassLoader = null;
          }
      }
```

  



#### Resources#getLayout

- android.content.res.Resources

```java
@NonNull
public XmlResourceParser getLayout(@LayoutRes int id) throws NotFoundException {
        return loadXmlResourceParser(id, "layout");
}
@NonNull
XmlResourceParser loadXmlResourceParser(@AnyRes int id, @NonNull String type)
            throws NotFoundException {
    	//获取临时 TypedValue，内部有缓存机制
        final TypedValue value = obtainTempTypedValue();
        try {
            final ResourcesImpl impl = mResourcesImpl;
            impl.getValue(id, value, true);
            if (value.type == TypedValue.TYPE_STRING) {
                //
                return loadXmlResourceParser(value.string.toString(), id,
                        value.assetCookie, type);
            }
            throw new NotFoundException("Resource ID #0x" + Integer.toHexString(id)
                    + " type #0x" + Integer.toHexString(value.type) + " is not valid");
        } finally {
            releaseTempTypedValue(value);
        }
}
@NonNull
XmlResourceParser loadXmlResourceParser(String file, int id, int assetCookie,
                                            String type) throws NotFoundException {
        return mResourcesImpl.loadXmlResourceParser(file, id, assetCookie, type);
}
```



##### XmlResourceParser

- android.content.res.XmlResourceParser 

```java
/**
 * The XML parsing interface returned for an XML resource.  This is a standard
 * {@link XmlPullParser} interface but also extends {@link AttributeSet} and
 * adds an additional {@link #close()} method for the client to indicate when
 * it is done reading the resource.
 */
//org.xmlpull.v1.XmlPullParser
public interface XmlResourceParser extends XmlPullParser, AttributeSet, AutoCloseable {
    String getAttributeNamespace (int index);

    /**
     * Close this parser. Calls on the interface are no longer valid after this call.
     */
    public void close();
}
```



##### ResourcesImpl#loadXmlResourceParser

```java
/** Size of the cyclical cache used to map XML files to blocks. */
private static final int XML_BLOCK_CACHE_SIZE = 4;

// Cyclical cache used for recently-accessed XML files.
private int mLastCachedXmlBlockIndex = -1;
private final int[] mCachedXmlBlockCookies = new int[XML_BLOCK_CACHE_SIZE];
private final String[] mCachedXmlBlockFiles = new String[XML_BLOCK_CACHE_SIZE];
private final XmlBlock[] mCachedXmlBlocks = new XmlBlock[XML_BLOCK_CACHE_SIZE];

final AssetManager mAssets;
/**
     * Loads an XML parser for the specified file.
     *
     * @param file the path for the XML file to parse
     * @param id the resource identifier for the file
     * @param assetCookie the asset cookie for the file
     * @param type the type of resource (used for logging)
     * @return a parser for the specified XML file
     * @throws NotFoundException if the file could not be loaded
     */
    @NonNull
    XmlResourceParser loadXmlResourceParser(@NonNull String file, @AnyRes int id, int assetCookie,
            @NonNull String type)
            throws NotFoundException {
        if (id != 0) {
            try {
                synchronized (mCachedXmlBlocks) {
                    final int[] cachedXmlBlockCookies = mCachedXmlBlockCookies;
                    final String[] cachedXmlBlockFiles = mCachedXmlBlockFiles;
                    final XmlBlock[] cachedXmlBlocks = mCachedXmlBlocks;
                    // First see if this block is in our cache.
                    //先查找缓存
                    final int num = cachedXmlBlockFiles.length;
                    for (int i = 0; i < num; i++) {
                        if (cachedXmlBlockCookies[i] == assetCookie && cachedXmlBlockFiles[i] != null
                                && cachedXmlBlockFiles[i].equals(file)) {
                            return cachedXmlBlocks[i].newParser(id);
                        }
                    }

                    // Not in the cache, create a new block and put it at
                    // the next slot in the cache.
                    //无缓存情况下
                    final XmlBlock block = mAssets.openXmlBlockAsset(assetCookie, file);
                    if (block != null) {
                        final int pos = (mLastCachedXmlBlockIndex + 1) % num;
                        mLastCachedXmlBlockIndex = pos;
                        final XmlBlock oldBlock = cachedXmlBlocks[pos];
                        if (oldBlock != null) {
                            oldBlock.close();
                        }
                        cachedXmlBlockCookies[pos] = assetCookie;
                        cachedXmlBlockFiles[pos] = file;
                        cachedXmlBlocks[pos] = block;
                        //实例化 XmlBlock$Parser 进行返回
                        return block.newParser(id);
                    }
                }
            } catch (Exception e) {
                final NotFoundException rnf = new NotFoundException("File " + file
                        + " from xml type " + type + " resource ID #0x" + Integer.toHexString(id));
                rnf.initCause(e);
                throw rnf;
            }
        }

        throw new NotFoundException("File " + file + " from xml type " + type + " resource ID #0x"
                + Integer.toHexString(id));
    }
```

- 实际返回了一个 XmlBlock$Parser

- XmlBlock$Parser 实现了 XmlResourceParser 接口

  

#### Xml#asAttributeSet

- android.util.Xml

- android.util.AttributeSet

```java
//上文传入的是 XmlResourceParser 类型 
public static AttributeSet asAttributeSet(XmlPullParser parser) {
     //XmlBlock$Parser implements XmlResourceParser
     //XmlResourceParser extends XmlPullParser, AttributeSet
        return (parser instanceof AttributeSet)
                ? (AttributeSet) parser
                : new XmlPullAttributes(parser);
}
```



#### advanceToRootNode

- 通过 parser.next() 移动

```java
  private void advanceToRootNode(XmlPullParser parser)
        throws InflateException, IOException, XmlPullParserException {
        // Look for the root node.
        //存放类型
        int type;
    	//找寻根节点的开始标签
        while ((type = parser.next()) != XmlPullParser.START_TAG &&
            type != XmlPullParser.END_DOCUMENT) {
            // Empty
        }
		//如果没找到就抛异常
        if (type != XmlPullParser.START_TAG) {
            throw new InflateException(parser.getPositionDescription()
                + ": No start tag found!");
        }
    }
```



#### rInflate

- XmlPullParser.START_DOCUMENT 文档开始标记，代表未读取任何内容

- XmlPullParser.END_DOCUMENT 文档结束标记
- XmlPullParser.START_TAG 开始标签
- XmlPullParser.END_TAG 结束标签
- XmlPullParser.TEXT 文本

```java
void rInflate(XmlPullParser parser, View parent, Context context,
            AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {
		//xml 的深度
        final int depth = parser.getDepth();
        int type;
        boolean pendingRequestFocus = false;
		//需要满足不是结束标签的情况
        while (
            	( 
                    (type = parser.next()) != XmlPullParser.END_TAG 
                 	|| parser.getDepth() > depth
           		)
               && type != XmlPullParser.END_DOCUMENT
        ) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }
			//比如 TextView
            final String name = parser.getName();
			
            if (TAG_REQUEST_FOCUS.equals(name)) {
                //判断 <requestFocus />
                pendingRequestFocus = true;
                consumeChildElements(parser);
            } else if (TAG_TAG.equals(name)) {
                //判断 <tag />
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
                //判断 <include />
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                //判断 <merge />
                throw new InflateException("<merge /> must be the root element");
            } else {
                //
                final View view = createViewFromTag(parent, name, context, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                //
                rInflateChildren(parser, view, attrs, true);
                viewGroup.addView(view, params);
            }
        }

        if (pendingRequestFocus) {
            parent.restoreDefaultFocus();
        }

        if (finishInflate) {
            parent.onFinishInflate();
        }
    }
```

- rInflate 调用了 rInflateChildren

#### createViewFromTag

```java
private View createViewFromTag(View parent, String name, Context context, AttributeSet attrs) {
        return createViewFromTag(parent, name, context, attrs, false);
}
View createViewFromTag(View parent, String name, Context context, AttributeSet attrs,
            boolean ignoreThemeAttr) {
        if (name.equals("view")) {
            name = attrs.getAttributeValue(null, "class");
        }

        // Apply a theme wrapper, if allowed and one is specified.
        if (!ignoreThemeAttr) {
            final TypedArray ta = context.obtainStyledAttributes(attrs, ATTRS_THEME);
            final int themeResId = ta.getResourceId(0, 0);
            if (themeResId != 0) {
                context = new ContextThemeWrapper(context, themeResId);
            }
            ta.recycle();
        }

        try {
            //通过 LayoutInflater$Factory2、LayoutInflater$Factory 两个接口执行 onCreateView 方法
            View view = tryCreateView(parent, name, context, attrs);

            if (view == null) {
                final Object lastContext = mConstructorArgs[0];
                mConstructorArgs[0] = context;
                try {
                    if (-1 == name.indexOf('.')) {
                        //最终也会调用 createView 方法
                        view = onCreateView(context, parent, name, attrs);
                    } else {
                        view = createView(context, name, null, attrs);
                    }
                } finally {
                    mConstructorArgs[0] = lastContext;
                }
            }

            return view;
        } catch (InflateException e) {
            throw e;

        } catch (ClassNotFoundException e) {
            final InflateException ie = new InflateException(
                    getParserStateDescription(context, attrs)
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;

        } catch (Exception e) {
            final InflateException ie = new InflateException(
                    getParserStateDescription(context, attrs)
                    + ": Error inflating class " + name, e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        }
    }
```

##### LayoutInflater#tryCreateView

```java
 @Nullable
    public final View tryCreateView(@Nullable View parent, @NonNull String name,
        @NonNull Context context,
        @NonNull AttributeSet attrs) {
        if (name.equals(TAG_1995)) {
            // Let's party like it's 1995!
            return new BlinkLayout(context, attrs);
        }
        View view;
        if (mFactory2 != null) {
            view = mFactory2.onCreateView(parent, name, context, attrs);
        } else if (mFactory != null) {
            view = mFactory.onCreateView(name, context, attrs);
        } else {
            view = null;
        }
        if (view == null && mPrivateFactory != null) {
            view = mPrivateFactory.onCreateView(parent, name, context, attrs);
        }
        return view;
 }
```

- 涉及换肤知识点

##### LayoutInflater#onCreateView

```java
@Nullable
    public View onCreateView(@NonNull Context viewContext, @Nullable View parent,
            @NonNull String name, @Nullable AttributeSet attrs)
            throws ClassNotFoundException {
        return onCreateView(parent, name, attrs);
}
 protected View onCreateView(View parent, String name, AttributeSet attrs)
            throws ClassNotFoundException {
        return onCreateView(name, attrs);
}
 protected View onCreateView(String name, AttributeSet attrs)
            throws ClassNotFoundException {
        return createView(name, "android.view.", attrs);
}
```

##### LayoutInflater#createView

```java
public final View createView(String name, String prefix, AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        Context context = (Context) mConstructorArgs[0];
        if (context == null) {
            context = mContext;
        }
        return createView(context, name, prefix, attrs);
}
@Nullable
    public final View createView(@NonNull Context viewContext, @NonNull String name,
            @Nullable String prefix, @Nullable AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        Objects.requireNonNull(viewContext);
        Objects.requireNonNull(name);
        Constructor<? extends View> constructor = sConstructorMap.get(name);
        if (constructor != null && !verifyClassLoader(constructor)) {
            constructor = null;
            sConstructorMap.remove(name);
        }
        Class<? extends View> clazz = null;

        try {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, name);

            if (constructor == null) {
                // Class not found in the cache, see if it's real, and try to add it
                clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                        mContext.getClassLoader()).asSubclass(View.class);

                if (mFilter != null && clazz != null) {
                    boolean allowed = mFilter.onLoadClass(clazz);
                    if (!allowed) {
                        failNotAllowed(name, prefix, viewContext, attrs);
                    }
                }
                constructor = clazz.getConstructor(mConstructorSignature);
                constructor.setAccessible(true);
                sConstructorMap.put(name, constructor);
            } else {
                // If we have a filter, apply it to cached constructor
                if (mFilter != null) {
                    // Have we seen this name before?
                    Boolean allowedState = mFilterMap.get(name);
                    if (allowedState == null) {
                        // New class -- remember whether it is allowed
                        clazz = Class.forName(prefix != null ? (prefix + name) : name, false,
                                mContext.getClassLoader()).asSubclass(View.class);

                        boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                        mFilterMap.put(name, allowed);
                        if (!allowed) {
                            failNotAllowed(name, prefix, viewContext, attrs);
                        }
                    } else if (allowedState.equals(Boolean.FALSE)) {
                        failNotAllowed(name, prefix, viewContext, attrs);
                    }
                }
            }

            Object lastContext = mConstructorArgs[0];
            mConstructorArgs[0] = viewContext;
            Object[] args = mConstructorArgs;
            args[1] = attrs;

            try {
                final View view = constructor.newInstance(args);
                if (view instanceof ViewStub) {
                    // Use the same context when inflating ViewStub later.
                    final ViewStub viewStub = (ViewStub) view;
                    viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
                }
                return view;
            } finally {
                mConstructorArgs[0] = lastContext;
            }
        } catch (NoSuchMethodException e) {
            final InflateException ie = new InflateException(
                    getParserStateDescription(viewContext, attrs)
                    + ": Error inflating class " + (prefix != null ? (prefix + name) : name), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;

        } catch (ClassCastException e) {
            // If loaded class is not a View subclass
            final InflateException ie = new InflateException(
                    getParserStateDescription(viewContext, attrs)
                    + ": Class is not a View " + (prefix != null ? (prefix + name) : name), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } catch (ClassNotFoundException e) {
            // If loadClass fails, we should propagate the exception.
            throw e;
        } catch (Exception e) {
            final InflateException ie = new InflateException(
                    getParserStateDescription(viewContext, attrs) + ": Error inflating class "
                            + (clazz == null ? "<unknown>" : clazz.getName()), e);
            ie.setStackTrace(EMPTY_STACK_TRACE);
            throw ie;
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
}
```

- 反射 Class#forName

- ClassLoader

  

#### rInflateChildren

```java
 final void rInflateChildren(XmlPullParser parser, View parent, AttributeSet attrs,
            boolean finishInflate) throws XmlPullParserException, IOException {
        rInflate(parser, parent, parent.getContext(), attrs, finishInflate);
}
```

- rInflateChildren 内部也调用了  rInflate 方法，所以就形成了递归调用











AppCompatDelegateImpl#onCreateView

```java
@Override
public final View onCreateView(View parent, String name, Context context, AttributeSet attrs) {
    return createView(parent, name, context, attrs);
}
 @Override
    public View createView(View parent, final String name, @NonNull Context context,
            @NonNull AttributeSet attrs) {
        if (mAppCompatViewInflater == null) {
            TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);
            String viewInflaterClassName =
                    a.getString(R.styleable.AppCompatTheme_viewInflaterClass);
            if (viewInflaterClassName == null) {
                // Set to null (the default in all AppCompat themes). Create the base inflater
                // (no reflection)
                mAppCompatViewInflater = new AppCompatViewInflater();
            } else {
                try {
                    Class<?> viewInflaterClass = Class.forName(viewInflaterClassName);
                    mAppCompatViewInflater =
                            (AppCompatViewInflater) viewInflaterClass.getDeclaredConstructor()
                                    .newInstance();
                } catch (Throwable t) {
                    Log.i(TAG, "Failed to instantiate custom view inflater "
                            + viewInflaterClassName + ". Falling back to default.", t);
                    mAppCompatViewInflater = new AppCompatViewInflater();
                }
            }
        }

        boolean inheritContext = false;
        if (IS_PRE_LOLLIPOP) {
            inheritContext = (attrs instanceof XmlPullParser)
                    // If we have a XmlPullParser, we can detect where we are in the layout
                    ? ((XmlPullParser) attrs).getDepth() > 1
                    // Otherwise we have to use the old heuristic
                    : shouldInheritContext((ViewParent) parent);
        }
		//createView
        return mAppCompatViewInflater.createView(parent, name, context, attrs, inheritContext,
                IS_PRE_LOLLIPOP, /* Only read android:theme pre-L (L+ handles this anyway) */
                true, /* Read read app:theme as a fallback at all times for legacy reasons */
                VectorEnabledTintResources.shouldBeUsed() /* Only tint wrap the context if enabled */
        );
}
```

AppCompatViewInflater#createView

```java
final View createView(View parent, final String name, @NonNull Context context,
            @NonNull AttributeSet attrs, boolean inheritContext,
            boolean readAndroidTheme, boolean readAppTheme, boolean wrapContext) {
        final Context originalContext = context;

        // We can emulate Lollipop's android:theme attribute propagating down the view hierarchy
        // by using the parent's context
        if (inheritContext && parent != null) {
            context = parent.getContext();
        }
        if (readAndroidTheme || readAppTheme) {
            // We then apply the theme on the context, if specified
            context = themifyContext(context, attrs, readAndroidTheme, readAppTheme);
        }
        if (wrapContext) {
            context = TintContextWrapper.wrap(context);
        }

        View view = null;

        // We need to 'inject' our tint aware Views in place of the standard framework versions
        switch (name) {
            case "TextView":
                view = createTextView(context, attrs);
                verifyNotNull(view, name);
                break;
            case "ImageView":
                view = createImageView(context, attrs);
                verifyNotNull(view, name);
                break;
            case "Button":
                view = createButton(context, attrs);
                verifyNotNull(view, name);
                break;
            case "EditText":
                view = createEditText(context, attrs);
                verifyNotNull(view, name);
                break;
            case "Spinner":
                view = createSpinner(context, attrs);
                verifyNotNull(view, name);
                break;
            case "ImageButton":
                view = createImageButton(context, attrs);
                verifyNotNull(view, name);
                break;
            case "CheckBox":
                view = createCheckBox(context, attrs);
                verifyNotNull(view, name);
                break;
            case "RadioButton":
                view = createRadioButton(context, attrs);
                verifyNotNull(view, name);
                break;
            case "CheckedTextView":
                view = createCheckedTextView(context, attrs);
                verifyNotNull(view, name);
                break;
            case "AutoCompleteTextView":
                view = createAutoCompleteTextView(context, attrs);
                verifyNotNull(view, name);
                break;
            case "MultiAutoCompleteTextView":
                view = createMultiAutoCompleteTextView(context, attrs);
                verifyNotNull(view, name);
                break;
            case "RatingBar":
                view = createRatingBar(context, attrs);
                verifyNotNull(view, name);
                break;
            case "SeekBar":
                view = createSeekBar(context, attrs);
                verifyNotNull(view, name);
                break;
            case "ToggleButton":
                view = createToggleButton(context, attrs);
                verifyNotNull(view, name);
                break;
            default:
                // The fallback that allows extending class to take over view inflation
                // for other tags. Note that we don't check that the result is not-null.
                // That allows the custom inflater path to fall back on the default one
                // later in this method.
                view = createView(context, name, attrs);
        }

        if (view == null && originalContext != context) {
            // If the original context does not equal our themed context, then we need to manually
            // inflate it using the name so that android:theme takes effect.
            view = createViewFromTag(context, name, attrs);
        }

        if (view != null) {
            // If we have created a view, check its android:onClick
            checkOnClickListener(view, attrs);
        }

        return view;
}
```

- 解析 xml ，判断 view 类型，实例化指定控件对象









## 总结

LayoutInflater 布局解析器是一个系统服务，作用是解析布局文件后动态生成 View

三种获取实例的方法最终都是通过 Context#getSystemService 方法实现的，里面涉及到单例模式

Activity#setContentView 方法内部也用到了 LayoutInflater 进行布局解析

LayoutInflater 内部采用了 org.xmlpull 解析器实现 xml 解析





- LayoutInflater 布局解析器进行 inflate 解析的流程是怎么样的？

1 LayoutInflater 有四个方法重载，其中两个针对 xml 布局文件，另外两个是针对现成的 XmlResourceParser

2 如果是 xml 布局文件，可以通过 Resources#getLayout 把 xml 布局文件解析加载得到 XmlResourceParser

3 因为 XmlResourceParser 包含属性信息，所以利用 Xml#asAttributeSet 将其转化成 AttributeSet 供后续使用

4 通过 advanceToRootNode 方法找到根标签，如遇到的不是 <merge/> 标签就通过 createViewFromTag 方法把根标签对应类通过反射的方式创建出来，然后调用 rInflateChildren 方法，而它就是调用了 rInflate 方法，从而实现了递归，而 rInflate 内部也调用了 createViewFromTag ，所以用递归调用的方式把子 View 一个个创建出来并通过 addView 加入其父 View 中，如遇到 <merge/> 标签就直接调用 rInflate 先进行处理一次，最终形成一个视图树

5 整个流程中还涉及到反射、ClassLoader、换肤等知识点可以展开



