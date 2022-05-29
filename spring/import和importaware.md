```
@Component
@ImportTest(importTest="helloImportAware")
public class Car implements BeanDefinitionRegistryPostProcessor, PriorityOrdered {
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import (ImportAwareTest.class)
public @interface ImportTest {
    String importTest() default "";
}

@Data
@Component
public class ImportAwareTest  implements ImportAware {
private String importTest;
    @Override
    public void setImportMetadata(AnnotationMetadata importMetadata) {
        //这里importMetadata不是ImportTest而是@ImportTest注解标注的类,这里就是Car
        //经常通过这种方式获取注解的信息
        Map<String, Object> map = importMetadata.getAnnotationAttributes (ImportTest.class.getName ());
        AnnotationAttributes attrs = AnnotationAttributes.fromMap(map);
         importTest = attrs.getString ("importTest");
    }
}

@RestController
public class ControllerTest {
    @Autowired
    private ImportAwareTest importAwareTest;
    @RequestMapping("importAwareTest")
    public String importAwareTest(){
        return importAwareTest.getImportTest ();
    }

}
```
