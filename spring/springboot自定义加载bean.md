Configuration方式：
```
@Import({Circle.class})
@Configuration
public class MainConfig {

}

 @Autowired
    private Circle circle;
```

annotation方式:
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({TUserRole.class})
public @interface EnableTUserTRole {
}

@EnableTUserTRole
@SpringBootApplication
public class KaleldoApplication {
    public static void main(String[] args) {
        SpringApplication.run(KaleldoApplication.class, args);
    }
}

  @Autowired
    private TUserRole tUserRole;
```
ImportSelector:
```
public class ApplicationSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{
                TRole.class.getName(),
                TUser.class.getName()
        };
    }
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ApplicationSelector.class})
public @interface EnableApplicationSelector {
}

@EnableApplicationSelector
@EnableTUserTRole
@SpringBootApplication
public class KaleldoApplication {
    public static void main(String[] args) {
        SpringApplication.run(KaleldoApplication.class, args);
    }
}

    @Autowired
    private TUser tUser;

    @Autowired
    private TRole tRole;
```

ImportBeanDefinitionRegistrar:
```
public class EImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {

        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TMenu.class);
            // 注册一个名字叫做 kaleldo 的 bean
        beanDefinitionRegistry.registerBeanDefinition("kaleldo", rootBeanDefinition);
    }
}


@Import({Circle.class, EImportBeanDefinitionRegistrar.class})
@Configuration
public class MainConfig {

}

    @Resource(name = "kaleldo")
    private TMenu tMenu;
```
