当用户携带token 请求资源服务器的资源时, OAuth2AuthenticationProcessingFilter 拦截token，进行token 和userdetails 过程，把无状态的token 转化成用户信息。

image
![avator](http://a.pig4cloud.com/20190125162610.png)


# 详解
OAuth2AuthenticationManager.authenticate()，filter执行判断的入口 image

![avator](http://a.pig4cloud.com/20190125160252.png)

当用户携带token 去请求微服务模块，被资源服务器拦截调用RemoteTokenServices.loadAuthentication ,执行所谓的check-token过程。 源码如下 image
![avator](http://a.pig4cloud.com/20190125150127.png)

CheckToken 处理逻辑很简单，就是调用redisTokenStore 查询token的合法性，及其返回用户的部分信息 （username ）

image
![avator](http://a.pig4cloud.com/20190125150237.png)


继续看 返回给 RemoteTokenServices.loadAuthentication 最后一句 tokenConverter.extractAuthentication 解析组装服务端返回的信息
image 
![avator](http://a.pig4cloud.com/20190125150335.png)

最重要的 userTokenConverter.extractAuthentication(map);

最重要的一步，是否判断是否有userDetailsService实现，如果有 的话去查根据 返回的 username 查询一次全部的用户信息，没有实现直接返回username，这也是很多时候问的为什么只能查询到username 也就是 EnablePigxResourceServer.details true 和false 的区别。
image
![avator](http://a.pig4cloud.com/20190125150416.png)

那根据的你问题，继续看 UerDetailsServiceImpl.loadUserByUsername 根据用户名去换取用户全部信息。 image
![avator](http://a.pig4cloud.com/20190125150620.png)