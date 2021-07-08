<section class="ouvJEz">

# Spring生命周期以及Aop的执行时机总结

<div class="rEsl9f">

<div class="_2mYfmT">[![](https://upload.jianshu.io/users/upload_avatars/15854876/2f4a25af-9408-4bc5-a70d-33c9a6a4f5c3.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](/u/c4ce397f770a)

<div style="margin-left: 8px;">

<div class="_3U4Smb"><span class="FxYr8x">[知止9528](/u/c4ce397f770a)</span><button data-locale="zh-CN" type="button" class="_3kba3h _1OyPqC _3Mi9q9 _34692-"><span>关注</span></button></div>

<div class="s-dsoj"><time datetime="2019-01-13T23:37:40.000Z">2019.01.14 07:37:40</time><span>字数 45</span><span>阅读 2,046</span></div>

</div>

</div>

</div>

<article class="_2rhmJa">

Spring生命周期总结

<div class="_2Uzcx_">

    Spring容器的refresh()【创建刷新】
    1.prepareRefresh()刷新前的预处理
        1)initPropertySources()初始化一些属性设置;子类自定义个性的属性设置方法;
        2)getEnvironment().validateRequiredProperties();检验属性的合法等
        3)this.earlyApplicationEvents = new LinkedHashSet<>();保存容器中的一些早期事件;
    2.obtainFreshBeanFactory();获取beanFactory;
        1)refreshBeanFactory();刷新beanFactory
          loadBeanDefinitions(beanFactory)解析xml,加载到beanFactory 包含了自定义标签的解析
          创建了一个this.beanFactory = new DefaultListableBeanFactory();
          设置id;
        2)getBeanFactory();返回通过GenericApplicationContext创建的BeanFactory对象;
        3)将创建的BeanFactory【DefaultListableBeanFactory】返回;
    3.prepareBeanFactory(beanFactory);BeanFactory的预准备工作(BeanFactory进行一些设置);
        1)设置BeanFactory的类加载器,支持表达式解析器...
        2)添加部分的BeanPostProcessor【ApplicationContextAwareProcessor】
        3)设置忽略的自动装配的接口EnvironmentAware,EmbeddedValueResolverAware...;
        4)注册可以解析的自动装配;我们直接在任何组件中自动注入;
          BeanFactory,ResourceLoader,ApplicationEventPublisher,ApplicationContext
        5)添加BeanPostProcessor【ApplicationListenerDetector】
        6)添加编译的AspectJ;
        7)给BeanFactory中注册一些能用的组件
        environment【ConfigurableEnvironment】,
        systemProperties【Map<String,Object>】
        systemEnvironment【Map<String,Object>】
    4.postProcessBeanFactory(beanFactory);BeanFactory准备工作完成后进行的后置处理工作;
        1)子类通过重写这个方法在BeanFactory创建并预备完成以后做进一步的设置
    ==============================以上是BeanFactory的创建以及预备准备工作======================
    5.invokeBeanFactoryPostProcessors(beanFactory);执行BeanFactoryPostProcessors;
        BeanFactoryPostProcessor;BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的;
        两个接口:BeanFactoryPostProcessor,BeanDefinitionRegistryPostProcessor
        1)执行BeanFactoryPostProcessor的方法
          1)获取所有的BeanDefinitionRegistryPostProcessor;
          2)看优先级排序PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor,
             postProcessor.postProcessBeanDefinitionRegistry(registry);
          3)在执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor；
             postProcessor.postProcessBeanDefinitionRegistry(registry);
          4)最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessor;
             postProcessor.postProcessBeanDefinitionRegistry(registry);
    6.registerBeanPostProcessors(beanFactory);注册BeanPostProcessor(Bean的后置处理器)  intercept bean creation.
        不同接口类型的BeanPostProcessor;在Bean创建前后的执行时机是不一样的
        BeanPostProcessor,
        DestructionAwareBeanPostProcessor,
        InstantiationAwareBeanPostProcessor,
        SmartInstantiationAwareBeanPostProcessor,
        MergedBeanDefinitionPostProcessor,

        1)获取所有的BeanPostProcessor;后置处理器都默认可以通过priorityOrdered,Ordered接口来执行优先级
        2)先注册PriorityOrdered优先级接口的BeanPostProcessor;
           把每一个BeanPostProcessor添加到BeanFactory中
           beanFactory.addBeanPostProcessor(postProcessor);
        3)在注册Ordered接口的
        4)最后注册没有实现任何优先级接口的
        5)最终注册MergedBeanDefinitionPostProcessor;
        6)注册一个ApplicationListenerDetector;来在Bean创建完成后检查是否是ApplicationListener;
    7.initMessageSource();初始化MessageSource组件(做国际化功能;消息绑定;消息解析);
        1)获取BeanFactory
        2)看容器是否有Id为messageSource的,类型是MessageSource的组件
          如果有赋值给messageSource,如果没有自己创建一个DelegatingMessageSource;
               MessageSource:取出国际化配置文件中的每个Key的值,能按照区域信息获取
        3)把创建好的MessageSouce注册在容器中,以后获取国际化配置文件的值的时候,可以自动化注册MessageSource
          beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
    8.initApplicationEventMulticaster();初始化事件派发器;
        1)获取BeanFactory
        2)从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster;
        3)如果上一步没有配置;创建一个SimpleApplicationEventMulticaster
        4)将创建的ApplicationEventMulticaster添加到BeanFactory中,以后其它组件直接自动注入
    9.onRefresh();留给子类
        1).子类重写这个方法,在容器刷新的时候可以自定义逻辑;
    10.registerListeners();给容器中将所有项目里面的ApplicationListener注册进来;
        1)从容器中拿到所有的ApplicationListener
        2)将每个监听器添加到事件派发器中;
          getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
        3)派发之前步骤产生的事件;
        getApplicationEventMulticaster().multicastEvent(earlyEvent);
    11.finishBeanFactoryInitialization(beanFactory);初始化所有身下的单实例bean;
        1.beanFactory.preInstantiateSingletons();
          1)获取容器中所有的Bean,依次进行初始化和创建对象
          2)获取Bean的定义信息;RootBeanDefinition
          3)Bean不是抽象的,是单实例的,是懒加载;
            1)判断是否是FactoryBean;是否是实现了FactoryBean接口的bean
            2)不是工厂Bean.利用getBean(beanName);创建对象
              0)getBean(beanName); ioc.getBean();
              1)doGetBean(name, null, null, false);
              2)先获取缓存中保存的单实例Bean.如果能获取到说明这个Bean之前被创建过(所有创建过的单实例Bean都会被缓存起来)
                private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
              3)缓存获取不到,开始Bean的创建对象流程;
              4)标记当前bean已经被创建
              5)获取bean的定义信息;
              6)获取当前bean依赖的其它bean;如果按照getBean()把依赖的Bean先创建出来
              7)启动单实例Bean的创建流程; 
                1)createBean(beanName, mbd, args);
                2)Obeject bean = resolveBeforeInstantiation(beanName, mbd);可以提前获取代理对象
                  InstantiationAwareBeanPostProcessor;提前执行;
                  先触发applyBeanPostProcessorsBeforeInstantiation();
                  如果有返回值;触发applyBeanPostProcessorsAfterInitialization();
                3)如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象;则调用4)
                4)Object beanInstance = doCreateBean(final String beanName, final RootBeanDefinition mbd, Object[] args);创建Bean
                  1)创建Bean实例;createBeanInstance(beanName, mbd, args);
                    利用工厂方法或者对象的构造器创建出Bean的实例
                  2)applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                     调用MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition方法
                  3)【Bean属性赋值】populateBean(beanName, mbd, instanceWrapper);
                    赋值之前:
                     1)拿到InstantiationAwareBeanPostProcessor后置处理器;
                       postProcessAfterInstantiation();
                     2)拿到InstantiationAwareBeanPostProcessor后置处理器;
                       postProcessPropertyValues();
                    ====赋值之前===
                     3)应用Bean属性的值;为属性利用setter方法等进行赋值;
                       applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
                  4)【Bean初始化】initializeBean(beanName, exposedObject, mbd);
                     1)【执行Aware接口方法】invokeAwareMethods(beanName,bean);执行xxxAware接口的方法;
                        BeanNameAware,BeanClassLoaderAware,BeanFactoryAware
                     2)【执行后置处理器初始化之前】applyBeanPostProcessorsBeforeInitialization(bean, beanName);
                        BeanPostProcessor.postProcessBeforeInitialization();
                     3)【执行初始化方法】invokeInitMethods(beanName, wrappedBean, mbd);
                       1)是否是InitializingBean接口的实现;执行接口规定的初始化;
                       2)是否自定义初始化方法;
                     4)【执行后置处理器初始化之后】applyBeanPostProcessorsAfterInitialization
                        BeanPostProcessor.postProcessAfterInitialization()
                  5)注册Bean的销毁方法;
                5)将创建的Bean添加到缓存中singletonObjects;
                ioc容器就是这些Map;很多的Map里面保存了单实例Bean,环境信息....

