title: 记一次印象深刻的Bug追踪过程
date: 2018/01/02 18:03
comments: true
tags:
- Android
- Bug
- 前端
categories:
- Bug Fix
---

>问题现象：使用安卓手机以小程序的形式分享产品到微信，使用微信打开，产品详情数据无法显示。而使用iPhone分享到微信，却始终可以正常打开，这个时候所有的矛头都指向了安卓同学。

![小程序中打开，显示空白](http://upload-images.jianshu.io/upload_images/703764-cdf26125f909ddcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

逻辑设计说明：这里的分享数据来自H5接口，通过`addJavascriptInterface`自定义接口完成H5和Java端的数据传递，产品ID来自后台接口获取。

这个时候，安卓同学首先做出了响应，通过调试拿到了JS端的数据，以下是这位小陈同学的截图消息：

![Android调试结果](http://upload-images.jianshu.io/upload_images/703764-bd7a8a062dfab805.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

小陈同学这个时候把问题抛给了Web前端同学小徐，以为小徐传递了科学计数法的ID字符串。

![](http://upload-images.jianshu.io/upload_images/703764-4dcd326299b5e4d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大家看小陈同学的截图，图中的ID是使用字符串接收的，这个时候我已经完全排除问题出现在安卓端的可能性了。于是，我问小徐，H5有对参数进行处理吗？得到的答案如下：
![](http://upload-images.jianshu.io/upload_images/703764-a28fd422fa316bcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大家看到图中，我已经给出了确定的答案，认为问题来自于后台。因为，后台同学之前的确出现过对ID进行toInt处理最终转换为负数的情况。现在在传递时出现这种低级错误的概率应该也挺高的。这段话抛出去之后，团队炸开了锅，有同学认为大家在互相推诿...

![](http://upload-images.jianshu.io/upload_images/703764-4ec531bb958ae6be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实，还有很长的截图，这里没有展示出来。群里提到最多的一句话就是：`iOS没问题啊`。就连我们的运维同学以及UI设计同学都加入了“讨伐”队伍，种种迹象似乎都指向了安卓同学。这个时候，我们的安卓同学真是“哑巴吃黄连，有苦说不出”，心里的潜台词肯定是：我TM的就用`string`接收了一下，我招谁惹谁了我！

但其实出现这种不知所踪的情况，完全可以理解，大家大都集中在单一平台开发，对于其它环节的理解难免有偏差。其实，用常识来理解这个问题的话，的确后台的概率比较大，前端同学对ID进行运算处理的概率几乎为0，这一点即使是刚刚入行的新手也不太可能。而我一直苦等的后台同学却迟迟没有响应，我目前始终无法确定问题到底来自于后台还是Web前端。直到我终于看到了下面的截图。
![](http://upload-images.jianshu.io/upload_images/703764-fc7196333abaedbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候，我终于有九成的把握确定问题来自于Web前端了。可是，我知道我不能明说。前端同学已经在聊天记录中给出了证据，在Chrome的控制台打印出了正常的id值，到了安卓端却出现了异常。前端同学这个时候心里也有了一个定性结论，问题来自安卓端。这个时候，我只能亲自上场，而恰好我在外面，正在办理深圳户口，比较不便。于是，我微信给小陈发消息，嘱咐它把详情页的源码“爬”下来，我回来看看源码。

回到家的时候，我问小陈html源码是否已经“爬”了下来，他给我发来截图，我意识到前端使用了https协议，没法获取html源码。于是，我想了一个办法，在源码中嵌入一段代码，通过代码的形式获取WebView产品详情页的数据。这个方法果然奏效，不一会儿，小陈就发来了页面的html源码。

![](http://upload-images.jianshu.io/upload_images/703764-de788a95449192ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

哎哟，我的天哪！混淆后的代码简直不堪入目，不过还好，我可以搜索方法关键字`showShareView`。可是，很遗憾没有搜索到，事件的绑定被放到了JS代码中。在这段源码中，我注意到一个文件名已经被混淆的JS文件，我猜想代码应该就在这里。可是，怎样抓到具体的方法呢？

灵机一动！我之前在代码中让小陈把Debug权限开发给了H5，这次正好可以派上用场。可是，对于混淆后的代码，我心里依然有点打退堂鼓。

连上手机，在Chrome浏览器中输入chrome://inpsect，点击相应链接，非常顺利地进入了调试界面：

![](http://upload-images.jianshu.io/upload_images/703764-753ad81eaaf2318d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在控制台的Source中，我通过关键词搜索找到了混淆后的JS代码片段，在方法名前面增加了一个断点，等调试到底方法位置的时候。这个时候已经获取到了JS的上下文，直接通过`this.gid`打印出了当前产品ID信息，居然是一个非常正常的整型数字。大家注意，这已经是一个在安卓端出问题的产品了，在JS端居然显示是正常的。这个时候，我的大脑非常转动，我的第一感觉应该是`webkit`内核看到接收的字符串全是数字做了”自以为是“的转换。于是，我给出了团队如下的答案：
![](http://upload-images.jianshu.io/upload_images/703764-02b3c3a21e2b0569.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了进一步确定我的猜想，我让小陈写了一个简单的Demo，通过JS接口传递一个非常大的数字字符串给Java端，看接收是否异常。不一会儿，我就得到了答案：
![](http://upload-images.jianshu.io/upload_images/703764-64ba7132c99d0a56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，我终于基本确定问题的原因了！
猜测：JS在传递数据给安卓端的时候，应该是使用了基本数据类型。而`webkit`内核在处理的时候可能是以JS端数据类型为准，在传递到Java端时候做了转换。

为了验证这个猜想，我使用`typeof`打印id的数据类型，得到了如下结果：

![](http://upload-images.jianshu.io/upload_images/703764-7615a576bf35a125.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

于是，我告诉小徐，问题来自于你没有传递正确的数据类型给安卓端。其实这是比较危险的，不同CPU可以容纳的最大整型值是不一样的。如果iOS端和安卓处理一致，也是以JS端数据类型为准，只不过iOS的CPU字节宽度较大，恰好在iPhone高端机型上面没有出现而低端机型出现的话。其实问题依然存在，而如果iOS的确是以Native端数据类型为准。这就根本不是一个问题。但答案虽然给了团队，可是小徐仍然一脸狐疑，没有经验的CTO也是跟着一脸狐疑，加上解决问题的时间较长。小徐在发布更新的时候也遇到了问题，导致更新失败，问题持续，整个问题一直在持续。

这个时候，我告诉小徐，你发布更新后先别着急，确定更新成功后再告诉团队小伙伴。

一直到确定更新成功，我们再次尝试分享，问题终于引刃而解！

>问题虽然解决了，可是，安卓系统为什么要这样处理呢？为什么不能以Native端数据类型为准呢？带着这个疑问，我开始查看安卓源码。

阅读安卓源码是一个痛苦的过程，随着系统版本的升级，安卓系统的兼容性代码越来越多，这给阅读带来了极大的困难。加上安卓系统本身源码量巨大，阅读源码就像在一个巨大的森林中寻找宝藏一样。这个时候，其实你非常容易迷路，而我知道，只要我坚信我想要什么，就一定可以找到。

这里我们以`addJavascriptInterface`这个方法作为突破口，进入源码：
```
 public void addJavascriptInterface(Object object, String name) {
        checkThread();
        mProvider.addJavascriptInterface(object, name);
    }
```
额，mProvider是什么鬼？难道WebView只是一个傀儡，真正处理业务的其实是mProvider？是的，没错！WebView只不过是一个壳而已！可是，mProvider的实现到底是什么呢？带着这个疑问，我们看到了如下mProvider实例创建的方法：
```
  private void ensureProviderCreated() {
        checkThread();
        if (mProvider == null) {
            // As this can get called during the base class constructor chain, pass the minimum
            // number of dependencies here; the rest are deferred to init().
            mProvider = getFactory().createWebView(this, new PrivateAccess());
        }
    }

    private static WebViewFactoryProvider getFactory() {
        return WebViewFactory.getProvider();
    }
```

又出现了一个工厂方法，别怕，继续往下追踪：
getProvider方法较长，我们截取部分，看下面源码：
```
static WebViewFactoryProvider getProvider() {
        synchronized (sProviderLock) {
            // For now the main purpose of this function (and the factory abstraction) is to keep
            // us honest and minimize usage of WebView internals when binding the proxy.
            if (sProviderInstance != null) return sProviderInstance;

            final int uid = android.os.Process.myUid();
            if (uid == android.os.Process.ROOT_UID || uid == android.os.Process.SYSTEM_UID
                    || uid == android.os.Process.PHONE_UID || uid == android.os.Process.NFC_UID
                    || uid == android.os.Process.BLUETOOTH_UID) {
                throw new UnsupportedOperationException(
                        "For security reasons, WebView is not allowed in privileged processes");
            }

            StrictMode.ThreadPolicy oldPolicy = StrictMode.allowThreadDiskReads();
            Trace.traceBegin(Trace.TRACE_TAG_WEBVIEW, "WebViewFactory.getProvider()");
            try {
                Class<WebViewFactoryProvider> providerClass = getProviderClass();
                Method staticFactory = null;
                try {
                    staticFactory = providerClass.getMethod(
                        CHROMIUM_WEBVIEW_FACTORY_METHOD, WebViewDelegate.class);
                } catch (Exception e) {
                    if (DEBUG) {
                        Log.w(LOGTAG, "error instantiating provider with static factory method", e);
                    }
                }
```

这里的单用户检测，安全调用之类的代码就先忽略了。集中注意力看Provider实例创建的代码，大家可以看到，这里的创建其实通过反射调用创建的。这里有一个关键的方法`getProviderClass()`，这个方法可能获取到真正的Provider类对象，跟踪这个方法调用，我们看到了如下的调用过程：
`getProviderClass() -> getWebViewProviderClass `

```
 public static Class<WebViewFactoryProvider> getWebViewProviderClass(ClassLoader clazzLoader)
            throws ClassNotFoundException {
        return (Class<WebViewFactoryProvider>) Class.forName(CHROMIUM_WEBVIEW_FACTORY,
                true, clazzLoader);
    }
```

看到了吗？`CHROMIUM_WEBVIEW_FACTORY` 这才是真正的`WebViewFactoryProvider`类声明，跟进这个常量：
``` 
private static final String CHROMIUM_WEBVIEW_FACTORY = "com.android.webview.chromium.WebViewChromiumFactoryProviderForO";
```

从命名ForO来看，这个类恰好是用于最新版本Android系统`Oreo`的。没错，这里我们就从最新版本的源码入手，找到真正的问题”元凶“。

可是，这个代码在哪里呢？你搜索安卓源码，根本搜索不到该类，这是为什么呢？也许你已经猜到了，其实这段代码就来自于Chrome核心工程 **chromium**。这段代码，大家通过谷歌搜索找找看，这里我们以官方版本的代码为准：
[WebViewChromiumFactoryProviderForO](https://chromium.googlesource.com/chromium/src.git/+/28cc253ce347f9a58a0e7c6b7b249c239c4b2669/android_webview/glue/java/src/com/android/webview/chromium/WebViewChromiumFactoryProviderForO.java)

具体代码很简单，如下：
```
package com.android.webview.chromium;
class WebViewChromiumFactoryProviderForO extends WebViewChromiumFactoryProvider {
    public static WebViewChromiumFactoryProvider create(android.webkit.WebViewDelegate delegate) {
        return new WebViewChromiumFactoryProviderForO(delegate);
    }
    protected WebViewChromiumFactoryProviderForO(android.webkit.WebViewDelegate delegate) {
        super(delegate);
    }
}
```

LOL，可是，你以为真的很简单吗？其实不然，实现在父类，跟进父类。这个时候千万保持清醒，别跟丢了哦。我们想要的是Provider的创建过程，这个是Provider工厂类的真正类型，由它完成WebViewProvider的创建。

如果你已经忘了，我们再来回顾一下刚刚创建WebViewProvider的代码，别走神，看这里：
```
  private void ensureProviderCreated() {
        checkThread();
        if (mProvider == null) {
            // As this can get called during the base class constructor chain, pass the minimum
            // number of dependencies here; the rest are deferred to init().
            mProvider = getFactory().createWebView(this, new PrivateAccess());
        }
    }
```

看到了吗？这里拿到工厂类之后，调用了createWebView方法创建了Provider对象。那好办了，我们在`WebViewChromiumFactoryProviderForO`的父类`WebViewChromiumFactoryProvider`直接搜索`createWebView`方法即可。
```
@Override
    public WebViewProvider createWebView(WebView webView, WebView.PrivateAccess privateAccess) {
        return new WebViewChromium(this, webView, privateAccess, mShouldDisableThreadChecking);
    }
```
怎么样，这段代码熟悉吗？这里直接返回了一个WebViewChromium对象，也就是说，WebView的所有操作，都由WebViewChromium帮忙完成。好吧，我们继续跟进这个类。可是跟进这个类做什么呢？哈哈，忘了吧，我们的目的是寻找`addJavascriptInterface`实现。稍等，容我先擦一把汗。

```
  @Override
    public void addJavascriptInterface(final Object obj, final String interfaceName) {
        if (checkNeedsPost()) {
            mFactory.addTask(new Runnable() {
                @Override
                public void run() {
                    addJavascriptInterface(obj, interfaceName);
                }
            });
            return;
        }
        mAwContents.addJavascriptInterface(obj, interfaceName);
    }
```

稍微瞅一眼这个方法`checkNeedsPost`
```
 protected boolean checkNeedsPost() {
        boolean needsPost = !mFactory.hasStarted() || !ThreadUtils.runningOnUiThread();
        if (!needsPost && mAwContents == null) {
            throw new IllegalStateException("AwContents must be created if we are not posting!");
        }
        return needsPost;
    }
```

简单理解一下，如果已经启动或者调用该方法的线程不在UI线程，则需要post到UI线程中去，这里很明显，我们的调用是在UI线程中。因此，我们之间走下面的分支: `mAwContents.addJavascriptInterface(obj, interfaceName);`。那么，问题来了，AwContent又是什么鬼？在哪里创建的呢？

仔细查找这个类，我们发现AwContent是在initForReal方法中被创建的。而initForReal调用来自init方法。可是，init方法是在哪里调用的呢？答案是：WebView。看下面的截图：
![](http://upload-images.jianshu.io/upload_images/703764-d2adacd680391f4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

OK，继续往下，看AwContent是怎么创建的。
```
 private void initForReal() {
        AwContentsStatics.setRecordFullDocument(sRecordWholeDocumentEnabledByApi
                || mAppTargetSdkVersion < Build.VERSION_CODES.LOLLIPOP);
        mAwContents = new AwContents(mFactory.getBrowserContextOnUiThread(), mWebView, mContext,
                new InternalAccessAdapter(), new WebViewNativeDrawGLFunctorFactory(),
                mContentsClientAdapter, mWebSettings.getAwSettings(),
                new AwContents.DependencyFactory() {
                    @Override
                    public AutofillProvider createAutofillProvider(
                            Context context, ViewGroup containerView) {
                        return mFactory.createAutofillProvider(context, mWebView);
                    }
                });
        if (mAppTargetSdkVersion >= Build.VERSION_CODES.KITKAT) {
            // On KK and above, favicons are automatically downloaded as the method
            // old apps use to enable that behavior is deprecated.
            AwContents.setShouldDownloadFavicons();
        }
        if (mAppTargetSdkVersion < Build.VERSION_CODES.LOLLIPOP) {
            // Prior to Lollipop, JavaScript objects injected via addJavascriptInterface
            // were not inspectable.
            mAwContents.disableJavascriptInterfacesInspection();
        }
        // TODO: This assumes AwContents ignores second Paint param.
        mAwContents.setLayerType(mWebView.getLayerType(), null);
    }
```
下面是一些版本兼容判断，与本文探讨主题无关，先忽略。好了，看到这里，大家是不是感觉被安卓源码忽悠的团团转，最开始我们天真地以为真正的调用来自WebView，安卓系统告诉我们来自WebViewProvider，我们以为这应该就是头了。可是现在又出现了一个AwContent。那么，它是不是真正的最终调用者呢？继续往下看：
```
/**
     * @see ContentViewCore#addPossiblyUnsafeJavascriptInterface(Object, String, Class)
     */
    @SuppressLint("NewApi")  // JavascriptInterface requires API level 17.
    public void addJavascriptInterface(Object object, String name) {
        if (TRACE) Log.i(TAG, "%s addJavascriptInterface=%s", this, name);
        if (isDestroyedOrNoOperation(WARN)) return;
        Class<? extends Annotation> requiredAnnotation = null;
        if (mAppTargetSdkVersion >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            requiredAnnotation = JavascriptInterface.class;
        }
        mContentViewCore.addPossiblyUnsafeJavascriptInterface(object, name, requiredAnnotation);
    }
```

我擦，又来了一个调用对象`mContentViewCore`。Relax，继续往下看，看它的实现：
```
    public void addPossiblyUnsafeJavascriptInterface(Object object, String name,
            Class<? extends Annotation> requiredAnnotation) {
        if (mNativeContentViewCore != 0 && object != null) {
            mJavaScriptInterfaces.put(name, object);
            nativeAddJavascriptInterface(mNativeContentViewCore, object, name, requiredAnnotation,
                    mRetainedJavaScriptObjects);
        }
    }
```

看方法名，nativeAddJavascriptInterface看起来最终调用来自于Native，继续往下看：
```
   private native void nativeAddJavascriptInterface(int nativeContentViewCoreImpl, Object object,
            String name, Class requiredAnnotation, HashSet<Object> retainedObjectSet);
```

接下来看C++代码，这里的中间调用过程没有深究，但最终应该是来到了这里：
```
static void AddJavascriptInterface(JNIEnv *env, jobject obj, jint nativeFramePointer,
        jobject javascriptObj, jstring interfaceName)
{
#ifdef ANDROID_INSTRUMENT
    TimeCounterAuto counter(TimeCounter::NativeCallbackTimeCounter);
#endif
    WebCore::Frame* pFrame = 0;
    if (nativeFramePointer == 0)
        pFrame = GET_NATIVE_FRAME(env, obj);
    else
        pFrame = (WebCore::Frame*)nativeFramePointer;
    LOG_ASSERT(pFrame, "nativeAddJavascriptInterface must take a valid frame pointer!");
    JavaVM* vm;
    env->GetJavaVM(&vm);
    LOGV("::WebCore:: addJSInterface: %p", pFrame);
#if USE(JSC)
    // Copied from qwebframe.cpp
    JSC::JSLock lock(false);
    WebCore::JSDOMWindow *window = WebCore::toJSDOMWindow(pFrame);
    if (window) {
        JSC::Bindings::RootObject *root = pFrame->script()->bindingRootObject();
        JSC::Bindings::setJavaVM(vm);
        // Add the binding to JS environment
        JSC::ExecState* exec = window->globalExec();
        JSC::JSObject *addedObject = WeakJavaInstance::create(javascriptObj,
                root)->createRuntimeObject(exec);
        const jchar* s = env->GetStringChars(interfaceName, NULL);
        if (s) {
            // Add the binding name to the window's table of child objects.
            JSC::PutPropertySlot slot;
            window->put(exec, JSC::Identifier(exec, (const UChar *)s, 
                    env->GetStringLength(interfaceName)), addedObject, slot);
            env->ReleaseStringChars(interfaceName, s);
            checkException(env);
        }
    }
#endif  // USE(JSC)
#if USE(V8)
    if (pFrame) {
        const char* name = JSC::Bindings::getCharactersFromJStringInEnv(env, interfaceName);
        NPObject* obj = JSC::Bindings::JavaInstanceToNPObject(new JSC::Bindings::JavaInstance(javascriptObj));
        pFrame->script()->bindToWindowObject(pFrame, name, obj);
        // JavaInstanceToNPObject calls NPN_RetainObject on the
        // returned one (see CreateV8ObjectForNPObject in V8NPObject.cpp).
        // BindToWindowObject also increases obj's ref count and decrease
        // the ref count when the object is not reachable from JavaScript
        // side. Code here must release the reference count increased by
        // JavaInstanceToNPObject.
        _NPN_ReleaseObject(obj);
        JSC::Bindings::releaseCharactersForJString(interfaceName, name);
    }
#endif
}
```
这里的代码量较大，我们主要关注下面这一行代码：
```
 window->put(exec, JSC::Identifier(exec, (const UChar *)s, 
                    env->GetStringLength(interfaceName)), addedObject, slot);
```
最终数据的处理原来来自于C++端的window对象，这又是什么呢？继续看：
```
    WebCore::JSDOMWindow *window = WebCore::toJSDOMWindow(pFrame);
```
这是在WebCore命名空间下面的`JSDOMWindow`对象，看到这里，其实大多数同学应该已经都没有兴趣看下去了。这实在是一个冗长的调用过程，而且在阅读源码过程中，我们还忽略多进程调用，忽略各种细节。对此，关于这段源码的阅读，我们暂且告一段落，等时间充裕，我再来补充。

# 总结
这次的问题牵扯了移动端、Web前端和后台，这种跨平台的问题解决起来的确存在很大的困难。其实，我已经很长时间没有写JS了，仅仅在几个月前使用RN的时候有了解一些ES6的语法。凭借刚刚工作时仅有的2个月JS经验，加上在多方面知识的累积，总算顺利解决了问题。其实，根据我的经验来看，越是看起来无头绪的问题，往往越是一个极其简单的问题。为了避免出现这种问题，在编码过程中，必须小心翼翼。尽量多检查几次，避免出现类似这样的错误。另外，要尝试接受不一样的观点，如果你一开始就接受了其他人的观点，在解决问题上就会有很强的目的性，解决问题的速度也就更快。

>最后，新的一年里，祝大家万事如意，阖家欢乐，工作顺顺利利，身体健健康康。
