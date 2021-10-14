客户端模式,没有具体的用户信息, 只有客户的client-id 信息 ,完成不了
upms的RBAC ,现有认证中心不支持放发Token

怎么支持认证中心发放token，有了token ，我其他的微服务不用upms 是否可以？ 当然 AuthorizationServerConfig.java 修改token 增强逻辑，如果为客户端模式就不进行增强，即不维护用户信息。

	/**
	 * token增强，客户端模式不增强。
	 *
	 * @return TokenEnhancer
	 */
	@Bean
	public TokenEnhancer tokenEnhancer() {
		return (accessToken, authentication) -> {
			if (SecurityConstants.CLIENT_CREDENTIALS
					.equals(authentication.getOAuth2Request().getGrantType())) {
				return accessToken;
			}

			final Map<String, Object> additionalInfo = new HashMap<>(8);
			PigxUser pigxUser = (PigxUser) authentication.getUserAuthentication().getPrincipal();
			additionalInfo.put("user_id", pigxUser.getId());
			additionalInfo.put("username", pigxUser.getUsername());
			additionalInfo.put("dept_id", pigxUser.getDeptId());
			additionalInfo.put("tenant_id", pigxUser.getTenantId());
			additionalInfo.put("license", SecurityConstants.PIGX_LICENSE);
			((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
			return accessToken;
		};
	}
即可方法令牌了，只是这个令牌只有客户端信息。 为何不增强参考 源码DefaultTokenServices

	private OAuth2AccessToken createAccessToken(OAuth2Authentication authentication, OAuth2RefreshToken refreshToken) {
		DefaultOAuth2AccessToken token = new DefaultOAuth2AccessToken(UUID.randomUUID().toString());
		int validitySeconds = getAccessTokenValiditySeconds(authentication.getOAuth2Request());
		if (validitySeconds > 0) {
			token.setExpiration(new Date(System.currentTimeMillis() + (validitySeconds * 1000L)));
		}
		token.setRefreshToken(refreshToken);
		token.setScope(authentication.getOAuth2Request().getScope());

		return accessTokenEnhancer != null ? accessTokenEnhancer.enhance(token, authentication) : token;
	}
不修改，没有用户信息再去调用原有增强逻辑会报错。