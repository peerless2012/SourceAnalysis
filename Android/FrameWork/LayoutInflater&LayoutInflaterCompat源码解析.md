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