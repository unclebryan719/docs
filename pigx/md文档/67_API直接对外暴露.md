# 现象
如果微服务接入了资源服务器，那么全部的资源被spring security oauth 拦截，如果没有合法token 直接会被拒绝。 如下图,提示如下错误。 
![avator](http://pic.pig4cloud.com/20190220194244_ZKCblx_Screenshot.jpeg)
# 服务暴露
3.2+ 版本直接在接口配置,若封装接口接口（例如swagger等）无法加，可以直接参考下文配置文件中声明
// 如果配置在controller类上 是整个类的接口对外暴露
@Inner(value = false)
@GetMapping("/")
public R api() {
}
低版本直接在对应微服务模块配置
security:
  oauth2:
    client:
      # 默认放行url,如果子模块重写这里的配置就会被覆盖
      ignore-urls:
        - /actuator/**
        - /v2/api-docs
        - 目标借口的Ant表达式即可


![avator](http://pic.pig4cloud.com/20190228174136_aisjwl_Screenshot.jpeg)