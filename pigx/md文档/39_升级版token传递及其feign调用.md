如下图客户端携带token访问A服务。
A服务通过FeginClient 调用B服务获取相关依赖数据。
所以只要带token 访问A 无论后边链路有多长 ABCD 都可以获取当前用户信息
权限需要有这些整个链路接口的全部权限才能成功
有token 调用不需要加 Inner /FROM_IN 等 ！！！！！
![avator](http://pic.pig4cloud.com/20190220195248_W5WBhh_token%E4%BC%A0%E9%80%92.jpeg)

# 核心代码
fein 拦截器将本服务的token 通过copyToken的形式传递给下游服务

public class PigFeignClientInterceptor extends OAuth2FeignRequestInterceptor {

	@Override
	public void apply(RequestTemplate template) {
		Collection<String> fromHeader = template.headers().get(SecurityConstants.FROM);
		if (CollUtil.isNotEmpty(fromHeader) && fromHeader.contains(SecurityConstants.FROM_IN)) {
			return;
		}

		accessTokenContextRelay.copyToken();
		if (oAuth2ClientContext != null
			&& oAuth2ClientContext.getAccessToken() != null) {
			super.apply(template);
		}
	}
}
# 无token请求，服务内部发起情况处理。
![avator](http://pic.pig4cloud.com/20190220195955_nDy07g_to.jpeg)



很多情况下，比如定时任务。A服务并没有token 去请求B服务，pig也对这种情况进行了兼容。类似于A对外暴露API，但是又安全限制。参考日志插入情况

FeignClient 需要带一个请求token,FROM_IN 声明是内部调用
remoteLogService.saveLog(sysLog, SecurityConstants.FROM_IN);

目标接口对内外调用进行限制 @Inner 注解，这样就避免接口对外暴露的安全问题。只能通过内部调用才能使用，浏览器不能直接访问该接口
	@Inner
	@PostMapping
	public R save(@Valid @RequestBody SysLog sysLog) {
		return new R<>(sysLogService.save(sysLog));
	}
# 后边专门讲@Inner 注解