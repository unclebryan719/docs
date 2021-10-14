大家好，我是世言，从Pigx创建到现在，我经历了从最初只有几个人到现在几千人的大群。最近看群里很多童鞋都在问Inner注解的原理和作用,写了个对Pigx里的@Inner注解的理解，希望可以帮助大家.

# Inner的诞生
Pigx种Url的访问大致分为两种，一种是由Gateway网关从外网路由进来，一种是内网通过Feign进行内部服务调用。那我们结合Security就存在以下几种应用场景：

外部从Gateway访问，需要鉴权（eg.CURD操作）。这种是最常使用的，用户登录后正常访问接口，不需要我们做什么处理（可能有的接口需要加权限字段）。
外部从Gateway访问，不需要鉴权（eg.短信验证码）。需要我们将uri加入到security.oauth2.client.ignore-urls配置中，可以不需要鉴权访问
内部服务间用Feign访问，不需要鉴权（eg.Auth查询用户信息）。也是需要我们将uri加入到security.oauth2.client.ignore-urls配置中，那与第二种的区别就是这种情况下大多数都是服务可以请求另一个服务的所有数据，不受约束，那我们如果仅仅只配置ignore-url的话，外部所有人都可以通过url请求到我们内部的链接，安全达不到保障。
鉴于上述第三种情况，我们配置了ignore-url和Feign，此时该接口不需要鉴权，服务内部通过Feign访问，服务外部通过url也可以访问，所以Pigx中，加入了一种@RequestHeader(SecurityConstants.FROM)的处理方式。即在接口方法中，对头部进行判断，只有请求带上相应的Header参数时，才允许通过访问，否则抛出异常。那这时候其实我们在外网通过Gateway访问的时候，也可以手动带上这个Header参数，来达到这个目的。所以我们便在Gateway中设置了一个GlobalFilter过滤器：