</div>

小结:

<div class="_2Uzcx_">

    1)Spring容器在启动的时候,先会保存所有注册进来的Bean的定义信息;
       1)xml注册bean;<bean>
       2)注解注册bean;@Servoce,@Component@Bean ...
    2)Spring容器会在何时的时机创建这些Bean
       1)用到这个bean的时候;利用getBean创建bean;创建好以后保存在容器中;
       2)统一创建剩下所有的bean的时候,finishBeanFactoryInitialization();
    3)后置处理器;BeanPostProcessor
       1)每一个bean创建完成,都会使用各种后置处理器进行处理,来增强bean的功能
          AutowiredAnnotationBeanPostProcess:处理自动注入
          AnnotationAwareAspectJAutoProxyCreator:来做Aop功能;
          xxx....
    4)事件驱动模型:
      ApplicationListener;时间监听
      ApplicationEventMulticaster:事件派发:

</div>

Spring的Bean对象创建完整流程图如下(从getBean开始)

<div class="image-package">

<div class="image-container" style="max-width: 700px; max-height: 2805px; background-color: transparent;">

<div class="image-view" data-width="873" data-height="3499">![](//upload-images.jianshu.io/upload_images/15854876-4d3e161994d39557.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/873/format/webp)</div>

</div>

<div class="image-caption">spring的完整流程图.jpg</div>

</div>

* * *

obtainFreshBeanFactory();获取beanFactory;  
1)refreshBeanFactory();刷新beanFactory  
loadBeanDefinitions(beanFactory)解析xml标签 如下

