#LayoutInflater/LayoutInflaterCompat源码解析
***

## 0x00 简介
> Instantiates a layout XML file into its corresponding View objects. It is never used directly. Instead, use `getLayoutInflater()` or `getSystemService(Class)` to retrieve a standard `LayoutInflater` instance that is already hooked up to the current context and correctly configured for the device you are running on. For example:

> `LayoutInflater inflater = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);`

> To create a new `LayoutInflater` with an additional `LayoutInflater.Factory` for your own views, you can use `cloneInContext(Context)` to clone an existing `ViewFactory`, and then call `setFactory(LayoutInflater.Factory)` on it to include your Factory.

> For performance reasons, view inflation relies heavily on pre-processing of XML files that is done at build time. Therefore, it is not currently possible to use `LayoutInflater` with an `XmlPullParser` over a plain XML file at runtime; it only works with an `XmlPullParser` returned from a compiled resource (R.something file.)
> 实例化布局的XML文件到相应的视图对象。它从来没有被直接使用。相反，使用getLayoutInflater（）或getSystemService（Class）来检索已经迷上了当前上下文并正确配置为您正在运行的设备标准LayoutInflater实例。 例如：

> LayoutInflater吹气=（LayoutInflater）context.getSystemService（Context.LAYOUT_INFLATER_SERVICE）;

> 要为你自己的看法额外LayoutInflater.Factory创建一个新的LayoutInflater，您可以使用cloneInContext（上下文），以克隆现有的ViewFactory，然后调用setFactory（LayoutInflater.Factory）就可以了，包括你的工厂。

> 由于性能原因，查看通货膨胀在很大程度上依赖于被在编译的时候做的XML文件预处理。因此，目前还无法使用LayoutInflater超过在运行时一个纯XML文件中的XmlPullParser;它只能从一个编译的资源返回XmlPullParser（R.something文件。）


## 获取LayoutInflater的三种方式：

1. `LayoutInflater inflater = getLayoutInflater();//调用Activity的getLayoutInflater() `
2.  `LayoutInflater inflater = LayoutInflater.from(context);`
3.  `LayoutInflater inflater =  (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE); `

但是，这三种方式本质上还是一样的：

1. `getLayoutInflater()`调用的还是Activity中的方法：
	
		public LayoutInflater getLayoutInflater() {
	        return getWindow().getLayoutInflater();
	    }
	其中`Window`是一个抽象类，它的 **唯一** 实现类是`PhoneWindow`。

	_com/android/internal/policyPhoneWindow.java_

		public PhoneWindow(Context context) {
        	super(context);
        	mLayoutInflater = LayoutInflater.from(context);
   		}
		/**
	     * Return a LayoutInflater instance that can be used to inflate XML view layout
	     * resources for use in this Window.
	     *
	     * @return LayoutInflater The shared LayoutInflater.
	     */
	    @Override
	    public LayoutInflater getLayoutInflater() {
	        return mLayoutInflater;
	    }
	可以看到，通过·`Activity`的`getLayoutInflater()`最终调用的还是第二种方法。
	
