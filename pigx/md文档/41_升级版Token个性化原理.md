默认通过调用 /oauth/token 返回的报文格式包含以下参数
{
    "access_token": "e6669cdf-b6cd-43fe-af5c-f91a65041382",
    "token_type": "bearer",
    "refresh_token": "da91294d-446c-4a89-bdcf-88aee15a75e8",
    "expires_in": 43199, 
    "scope": "server"
}
并没包含用户的业务信息比如用户信息、租户信息等。

扩展生成包含业务信息（如下）,避免系统多次调用，直接可以通过认证接口获取到用户信息等，大大提高系统性能
{
    "access_token":"a6f3b6d6-93e6-4eb8-a97d-3ae72240a7b0",
    "token_type":"bearer",
    "refresh_token":"710ab162-a482-41cd-8bad-26456af38e4f",
    "expires_in":42396,
    "scope":"server",
    "tenant_id":1,
    "license":"made by pigx",
    "dept_id":1,
    "user_id":1,
    "username":"admin"
}
# 密码模式生成Token 源码解析
image

![avator](http://pic.pig4cloud.com/20190706114317_UnHHZL_Screenshot.jpeg)
​ 主页参考红框部分

ResourceOwnerPasswordTokenGranter （密码模式）根据用户的请求信息，进行认证得到当前用户上下文信息

protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {
    Map<String, String> parameters = new LinkedHashMap<String, String>(tokenRequest.getRequestParameters());
	String username = parameters.get("username");
	String password = parameters.get("password");
	// Protect from downstream leaks of password
	parameters.remove("password");
    Authentication userAuth = new UsernamePasswordAuthenticationToken(username, password);
   ((AbstractAuthenticationToken) userAuth).setDetails(parameters);
		
    userAuth = authenticationManager.authenticate(userAuth);

    OAuth2Request storedOAuth2Request =  getRequestFactory().createOAuth2Request(client, tokenRequest);		
		return new OAuth2Authentication(storedOAuth2Request, userAuth);
}
然后调用AbstractTokenGranter.getAccessToken() 获取OAuth2AccessToken

protected OAuth2AccessToken getAccessToken(ClientDetails client, TokenRequest tokenRequest) {
   return tokenServices.createAccessToken(getOAuth2Authentication(client, tokenRequest));
}
默认使用DefaultTokenServices来获取token

public OAuth2AccessToken createAccessToken(OAuth2Authentication authentication) throws AuthenticationException {

	... 一系列判断 ，合法性、是否过期等判断	
	OAuth2AccessToken accessToken = createAccessToken(authentication, refreshToken);
		tokenStore.storeAccessToken(accessToken, authentication);
		// In case it was modified
		refreshToken = accessToken.getRefreshToken();
		if (refreshToken != null) {
			tokenStore.storeRefreshToken(refreshToken, authentication);
		}
		return accessToken;
}

createAccessToken 核心逻辑

// 默认刷新token 的有效期
private int refreshTokenValiditySeconds = 60 * 60 * 24 * 30; // default 30 days.
// 默认token 的有效期
private int accessTokenValiditySeconds = 60 * 60 * 12; // default 12 hours.

private OAuth2AccessToken createAccessToken(OAuth2Authentication authentication, OAuth2RefreshToken refreshToken) {
    DefaultOAuth2AccessToken token = new DefaultOAuth2AccessToken(uuid);
    token.setExpiration(Date)
    token.setRefreshToken(refreshToken);
    token.setScope(authentication.getOAuth2Request().getScope());
    return accessTokenEnhancer != null ? accessTokenEnhancer.enhance(token, authentication) : token;
}
如上代码，在拼装好token对象后会调用认证服务器配置TokenEnhancer( 增强器) 来对默认的token进行增强。

TokenEnhancer.enhance 通过上下文中的用户信息来个性化Token

public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
    final Map<String, Object> additionalInfo = new HashMap<>(8);
    PigxUser pigxUser = (PigxUser) authentication.getUserAuthentication().getPrincipal();
    additionalInfo.put("user_id", pigxUser.getId());
    additionalInfo.put("username", pigxUser.getUsername());
    additionalInfo.put("dept_id", pigxUser.getDeptId());
    additionalInfo.put("tenant_id", pigxUser.getTenantId());
    additionalInfo.put("license", SecurityConstants.PIGX_LICENSE);
    ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
    return accessToken;
}