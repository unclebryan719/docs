# 分包说明
![avator](http://pic.pig4cloud.com/20190220232605_DAHeuX_Screenshot.jpeg)
 具体参考上一章节[EnablePigFeignClients原理解析]

main类 使用了@EnablePigFeignClients 默认扫描
出现FeignClient 调用报错等，请检查你的分包是否符合pig 的标准
包名 com.pig4cloud.pig.模块.api
# 新增feignClient
![avator](http://pic.pig4cloud.com/20190221105453_xwRUY1_Screenshot.jpeg)

RemoteXXXService FeignClient 客户端,声明具体的接口和降级工程
@FeignClient(value = ServiceNameConstants.UMPS_SERVICE, fallbackFactory = RemoteLogServiceFallbackFactory.class)

降级工厂类，注意这里的@Component 注入到Spring
@Component
public class RemoteLogServiceFallbackFactory implements FallbackFactory<RemoteLogService> {

	@Override
	public RemoteLogService create(Throwable throwable) {
		RemoteLogServiceFallbackImpl remoteLogServiceFallback = new RemoteLogServiceFallbackImpl();
		remoteLogServiceFallback.setCause(throwable);
		return remoteLogServiceFallback;
	}
}
具体的降级业务
@Component
public class RemoteLogServiceFallbackImpl implements RemoteLogService {
	@Setter
	private Throwable cause;
	public R<Boolean> saveLog(SysLog sysLog, String from) {
		log.error("feign 插入日志失败", cause);
		return null;
	}
}

# 为了保证引入api模块可以直接使用，在spring.factories 配置bean
![avator](http://pic.pig4cloud.com/20190221105841_hJ0rah_Screenshot.jpeg)

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.pig4cloud.pig.admin.api.feign.fallback.RemoteXXXServiceFallbackImpl,\
  com.pig4cloud.pig.admin.api.feign.factory.RemoteXXXServiceFallb