<div class="image-package">

<div class="image-container" style="max-width: 700px; max-height: 541px; background-color: transparent;">

<div class="image-view" data-width="1042" data-height="806">![](//upload-images.jianshu.io/upload_images/15854876-9c4e56dc13c064f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1042/format/webp)</div>

</div>

<div class="image-caption">image.png</div>

</div>

</article>

<div class="_1kCBjS">

<div class="_18vaTa">

<div class="_3BUZPB"><span class="_1LOh_5" role="button" tabindex="-1" aria-label="查看点赞列表">5人点赞</span></div>

</div>

<div class="_18vaTa">[<span>spring</span>](/nb/33256309)</div>

</div>

<div class="_13lIbp">

<div class="_16AzcO">更多精彩内容，就在简书APP</div>

<div class="_6S_NkV">

<canvas height="110" width="110" style="height: 110px; width: 110px;"></canvas>

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABacAAAWnCAMAAABgpz87AAAAh1BMVEX////aZWf45eL99/bcb2788O3ijojeeXf0083vwLnprKbghIDstq/no5zxysT33NjkmJL++vraZ2jklI7ffnv88/HbbGzbaWnoqKH+/f366ebmnJX119LddnPccnH44NzrsaruvLbii4b77evzzsjwx8D22tXhh4PnoJntubLww7z449/deHYjPTZEAAAwHklEQVR42uzBgQAAAACAoP2pF6kCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGD24EAAAAAAAMj/tRFUVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVYU9OBAAAAAAAPJ/bQRVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVWEPDgQAAAAAgPxfG0FVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVdiDAwEAAAAAIP/XRlBVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVFfbgQAAAAAAAyP+1EVRVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVhT04EAAAAAAA8n9tBFVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVYQ8OBAAAAACA/F8bQVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVV2IMDAQAAAAAg/9dGUFVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVUV9uCQAAAAAEDQ/9dusAMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA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)</div>

<div class="_191KSt">"小礼物走一走，来简书关注我"</div>

<button type="button" class="_1OyPqC _3Mi9q9 _2WY0RL _1YbC5u"><span>赞赏支持</span></button><span class="_3zdmIj">还没有人赞赏，支持一下</span></div>

<div class="d0hShY">[![  ](https://upload.jianshu.io/users/upload_avatars/15854876/2f4a25af-9408-4bc5-a70d-33c9a6a4f5c3.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](/u/c4ce397f770a)

<div class="Uz-vZq">

<div class="Cqpr1X">[知止9528](/u/c4ce397f770a "知止9528")<span class="_2WEj6j" title=""></span></div>

<div class="lJvI3S"><span>总资产2 (约0.14元)</span><span>共写了28.2W字</span><span>获得377个赞</span><span>共104个粉丝</span></div>

</div>

<button data-locale="zh-CN" type="button" class="_1OyPqC _3Mi9q9"><span>关注</span></button></div>

</section>