2.  通过 `LayoutInflater.from(context);` 获取`LayoutInflater`对象。
	
	_android/view/LayoutInflater.java_

		/**
	     * Obtains the LayoutInflater from the given context.
	     */
	    public static LayoutInflater from(Context context) {
			//通过获取系统服务的方式获取到LayoutInflater实例对象
	        LayoutInflater LayoutInflater =
	                (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	        if (LayoutInflater == null) {
	            throw new AssertionError("LayoutInflater not found.");
	        }
	        return LayoutInflater;
	    }
	最终第2中方式调用的还是上述中第三种方式，获取LayoutInflater实例对象。

最终：不管以什么样的方式获取到LayoutInflater对象，最终都是[通过系统服务来获取实例对象](http://blog.csdn.net/chunqiuwei/article/details/50495686)。

## 主要方法
*  `public static LayoutInflater from(Context context);`
	> 获取LayoutInflater实例化对象。
*  `public void setFactory(Factory factory);`
	> 设置使用当前LayoutInflater创建View的自定义实例化工厂。
*  `public void setFactory2(Factory2 factory);`
	> 同上，区别是工厂2多了对实例化View的时候Parent的支持。
*  `public void setFilter(Filter filter);`
	> 给当前LayoutInflater设置过滤器，如果要被填充的View不被这个过滤器允许，则会抛出InflateException。这个过滤器会覆盖当前LayoutInflater之上的任何之前的过滤器。
*  `public View inflate(int resource, ViewGroup root);`
	> 把指定的布局资源填充成View，如果root不为空，则把填充的View挂载到root上。
*  `public View inflate(XmlPullParser parser, ViewGroup root);`
	> 通过布局资源的XML解析器把布局资源填充成View，如果root不为空，则把填充的View挂载到root上。
*  `public View inflate(int resource, ViewGroup root, boolean attachToRoot);`
	> 同上
*  `public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot);`
	> 同上
*  `public final View createView(String name, String prefix, AttributeSet attrs);`
	> 同上
*  `protected View onCreateView(String name, AttributeSet attrs);`
	> 同上
*  `protected View onCreateView(View parent, String name, AttributeSet attrs);`
	> 同上
*  `View createViewFromTag(View parent, String name, AttributeSet attrs);`
	> 同上
*  `void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs,
            boolean finishInflate);`
	> 同上


## 加载流程
通过 __获取LayoutInflater的三种方式__ 我们知道，通过布局文件填充成View对象最终调用的是下面两个方法

	/************************字段定义区**********************/
	final Object[] mConstructorArgs = new Object[2];

    static final Class<?>[] mConstructorSignature = new Class[] {
            Context.class, AttributeSet.class};	

	/************************方法区**********************/
	public View inflate(int resource, ViewGroup root, boolean attachToRoot) {
        if (DEBUG) System.out.println("INFLATING from resource: " + resource);
		//获取资源id布局的xml解析器
        XmlResourceParser parser = getContext().getResources().getLayout(resource);
        try {
            return inflate(parser, root, attachToRoot);
        } finally {
            parser.close();
        }
    } 

	//最终调用的是此方法
	public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

			//判断parser是否是AttributeSet，如果不是则用XmlPullAttributes去包装一下。
            final AttributeSet attrs = Xml.asAttributeSet(parser);
			//保存上次的Context
            Context lastContext = (Context)mConstructorArgs[0];
            mConstructorArgs[0] = mContext;
            View result = root;

            try {
                // 查找开始标签
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }

				//如果没找到有效的开始标签
                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }
				
				//获取控件的名称
                final String name = parser.getName();
                
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

				// 如果根节点是“merge”标签
                if (TAG_MERGE.equals(name)) {
					// 根节点为空或者不添加到根节点上，则抛出异常。因为“merge”标签必须是要被添加到父节点上的，不能独立存在。
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, attrs, false);
                } else {
					// 当前xml的根节点的View
                    View temp;
					// 如果根标签是“1995”，则创建一个“BlinkLayout”（其实就是一个FrameLayout）
					// ps：这个没见过在哪里用到了。
                    if (TAG_1995.equals(name)) {
                        temp = new BlinkLayout(mContext, attrs);
                    } else {
						// 通过标签名、父View、属性创建View。
                        temp = createViewFromTag(root, name, attrs);
                    }

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // 创建父布局类型的LayoutParams参数
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
							// 如果不把填充的View 关联在父View上，则把父View的LayoutParams参数设置给它
                            // 如果把填充的View关联在父View上，则会走下面addView的逻辑
							temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }
                    // 填充跟节点View下面的子View。
                    rInflate(parser, temp, attrs, true);
                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

					// Google建议关联所有找到的View
                    // 如果根节点不为null，并且需要把填充的View关联到父View上，则使用addView方法把布局填充成的View树添加到父View上。
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

					// 决定返回的rootView还是xml中的跟节点的View。
                    // 通过对这两个参数的分析，发现上面的方法和下面的方法肯定会执行一个
					// 如果走上面的判断，则返回的就是传入的父View；
					// 如果走下面的判断，则返回的是xml填充成的View；
					// 注意两者的区别。
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                InflateException ex = new InflateException(e.getMessage());
                ex.initCause(e);
                throw ex;
            } catch (IOException e) {
                InflateException ex = new InflateException(
                        parser.getPositionDescription()
                        + ": " + e.getMessage());
                ex.initCause(e);
                throw ex;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;
            }

            Trace.traceEnd(Trace.TRACE_TAG_VIEW);

            return result;
        }
    }
	
	/*
     * 缺省方法可见性，好让BridgeInflater能重写它。
     * 根据父View、View名称、属性创建View。
     */
    View createViewFromTag(View parent, String name, AttributeSet attrs) {
		// 如果是View标签，则用class指向的类的完整名称来替换当前名称。(我们都知道，用Fragment的时候，可指定 class="Fragment完整路径名"，其他widget控件也类似)
        if (name.equals("view")) {
            name = attrs.getAttributeValue(null, "class");
        }

        if (DEBUG) System.out.println("******** Creating view: " + name);

        try {
            View view;
			// 尝试通过 mFactory2 或者 mFactory来创建View，这两个是通过setFactory和setFactory2来设置的。
            if (mFactory2 != null) view = mFactory2.onCreateView(parent, name, mContext, attrs);
            else if (mFactory != null) view = mFactory.onCreateView(name, mContext, attrs);
            else view = null;

			// 如果没有设置自定义工厂并且LayoutInflater本身私有的View工厂不为空，则用私有View工厂创建View。
            if (view == null && mPrivateFactory != null) {
                view = mPrivateFactory.onCreateView(parent, name, mContext, attrs);
            }
            
			// 如果View还为空，也即没有工厂，或者工厂未能正确创建View，则尝试通过
            if (view == null) {
                if (-1 == name.indexOf('.')) {
					// 如果View标签中没有"."，则代表是系统的widget，则调用onCreateView，这个方法会通过"createView"方法创建View，不过前缀字段会自动补上"android.view."。
                    view = onCreateView(parent, name, attrs);
                } else {
					// 非系统控件，则name本身就是控件的完整路径名。
					//通过widget完整路径名以及属性创建View。
                    view = createView(name, null, attrs);
                }
            }

            if (DEBUG) System.out.println("Created view is: " + view);
            return view;

        } catch (InflateException e) {
            throw e;

        } catch (ClassNotFoundException e) {
            InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name);
            ie.initCause(e);
            throw ie;

        } catch (Exception e) {
            InflateException ie = new InflateException(attrs.getPositionDescription()
                    + ": Error inflating class " + name);
            ie.initCause(e);
            throw ie;
        }
    }