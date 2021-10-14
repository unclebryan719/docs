在微服务架构下，通常每个微服务都会使用Swagger来管理我们的接口文档，当微服务越来越多，接口查找管理无形中要浪费我们不少时间，毕竟懒是程序员的美德。

​ 由于swagger2暂时不支持webflux 走了很多坑，完成这个效果感谢 @dreamlu @世言。

# 文档聚合效果
通过访问网关的 host:port/swagger-ui.html，即可实现: pig聚合文档效果预览传送门

通过右上角的Select a spec 选择服务模块来查看swagger文档 image
![avatar](https://ask.qcloudimg.com/http-save/1219867/0mzen0pzi3.png?imageView2/2/w/1620)


# PigX的Spring Cloud Gateway 实现
# 注入路由到SwaggerResource
@Component
@Primary
@AllArgsConstructor
public class SwaggerProvider implements SwaggerResourcesProvider {
	public static final String API_URI = "/v2/api-docs";
	private final RouteLocator routeLocator;
	private final GatewayProperties gatewayProperties;


	@Override
	public List<SwaggerResource> get() {
		List<SwaggerResource> resources = new ArrayList<>();
		List<String> routes = new ArrayList<>();
		routeLocator.getRoutes().subscribe(route -> routes.add(route.getId()));
		gatewayProperties.getRoutes().stream().filter(routeDefinition -> routes.contains(routeDefinition.getId()))
			.forEach(routeDefinition -> routeDefinition.getPredicates().stream()
				.filter(predicateDefinition -> "Path".equalsIgnoreCase(predicateDefinition.getName()))
				.filter(predicateDefinition -> !"pigx-auth".equalsIgnoreCase(routeDefinition.getId()))
				.forEach(predicateDefinition -> resources.add(swaggerResource(routeDefinition.getId(),
					predicateDefinition.getArgs().get(NameUtils.GENERATED_NAME_PREFIX + "0")
						.replace("/**", API_URI)))));
		return resources;
	}

	private SwaggerResource swaggerResource(String name, String location) {
		SwaggerResource swaggerResource = new SwaggerResource();
		swaggerResource.setName(name);
		swaggerResource.setLocation(location);
		swaggerResource.setSwaggerVersion("2.0");
		return swaggerResource;
	}
}

# 提供swagger 对外接口配置


@Slf4j
@Configuration
@AllArgsConstructor
public class RouterFunctionConfiguration {
	private final SwaggerResourceHandler swaggerResourceHandler;
	private final SwaggerSecurityHandler swaggerSecurityHandler;
	private final SwaggerUiHandler swaggerUiHandler;

	@Bean
	public RouterFunction routerFunction() {
		return RouterFunctions.route(
			.andRoute(RequestPredicates.GET("/swagger-resources")
				.and(RequestPredicates.accept(MediaType.ALL)), swaggerResourceHandler)
			.andRoute(RequestPredicates.GET("/swagger-resources/configuration/ui")
				.and(RequestPredicates.accept(MediaType.ALL)), swaggerUiHandler)
			.andRoute(RequestPredicates.GET("/swagger-resources/configuration/security")
				.and(RequestPredicates.accept(MediaType.ALL)), swaggerSecurityHandler);

	}
}
# 业务handler 的实现
	@Override
	public Mono<ServerResponse> handle(ServerRequest request) {
		return ServerResponse.status(HttpStatus.OK)
			.contentType(MediaType.APPLICATION_JSON_UTF8)
			.body(BodyInserters.fromObject(swaggerResources.get()));
	}

    @Override
	public Mono<ServerResponse> handle(ServerRequest request) {
		return ServerResponse.status(HttpStatus.OK)
			.contentType(MediaType.APPLICATION_JSON_UTF8)
			.body(BodyInserters.fromObject(
				Optional.ofNullable(securityConfiguration)
					.orElse(SecurityConfigurationBuilder.builder().build())));
	}

    @Override
    public Mono<ServerResponse> handle(ServerRequest request) {
        return ServerResponse.status(HttpStatus.OK)
			.contentType(MediaType.APPLICATION_JSON_UTF8)
			.body(BodyInserters.fromObject(
				Optional.ofNullable(uiConfiguration)
					.orElse(UiConfigurationBuilder.builder().build())));
	}
# swagger路径转换
通过以上配置，可以实现文档的参考和展示了，但是使用swagger 的 try it out 功能发现路径是路由切割后的路径比如：

swagger 文档中的路径为： 主机名：端口：映射路径 少了一个 服务路由前缀，是因为展示handler 经过了 StripPrefixGatewayFilterFactory 这个过滤器的处理，原有的 路由前缀被过滤掉了！

# 方案1，通过swagger 的host 配置手动维护一个前缀
return new Docket(DocumentationType.SWAGGER_2)
    .apiInfo(apiInfo())
    .host("主机名：端口：服务前缀")  //注意这里的主机名：端口是网关的地址和端口
    .select()
    .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
    .paths(PathSelectors.any())
    .build()
    .globalOperationParameters(parameterList);
# 方案2，增加X-Forwarded-Prefix
swagger 在拼装URL 数据时候，会增加X-Forwarder-Prefix 请求头里面的信息为前缀
![avatar](https://ask.qcloudimg.com/http-save/1219867/p9ix1y3i9o.png?imageView2/2/w/1620)
imageimage 通过如上分析，知道应该在哪里下手了吧，在 网关上追加一个请求头即可

@Component
public class SwaggerHeaderFilter extends AbstractGatewayFilterFactory {
	private static final String HEADER_NAME = "X-Forwarded-Prefix";

	@Override
	public GatewayFilter apply(Object config) {
		return (exchange, chain) -> {
			ServerHttpRequest request = exchange.getRequest();
			String path = request.getURI().getPath();
			if (!StringUtils.endsWithIgnoreCase(path, SwaggerProvider.API_URI)) {
				return chain.filter(exchange);
			}

			String basePath = path.substring(0, path.lastIndexOf(SwaggerProvider.API_URI));


			ServerHttpRequest newRequest = request.mutate().header(HEADER_NAME, basePath).build();
			ServerWebExchange newExchange = exchange.mutate().request(newRequest).build();
			return chain.filter(newExchange);
		};
	}
}