![avator](http://qiniu.wxdfun.top/pigx/EWF%5B7%28G68%5B7Q4R3HIY4UX@H.png)

这个过滤器在处理HttpRequest的时候，会删除从外部请求头里的SecurityConstants.FROM这个参数。此时的效果就是，这个URL从外部访问不需要鉴权，但由于Gateway的过滤，最终到达我们接口方法时，由于缺少头部信息，被拒绝访问；而服务间通过Feign访问，不经过Gateway，则可以正常访问。

那原始的处理方法和处理逻辑就是这样：首先将uri加入ingore-url，然后在接口的方法和Feign的接口参数中写上RequestHeader参数，最后在Feign-Client中带上这个SecurityConstants.FROM参数。既然这种逻辑都是相同的，那后面的pigx版本发行后，就使用AOP将此步骤抽离出来，成为了Inner。

# Inner的处理流程
# 统一的ignore-url处理
首先我们来看看这个注解的代码

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Inner {
	/**
	 * 是否AOP统一处理(可以理解为是否仅允许Feign之间调用)
	 *
	 * @return false, true
	 */
	boolean value() default true;

	/**
	 * 需要特殊判空的字段(预留)
	 *
	 * @return {}
	 */
	String[] field() default {};
}
首先，在我们项目加载阶段，我们获取有Inner注解的类和方法，然后获取我们配置的uri，经过正则替换后面的可变参数为*，然后将此uri加入到ignore-url中。此时我们就能达到所有Inner配置的方法/类上的接口地址，都统一在项目加载阶段自动帮我们加到ignore-url中，不需要我们手动配置，免去了很多开发工作，同时也能避免我们忘记配置，而浪费开发时间。核心代码如下：

@Slf4j
@Configuration
@ConditionalOnExpression("!'${security.oauth2.client.ignore-urls}'.isEmpty()")
@ConfigurationProperties(prefix = "security.oauth2.client")
public class PermitAllUrlProperties implements InitializingBean {
	private static final Pattern PATTERN = Pattern.compile("\\{(.*?)\\}");
	@Autowired
	private WebApplicationContext applicationContext;
	@Getter
	@Setter
	private List<String> ignoreUrls = new ArrayList<>();
	@Override
	public void afterPropertiesSet() {
		RequestMappingHandlerMapping mapping = applicationContext.getBean(RequestMappingHandlerMapping.class);
		Map<RequestMappingInfo, HandlerMethod> map = mapping.getHandlerMethods();
		map.keySet().forEach(info -> {
			HandlerMethod handlerMethod = map.get(info);
			// 获取方法上边的注解 替代path variable 为 *
			Inner method = AnnotationUtils.findAnnotation(handlerMethod.getMethod(), Inner.class);
			Optional.ofNullable(method)
					.ifPresent(inner -> info.getPatternsCondition().getPatterns()
							.forEach(url -> ignoreUrls.add(ReUtil.replaceAll(url, PATTERN, StringPool.ASTERISK))));
			// 获取类上边的注解, 替代path variable 为 *
			Inner controller = AnnotationUtils.findAnnotation(handlerMethod.getBeanType(), Inner.class);
			Optional.ofNullable(controller)
					.ifPresent(inner -> info.getPatternsCondition().getPatterns()
							.forEach(url -> ignoreUrls.add(ReUtil.replaceAll(url, PATTERN, StringPool.ASTERISK))));
		});
	}
}
# 统一的安全性处理
那上面讲到的，如果我们不希望这个url可以直接被外网调用，仅能在Feign服务中调用，改如何统一处理呢？

我们使用一个Spring-AOP，在对所有Inner注解的方法做一个环绕增强的切点，进行统一的处理。在上面我们提到的Inner的value参数，当该参数为true时，我们对方法的入参进行判断，仅当符合我们定制的入参规则时（Pigx这里是用的@RequestHeader(SecurityConstants.FROM) 与SecurityConstants.FROM_IN做比较）,我们对它进行放行，不符合时，抛出异常；当value为false时，咱不做任何处理，此时Inner仅起到了一个ignore-url的作用。

@Slf4j
@Aspect
@Component
@AllArgsConstructor
public class PigxSecurityInnerAspect {
	private final HttpServletRequest request;
	@SneakyThrows
	@Around("@annotation(inner)")
	public Object around(ProceedingJoinPoint point, Inner inner) {
		String header = request.getHeader(SecurityConstants.FROM);
		if (inner.value() && !StrUtil.equals(SecurityConstants.FROM_IN, header)) {
			log.warn("访问接口 {} 没有权限", point.getSignature().getName());
			throw new AccessDeniedException("Access is denied");
		}
		return point.proceed();
	}
}
通过这两步呢，我们首先是在加载时通过找到Inner注解，将相应的uri加入到ignore-url中，达到自动化配置的目的；之后我们又使用切面对Inner的方法进行环绕处理，达到安全控制。对比之前的处理方式，现在我们使用一个@Inner注解，就能很快的满足上面说的两种场景，大大节省了我们的开发时间。

# 合理的使用@Inner注解
上面提到的两种应用场景，在我们的代码中，其实都是可以使用Inner注解的，下面结合Feign做一个简单的示例，示例场景就是我们的用户密码登录中的一环：

在接口上使用@Inner注解，使得url无需鉴权

	/**
	 * 获取指定用户全部信息
	 *
	 * @return 用户信息
	 */
	@Inner
	@GetMapping("/info/{username}")
	public R info(@PathVariable String username) {
		SysUser user = userService.getOne(Wrappers.<SysUser>query()
				.lambda().eq(SysUser::getUsername, username));
		if (user == null) {
			return R.failed(null, String.format("用户信息为空 %s", username));
		}
		return R.ok(userService.findUserInfo(user));
	}
编写Feign接口

@FeignClient(contextId = "remoteUserService", value = ServiceNameConstants.UMPS_SERVICE)
public interface RemoteUserService {
	/**
	 * 通过用户名查询用户、角色信息
	 *
	 * @param username 用户名
	 * @param from     调用标志
	 * @return R
	 */
	@GetMapping("/user/info/{username}")
	R<UserInfo> info(@PathVariable("username") String username
			, @RequestHeader(SecurityConstants.FROM) String from);
}
Feign-Client中调用接口，带上SecurityConstants.FROM_IN参数为内部识别

	/**
	 * 用户密码登录
	 *
	 * @param username 用户名
	 * @return
	 * @throws UsernameNotFoundException
	 */
	@Override
	@SneakyThrows
	public UserDetails loadUserByUsername(String username) {
		Cache cache = cacheManager.getCache(CacheConstants.USER_DETAILS);
		if (cache != null && cache.get(username) != null) {
			return (PigxUser) cache.get(username).get();
		}
		R<UserInfo> result = remoteUserService.info(username, SecurityConstants.FROM_IN);
		UserDetails userDetails = getUserDetails(result);
		cache.put(username, userDetails);
		return userDetails;
	}
现在"/info/{username}" 这个uri从网关外部我们访问是报错的（一般来说服务都是走网关暴露接口），而Feign内部带上参数是可以正常访问的。大功告成喝杯啤酒。