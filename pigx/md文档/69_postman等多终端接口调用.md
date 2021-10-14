# 通过网关访问auth-server 获取access-token
请求地址为 http://pig:9999/auth/oauth/token
headr参数为： Authorization Basic dGVzdDp0ZXN0， 这里为Base64(test:test),只能使用test终端，其他终端需要输验证码，可以参考 验证码处理章节 

![avator](http://pic.pig4cloud.com/20190220193045_WO4FXU_Screenshot.jpeg)


参数如下 
![avator](http://pic.pig4cloud.com/20190220193128_xICRpK_Screenshot.jpeg)

# 通过access-token 访问受保护的资源

![avator](http://pic.pig4cloud.com/20190220193607_82J9x6_Screenshot.jpeg)
# 刷新token
请求接口通获取token接口一致，header参数一致 

![avator](http://pic.pig4cloud.com/20190220193837_cgrdQp_Screenshot.jpeg)

注意grant_type refresh_token 
![avator](http://pic.pig4cloud.com/20190220193926_VU9Xag_Screenshot.jpeg)