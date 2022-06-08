## 前言
说到Spring Security的实现原理，大部分都知道是通过过滤器链。因为Spring Security中的所有功能都是通过过滤器链来实现的，这些过滤器组成一个完整的过滤器链。那么这些过滤器链是如何初始化的？我们之前说的AuthenticationManager又是如何初始化的？前面我们对Spring Security的一些核心过滤器以及组件有了一定的了解。由于初始化流程相对复杂，因此我们并没有一开始从这一块分析。

### 初始化流程分析
Spring Security初始化流程整体上来说还是很好理解的，但是这里涉及的零碎知识比较多，因此这里先介绍一下常见的关键组件，在理解这些组件的基础上，在来分析初始化流程。

### ObjectPostProcessor
ObjectPostProcessor是Spring Security中使用频率最高的组件之一，它是一个对象后置处理器，也就是当一个对象创建成功后，如果还有一些额外的事情需要补充，那么可以通过ObjectPostProcessor来进行处理。这个接口中只有一个默认的postProcess

ObjectPostProcessor默认有两个继承类：
![image](https://user-images.githubusercontent.com/15857347/172551478-efb7eb70-ebeb-417b-9ea9-f4b8cf1952d8.png)


- AutowireBeanFactoryObjectPostProcessor：由于SpringSecurity中大量采用了JAVA配置，许多过滤器都是直接new出来的，这些直接new出来的对象并不会自动注入到Spring容器中。Spring Security这样做的本意是简化配置，但是却带来了另一个问题，大量new出来的对象需要我们手动注册到Spring容器中。AutowireBeanFactoryObjectPostProcessor对象所承担的就是这件事情，一个对象new出来之后，只要调用AutowireBeanFactoryObjectPostProcessor#postProcess方法就可以成功注入到Spring容器中，它的实现原理就是通过调用Spring容器中的AutowireCapableBeanFactory对象将一个new出来的对象注入到Spring容器中。

- CompositeObjectPostProcessor：这是另一个实现，一个对象可以有一个后置处理器，我们也可以自定义多个对象后置处理器。CompositeObjectPostProcessor是一个组合的对象后置处理器。他里面维护了一个List集合，集合中存放了某一个对象的所有后置处理器，当需要执行对的后置 处理器时，会遍历集合中的所有ObjectPostProcess实例，分别调用实例的postProcess方法进行对象后置处理。在Spring Security框架中，最终使用的对象后置处理器其实就是CompositeObjectPostProcessor，他里面的集合默认只有一个对象，就是AutowireBeanFactoryObjectPostProcessor。

在Spring Security中，我们可以灵活的配置 项目中需要哪些Spring Security过滤器，一旦选的过滤器之后，每一个过滤器都会有一个对应的配置器，叫做xxxConfigurer（例如：CorsConfigurer、CsrfConfigurer），过滤器都是在xxxConfigurer中new出来的，然后在postProcess方法中处理一遍，就将这些过滤器注入到Spring容器中了。

这就是对象后置处理器ObjectPostProcessor的主要作用。

### SecurityFilterChain
从名字上看，我们可以知道SecurityFilterChain是Spring Security中的过滤器链对象：
```
public interface SecurityFilterChain {

	boolean matches(HttpServletRequest request);

	List<Filter> getFilters();

}
```
- matches：该方法用来判断request请求是否应该被当前过滤器链所处理。
- getFilters：该方法返回一个List集合，集合中存放的就是Spring Security中的过滤器。如果matches方法返回true，那么request请求就会在该方法所返回的Filter集合中被处理。
SecurityFilterChain只有一个默认的实现类DefaultSecurityFilterChain，器中定义了两个属性，并具体实现了SecurityFilterChain中的两个方法：
```
public final class DefaultSecurityFilterChain implements SecurityFilterChain {

	private static final Log logger = LogFactory.getLog(DefaultSecurityFilterChain.class);

	private final RequestMatcher requestMatcher;

	private final List<Filter> filters;

	public DefaultSecurityFilterChain(RequestMatcher requestMatcher, Filter... filters) {
		this(requestMatcher, Arrays.asList(filters));
	}

	public DefaultSecurityFilterChain(RequestMatcher requestMatcher, List<Filter> filters) {
		logger.info(LogMessage.format("Will secure %s with %s", requestMatcher, filters));
		this.requestMatcher = requestMatcher;
		this.filters = new ArrayList<>(filters);
	}

	public RequestMatcher getRequestMatcher() {
		return this.requestMatcher;
	}

	@Override
	public List<Filter> getFilters() {
		return this.filters;
	}

	@Override
	public boolean matches(HttpServletRequest request) {
		return this.requestMatcher.matches(request);
	}

	@Override
	public String toString() {
		return this.getClass().getSimpleName() + " [RequestMatcher=" + this.requestMatcher + ", Filters=" + this.filters
				+ "]";
	}

}
```
可以看到，在DefaultSecurityFilterChain的构造方法中，需要传入两个参数，一个是请求匹配器requestMatcher，另一个则是过滤器集合或者过滤器数组Filter。这个类的实现很简单。

*需要注意的是，在一个Spring Security项目中，SecurityFilterChain的实例可能会有多个。

### SecurityBuilder
Spring Secuirty中所有需要构建的对象都可以通过SecurityBuilder来实现，默认的过滤器链、代理过滤器、AuthenticationManager等，都可以通过SecurityBuilder来构建。

![image](https://user-images.githubusercontent.com/15857347/172551527-f12cbfaa-342d-4c65-ae70-54b28725bcfb.png)


SecurityBuilder，我们先开看看它的源码：
```
public interface SecurityBuilder<O> {

	
	O build() throws Exception;

}
```
可以看到接口中只有一个build方法，就是对象构建方法。build方法的返回值，就是具体构建的对象泛型O，也就是说不通的SecurityBuild将会构建出不同的对象。

### HttpSecurityBuilder
HttpSecurityBuilder是用来构建HttpSecurity对象的，HttpSecurityBuilder的定义如下
```
public interface HttpSecurityBuilder<H extends HttpSecurityBuilder<H>>
		extends SecurityBuilder<DefaultSecurityFilterChain> {

	
	<C extends SecurityConfigurer<DefaultSecurityFilterChain, H>> C getConfigurer(Class<C> clazz);

	
	<C extends SecurityConfigurer<DefaultSecurityFilterChain, H>> C removeConfigurer(Class<C> clazz);

	
	<C> void setSharedObject(Class<C> sharedType, C object);

	
	<C> C getSharedObject(Class<C> sharedType);

	
	H authenticationProvider(AuthenticationProvider authenticationProvider);

	
	H userDetailsService(UserDetailsService userDetailsService) throws Exception;

	H addFilterAfter(Filter filter, Class<? extends Filter> afterFilter);

	
	H addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter);

	
	H addFilter(Filter filter);

}

```
我们简单分析一下：

- （1）、HttpSecurityBuilder对象本身在定义时就有一个泛型，这个泛型是HttpSecurityBuilder的子类，由于默认情况下HttpSecurityBuilder的实现类只有一个HTTPSecurity，所以可暂且把接口中的H 都当成HttpSecurity

- （2）、HttpSecurityBuilder继承自SecurityBuild接口，同时也指定了SecurityBuilder中的泛型为DefaultSecurityFilterChain，也就是说，HttpSecurityBuilder最终想要构建的对象就是DefaultSecurityFilterChain

- （3）、getConfigurer方法用来获取一个配置器，也就是xxxConfigurer，我们在后续会说到

- （4）、removeConfigurer方法用来移除一个配置器（相当于从Spring Security过滤器链中移除一个过滤器）。

- （5）、setSharedObject/getSharedObject这两个方法用来设置或获取一个可以在多个配置器之间共享的对象。

- （6）、authenticationProvider方法可以用来配置一个认证器AuthenticationProvider

- （7）、userDetailsService方法可以用来配置一个数据源UserDetailsService

- （8）、addFilterAfter/addFilterBefore方法表示在某一个过滤器之后或者之前添加一个自定义的过滤器。

- （9）、addFilter方法可以添加一个过滤器，这个过滤器必须是SpringSecurity提供的过滤器的一个实例或者其扩展，会自动进行过滤器的排序。

### AbstractSecurityBuilder
AbstractConfiguredSecurityBuilder实现了ScurityBuilder接口，并对build做了完善，确保只build一次，我们来看一下：
```
public abstract class AbstractSecurityBuilder<O> implements SecurityBuilder<O> {

	private AtomicBoolean building = new AtomicBoolean();

	private O object;

	@Override
	public final O build() throws Exception {
		if (this.building.compareAndSet(false, true)) {
			this.object = doBuild();
			return this.object;
		}
		throw new AlreadyBuiltException("This object has already been built");
	}

	public final O getObject() {
		if (!this.building.get()) {
			throw new IllegalStateException("This object has not been built");
		}
		return this.object;
	}

	
	protected abstract O doBuild() throws Exception;

}
```
由上述可知，在AbstractSecurityBuilder类中：

- （1）、首先声明了building变量，可以确保即使在多线程环境下，配置类也只构建一次。

- （2）、对build方法进行重写，并且设置final，这样在AbstractSecurityBuilder的子类中将不能再次重写build方法。在build方法内部，通过building变量来控制配置类只构建一次，具体的构建工作则交给doBuild方法去完成。

- （3）、getObject方法用来返回构建的对象。

- （4）、doBuild方法则是具体的构建方法，该方法在AbstractSecurityBuilder中是一个抽象方法，具体的实现在其子类中。

### AbstractConfiguredSecurityBuilder
AbstractConfiguredSecurityBuilder的源码较长，我们分开来看。

首先在AbstractConfiguredSecurityBuilder中声明一个枚举，用来描述构建过程的不同状态
```
private enum BuildState {

		
		UNBUILT(0),

		
		INITIALIZING(1),

		CONFIGURING(2),

		BUILDING(3),

		
		BUILT(4);

		private final int order;

		BuildState(int order) {
			this.order = order;
		}

		public boolean isInitializing() {
			return INITIALIZING.order == this.order;
		}

		
		public boolean isConfigured() {
			return this.order >= CONFIGURING.order;
		}

	}
```
- UNBUILT：配置类构建前
- INITIALIZING：初始化中（初始化完成之前是这个状态）
- CONFIGURING：配置中（开始构建之前是这个状态）
- BUILDING：构建中
- BUILT：构建完成
这个枚举里面还提供了两个判断方法，isInitializing表示是否正在初始化中，isConfigured方法表示是否已完成配置。

AbstractConfiguredSecurityBuilder中还生命了configurers变量，用来保存所有的配置类。针对configurers变量，我们可以进行添加配置、移除配置等操作，相关方法如下：
```
public abstract class AbstractConfiguredSecurityBuilder<O, B extends SecurityBuilder<O>>
		extends AbstractSecurityBuilder<O> {
    
    		private final LinkedHashMap<Class<? extends SecurityConfigurer<O, B>>, List<SecurityConfigurer<O, B>>> configurers = new LinkedHashMap<>();

		public <C extends SecurityConfigurerAdapter<O, B>> C apply(C configurer) throws Exception {
		configurer.addObjectPostProcessor(this.objectPostProcessor);
		configurer.setBuilder((B) this);
		add(configurer);
		return configurer;
	}
    
    public <C extends SecurityConfigurer<O, B>> C apply(C configurer) throws Exception {
		add(configurer);
		return configurer;
	}
    
    private <C extends SecurityConfigurer<O, B>> void add(C configurer) {
		Assert.notNull(configurer, "configurer cannot be null");
		Class<? extends SecurityConfigurer<O, B>> clazz = (Class<? extends SecurityConfigurer<O, B>>) configurer
				.getClass();
		synchronized (this.configurers) {
			if (this.buildState.isConfigured()) {
				throw new IllegalStateException("Cannot apply " + configurer + " to already built object");
			}
			List<SecurityConfigurer<O, B>> configs = null;
			if (this.allowConfigurersOfSameType) {
				configs = this.configurers.get(clazz);
			}
			configs = (configs != null) ? configs : new ArrayList<>(1);
			configs.add(configurer);
			this.configurers.put(clazz, configs);
			if (this.buildState.isInitializing()) {
				this.configurersAddedInInitializing.add(configurer);
			}
		}
	}
    
    public <C extends SecurityConfigurer<O, B>> List<C> getConfigurers(Class<C> clazz) {
		List<C> configs = (List<C>) this.configurers.get(clazz);
		if (configs == null) {
			return new ArrayList<>();
		}
		return new ArrayList<>(configs);
	}
    
    public <C extends SecurityConfigurer<O, B>> List<C> removeConfigurers(Class<C> clazz) {
		List<C> configs = (List<C>) this.configurers.remove(clazz);
		if (configs == null) {
			return new ArrayList<>();
		}
		return new ArrayList<>(configs);
	}
    
    public <C extends SecurityConfigurer<O, B>> C getConfigurer(Class<C> clazz) {
		List<SecurityConfigurer<O, B>> configs = this.configurers.get(clazz);
		if (configs == null) {
			return null;
		}
		Assert.state(configs.size() == 1,
				() -> "Only one configurer expected for type " + clazz + ", but got " + configs);
		return (C) configs.get(0);
	}
    
    public <C extends SecurityConfigurer<O, B>> C removeConfigurer(Class<C> clazz) {
		List<SecurityConfigurer<O, B>> configs = this.configurers.remove(clazz);
		if (configs == null) {
			return null;
		}
		Assert.state(configs.size() == 1,
				() -> "Only one configurer expected for type " + clazz + ", but got " + configs);
		return (C) configs.get(0);
	}
    
    private Collection<SecurityConfigurer<O, B>> getConfigurers() {
		List<SecurityConfigurer<O, B>> result = new ArrayList<>();
		for (List<SecurityConfigurer<O, B>> configs : this.configurers.values()) {
			result.addAll(configs);
		}
		return result;
	}
}
```
我们来看一下：

- （1）、首先声明了一个configurers变量，用来保存所有的配置类，key是配置类Class对象，值是List集合中放着配置类。

- （2）、apply方法有两个，参数类型略有差异，主要功能基本一致，都是向configurer变量中添加配置类，具体的添加过程则是调用add方法。

- （3）、add方法用来将所有的配置类保存到configurers中，在添加过程中，如果allowConfigurersOfSameType变量变为true，则表示允许相同类型的配置类存在，也就是List集合中可以存在多个相同类型的配置类。默认情况下，如果是普通配置类，allowConfigurersOfSameType是false，所以List集合中的配置类始终只有一个配置类；如果在AuthenticationManagerBuilder中设置allowConfigurersOfSameType为true，此时相同类型的配置可以有多个支持。

- （4）、getConfigurers方法可以从configurers中返回某一个配置类对应的所有实例。

- （5）、removeConfigurer方法是一个私有方法，主要把所有的配置类实例放到一个集合中返回。在配置类初始化和配置的时候，会调用该方法。

这就是AbstractConfiguredSecurityBuilder中关于configurers的所有操作。

接下来就是AbstractConfiguredSecurityBuilder中的doBuild方法，这是核心的构建方法：
```
@Override
	protected final O doBuild() throws Exception {
		synchronized (this.configurers) {
			this.buildState = BuildState.INITIALIZING;
			beforeInit();
			init();
			this.buildState = BuildState.CONFIGURING;
			beforeConfigure();
			configure();
			this.buildState = BuildState.BUILDING;
			O result = performBuild();
			this.buildState = BuildState.BUILT;
			return result;
		}
	}

protected void beforeInit() throws Exception {
	}

protected void beforeConfigure() throws Exception {
	}
private void configure() throws Exception {
		Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();
		for (SecurityConfigurer<O, B> configurer : configurers) {
			configurer.configure((B) this);
		}
	}
protected abstract O performBuild() throws Excep
    
    private void init() throws Exception {
		Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();
		for (SecurityConfigurer<O, B> configurer : configurers) {
			configurer.init((B) this);
		}
		for (SecurityConfigurer<O, B> configurer : this.configurersAddedInInitializing) {
			configurer.init((B) this);
		}
	}
```
- （1）、在doBuild方法中，一遍更新构建状态，一遍执行构建方法。在构建方法中，beforeInit是一个空的初始化方，如果需要在初始化之前做一些准备工作，可以通过重写该方法实现。

- （2）、init方法是所有配置类的初始化方法，在该方法中，遍历所有的配置类，并调用其init方法完成初始化操作。

- （3）、beforeConfigure方法可以在configure方法执行之前做一些准备操作。该方法默认也是一个空方法。

- （4）、configure方法用来完成所有配置类的配置，在configure方法中，遍历所有的配置类，分别调用configure方法完成配置。

- （5）、performBuild方法用来做最终的构建操作，前面的准备工作完成后，最后在performBuild方法中完成构建，这是一个抽象方法，具体的实现则在不同的配置类中。

这就是AbstractConfiguredSecurityBuilder中最主要的几个方法，其他一些方法比较简单了就。

### ProviderManagerBuilder
ProviderManagerBuilder继承自SecurityBuild接口，并指定了构建的对象是AuthenticationManager，如下：
```
public interface ProviderManagerBuilder<B extends ProviderManagerBuilder<B>>
		extends SecurityBuilder<AuthenticationManager> {

	
	B authenticationProvider(AuthenticationProvider authenticationProvider);

}
```
可以看到，ProviderManagerBuilder中增加了一个authenticationProvider方法，同时通过泛型指定了构建的对象为AuthenticationManager。

### AuthenticationManagerBuilder
AuthenticationManagerBuilder用来构建AuthenticationManager对象，它继承自AbstractConfiguredSecurityBuilder，并且实现了ProviderManagerBuilder接口，我们截取部分常用代码来看：
```
public class AuthenticationManagerBuilder
		extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>
		implements ProviderManagerBuilder<AuthenticationManagerBuilder> {

	
    	public AuthenticationManagerBuilder(ObjectPostProcessor<Object> objectPostProcessor) {
		super(objectPostProcessor, true);
	}
    
	
    	public AuthenticationManagerBuilder parentAuthenticationManager(AuthenticationManager authenticationManager) {
		if (authenticationManager instanceof ProviderManager) {
			eraseCredentials(((ProviderManager) authenticationManager).isEraseCredentialsAfterAuthentication());
		}
		this.parentAuthenticationManager = authenticationManager;
		return this;
	}
	
    
    	public InMemoryUserDetailsManagerConfigurer<AuthenticationManagerBuilder> inMemoryAuthentication()
			throws Exception {
		return apply(new InMemoryUserDetailsManagerConfigurer<>());
	}
    
    	public JdbcUserDetailsManagerConfigurer<AuthenticationManagerBuilder> jdbcAuthentication() throws Exception {
		return apply(new JdbcUserDetailsManagerConfigurer<>());
	}
    
    	public <T extends UserDetailsService> DaoAuthenticationConfigurer<AuthenticationManagerBuilder, T> userDetailsService(
			T userDetailsService) throws Exception {
		this.defaultUserDetailsService = userDetailsService;
		return apply(new DaoAuthenticationConfigurer<>(userDetailsService));
	}
	
    
    	@Override
	public AuthenticationManagerBuilder authenticationProvider(AuthenticationProvider authenticationProvider) {
		this.authenticationProviders.add(authenticationProvider);
		return this;
	}
    
    
	@Override
	protected ProviderManager performBuild() throws Exception {
		if (!isConfigured()) {
			this.logger.debug("No authenticationProviders and no parentAuthenticationManager defined. Returning null.");
			return null;
		}
		ProviderManager providerManager = new ProviderManager(this.authenticationProviders,
				this.parentAuthenticationManager);
		if (this.eraseCredentials != null) {
			providerManager.setEraseCredentialsAfterAuthentication(this.eraseCredentials);
		}
		if (this.eventPublisher != null) {
			providerManager.setAuthenticationEventPublisher(this.eventPublisher);
		}
		providerManager = postProcess(providerManager);
		return providerManager;
	}
    
}
```
- （1）、首先在AuthenticationManagerBuilder构造方法中，调用了父类的构造方法，注意第二参数传递了true，表示允许相同类型的配置类同时存在。

- （2）、parentAuthenticationManager 方法用来给一个AuthenticationManager设置parent。

- （3）、inMemoryAuthentication方法用来配置基于内存的数据源，该方法会自动创建InMemoryUserDetailsManagerConfigurer配置类，并最终将该配置类添加到父类的configurers变量中，由于设置了允许相同类型的配置类同时存在，因此该方法可以反复调用多次。

- （4）、jdbcAuthentication以及userDetailsService方法上（3）方法类似。

- （5）、authenticationProvider方法用来向authenticaitonProviders集合添加AuthenticationProvider对象，根据前几篇文章说的，我们已经知道一个AuthenticationManager实例中包含多个AuthenticationProvider实例，那么多个AuthenticationProvider实例可以通过authenticationProvider方法进行添加。

- （6）、performBuild 方法则只需具体的构建工作，常用的AuthenticationManager实例就是ProviderManager，所以这里创建ProviderManager对象，并且配置authenticationProviders和parentAuthenticationManager对象，ProviderManager对象创建成功之后，再去创建后置处理器中处理一遍在返回。

这就是AuthenticationManagerBuilder中的一个大致逻辑。
