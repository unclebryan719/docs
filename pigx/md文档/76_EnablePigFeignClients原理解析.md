# 源码解析
只是给默认的EnableFeignClients 增加了一个默认值。

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EnableFeignClients
public @interface EnablePigxFeignClients {

	String[] value() default {};
    
    // 指定默认的扫描范围
	String[] basePackages() default {"com.pig4cloud.pigx"};

	Class<?>[] basePackageClasses() default {};

	Class<?>[] defaultConfiguration() default {};

	Class<?>[] clients() default {};
}

# 以UPMS为例分析封装的好处
![avator](http://pic.pig4cloud.com/20190220232605_DAHeuX_Screenshot.jpeg)

如果使用原生的EnableFeignClients 默认的扫描范围是 com.pig4cloud.pig.admin 包的所有FeignClient。
而由于微服务拆分所有的feignClient 都在 com.pig4cloud.pig.模块.api包里面，这样默认情况会扫描不到
除非明确指定扫描范围 @EnableFeignClients("com.pig4cloud.pig.模块.api")
使用了@EnablePigFeignClients 默认扫描 com.pig4cloud.pigx下边的feignClient 更为简洁
# @EnableFeignClients
@EnableFeignClients
@SpringCloudApplication
public class PigAdminApplication {

}

# @EnablePigFeignClients
@EnablePigFeignClients
@SpringCloudApplication
public class PigAdminApplication {
	public static void main(String[] args) {
		SpringApplication.run(PigAdminApplication.class, args);
	}

}