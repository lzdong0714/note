---

typora-root-url: Pic4Security
---

## 主题

### 登录Filter解析

authentication: 身份验证

authorization:授权

![](/企业微信截图_15784473033170.png)

Filter 是作用与SpringMVC的Dispatched之前，直接作用的是servlet，而Intercepter是作用在dispatch之后，作用的是针对映射的control之前。所以看到FilterChainProxy作用都是直接ServletRequest request, ServletResponse response

登录filterchain

![](/企业微信截图_15784474062641.png)

``` java

```



#### OncePerRequestFilter

代理了一些filter的作用:

```java
WebAsyncManagerIntegrationFilter
HeaderWriterFilter
BasicAuthenticationFilter
```

 #### SecurityContextPersistenceFilter

根据说明，是建立SecurityContext的上下文，每次request建立一次filter，确保SecurityContextHolder中包含有效的SecurityContext，似乎使用session来保存

``` java
public class SecurityContextPersistenceFilter extends GenericFilterBean {
	// ...
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		if (request.getAttribute(FILTER_APPLIED) != null) {
			// ensure that filter is only applied once per request
			chain.doFilter(request, response);
			return;
		}

		final boolean debug = logger.isDebugEnabled();
		// 设定为一次的访问
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);

		if (forceEagerSessionCreation) {
			HttpSession session = request.getSession();

			if (debug && session.isNew()) {
				logger.debug("Eagerly created session: " + session.getId());
			}
		}
		// 获取一次访问的上下文holder
		HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request,
				response);
        // 建立一次访问的SecurityContext
		SecurityContext contextBeforeChainExecution = repo.loadContext(holder);

		try {
            // 添加到SecurityContextHolder中
			SecurityContextHolder.setContext(contextBeforeChainExecution);
			
            // 后续调用放在了try中，用finally保证最后总会执行清除上下文
			chain.doFilter(holder.getRequest(), holder.getResponse());

		}
		finally {
            // 调用完毕后，清除这次的上下文SecurityContext
			SecurityContext contextAfterChainExecution = SecurityContextHolder
					.getContext();
			// Crucial removal of SecurityContextHolder contents - do this before anything
			// else.
			SecurityContextHolder.clearContext();
			repo.saveContext(contextAfterChainExecution, holder.getRequest(),
					holder.getResponse());
			request.removeAttribute(FILTER_APPLIED);

			if (debug) {
				logger.debug("SecurityContextHolder now cleared, as request processing completed");
			}
		}
	}
    
}
```

#### LogoutFilter

处理退出，成功退出会被重定向到LogoutSuccessHandler中设定的url， 或者logoutSuccessUrl的属性的设顶url中。

``` java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
		// 匹配是否是登出的request
		if (requiresLogout(request, response)) {
            // 上下文获取身份认证
			Authentication auth = SecurityContextHolder.getContext().getAuthentication();

			if (logger.isDebugEnabled()) {
				logger.debug("Logging out user '" + auth
						+ "' and transferring to logout destination");
			}

			this.handler.logout(request, response, auth);

			logoutSuccessHandler.onLogoutSuccess(request, response, auth);

			return;
		}

		chain.doFilter(request, response);
	}
```



![企业微信截图_157838075422](/企业微信截图_157838075422.png)

可见大概还是url匹配的模式，确定是否是登出请求，没有匹配成功直接执行下一个。



#### ClientCredentialsTokenEndpointFilter

先进入父类AbstractAuthenticationProcessingFilter中做url验证，是否是获取token的登录请求，如果是，

那么就进入ClientCredentialsTokenEndpointFilter中的attemptAuthentication()方法去验证，

attemptAuthentication()中会继续去调用ProviderManager的authenticate()方法去验证，这里就是添加自定义用户验证进行验证的地方。



``` java
public class ClientCredentialsTokenEndpointFilter extends AbstractAuthenticationProcessingFilter {
    ...
     // 继承于AbstractAuthenticationProcessingFilter， 主要是映射默认的验证路径
        "/oauth/token"
       
@Override 
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

		if (allowOnlyPost && !"POST".equalsIgnoreCase(request.getMethod())) {
			throw new HttpRequestMethodNotSupportedException(request.getMethod(), new String[] { "POST" });
		}

		String clientId = request.getParameter("client_id");
		String clientSecret = request.getParameter("client_secret");

		// If the request is already authenticated we can assume that this
		// filter is not needed
        // 验证上下文中是否有authentication ，有就直接返回。
		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authentication != null && authentication.isAuthenticated()) {
			return authentication;
		}

		if (clientId == null) {
			throw new BadCredentialsException("No client credentials presented");
		}

		if (clientSecret == null) {
			clientSecret = "";
		}

		clientId = clientId.trim();
        //  返回一个客户端 id 和 secret 的authentication
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(clientId,
				clientSecret);
		
        // 进入到ProviderManager 中去验证
		return this.getAuthenticationManager().authenticate(authRequest);

	}
}
```

登录验证主要看这个AbstractAuthenticationProcessingFilter作用干了什么：

``` java 
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean
		implements ApplicationEventPublisherAware, MessageSourceAware {
		//...
		
		public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
		// 验证是否是登录请求，还是通过url匹配： 见图
		// 不是登录验证就直接下一步
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);

			return;
		}

		if (logger.isDebugEnabled()) {
			logger.debug("Request is to process authentication");
		}

		Authentication authResult;

		try {
		// 尝试获取认证匹配
		// 实际调用ClientCredentialsTokenEndpointFilter的attemptAuthentication方法
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				// authentication
				return;
			}
			sessionStrategy.onAuthentication(authResult, request, response);
		}
		catch (InternalAuthenticationServiceException failed) {
			logger.error(
					"An internal error occurred while trying to authenticate the user.",
					failed);
			unsuccessfulAuthentication(request, response, failed);

			return;
		}
		catch (AuthenticationException failed) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, failed);

			return;
		}

		// Authentication success
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}

		successfulAuthentication(request, response, chain, authResult);
	}
	// .........	
 }
```



``` java
public class ProviderManager implements AuthenticationManager, MessageSourceAware,
		InitializingBean {

public Authentication authenticate(Authentication authentication)throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();
 // 遍历provider验证器
		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}

			if (debug) {
				logger.debug("Authentication attempt using "
						+ provider.getClass().getName());
			}

			try {
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			}
			catch (InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				throw e;
			}
			catch (AuthenticationException e) {
				lastException = e;
			}
		}

		if (result == null && parent != null) {
			// Allow the parent to try.
			try {
				result = parentResult = parent.authenticate(authentication);
			}
			catch (ProviderNotFoundException e) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException e) {
				lastException = parentException = e;
			}
		}

		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).

		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}

		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}

		throw lastException;
	}
            
            
}
```

![](/企业微信截图_15783859033718.png)

这里验证通过就

#### LoginLoggingFilter

自定义的插入日志filter，记录登录信息

#### RequestCacheAwareFilter

？？ 啥也没干

#### SecurityContextHolderAwareRequestFilter

？？ 没看懂

``` java

```





#### AnonymousAuthenticationFilter

如果认证身份为空，那么就建立一个匿名身份：

``` java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		//如果认证身份为空，那么就建立一个匿名身份
		if (SecurityContextHolder.getContext().getAuthentication() == null) {
			SecurityContextHolder.getContext().setAuthentication(
					createAuthentication((HttpServletRequest) req));

			if (logger.isDebugEnabled()) {
				logger.debug("Populated SecurityContextHolder with anonymous token: '"
						+ SecurityContextHolder.getContext().getAuthentication() + "'");
			}
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("SecurityContextHolder not populated with anonymous token, as it already contained: '"
						+ SecurityContextHolder.getContext().getAuthentication() + "'");
			}
		}

		chain.doFilter(req, res);
	}
// 如果认证身份为空，那么就建立一个匿名身份，建造匿名身份方法
protected Authentication createAuthentication(HttpServletRequest request) {
		AnonymousAuthenticationToken auth = new AnonymousAuthenticationToken(key,
				principal, authorities);
		auth.setDetails(authenticationDetailsSource.buildDetails(request));

		return auth;
	}
```



#### SessionManagementFilter

查用户身份，做一些于session相关的操作， 我们没用到，直接调用返回。

``` java
/**
 * Detects that a user has been authenticated since the start of the request and, if they
 * have, calls the configured {@link SessionAuthenticationStrategy} to perform any
 * session-related activity such as activating session-fixation protection mechanisms or
 * checking for multiple concurrent logins.
 *
 * @author Martin Algesten
 * @author Luke Taylor
 * @since 2.0
 */
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
		// 检查是否是有session 操作标记
		if (request.getAttribute(FILTER_APPLIED) != null) {
			chain.doFilter(request, response);
			return;
		}
		// 没有的话，这里添加操作标记
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);

		if (!securityContextRepository.containsContext(request)) {
			Authentication authentication = SecurityContextHolder.getContext()
					.getAuthentication();

			if (authentication != null && !trustResolver.isAnonymous(authentication)) {
				// The user has been authenticated during the current request, so call the
				// session strategy
				try {
					sessionAuthenticationStrategy.onAuthentication(authentication,
							request, response);
				}
				catch (SessionAuthenticationException e) {
					// The session strategy can reject the authentication
					logger.debug(
							"SessionAuthenticationStrategy rejected the authentication object",
							e);
					SecurityContextHolder.clearContext();
					failureHandler.onAuthenticationFailure(request, response, e);

					return;
				}
				// Eagerly save the security context to make it available for any possible
				// re-entrant
				// requests which may occur before the current request completes.
				// SEC-1396.
				securityContextRepository.saveContext(SecurityContextHolder.getContext(),
						request, response);
			}
			else {
				// No security context or authentication present. Check for a session
				// timeout
				if (request.getRequestedSessionId() != null
						&& !request.isRequestedSessionIdValid()) {
					if (logger.isDebugEnabled()) {
						logger.debug("Requested session ID "
								+ request.getRequestedSessionId() + " is invalid.");
					}

					if (invalidSessionStrategy != null) {
						invalidSessionStrategy
								.onInvalidSessionDetected(request, response);
						return;
					}
				}
			}
		}

		chain.doFilter(request, response);
	}
```



#### ExceptionTranslationFilter

异常信息处理器，可以自己添加异常信息处理。

``` java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		try {
            // 不干嘛，直接try-catch 
			chain.doFilter(request, response);

			logger.debug("Chain processed normally");
		}
		catch (IOException ex) {
			throw ex;
		}
		catch (Exception ex) {
			// Try to extract a SpringSecurityException from the stacktrace
			Throwable[] causeChain = throwableAnalyzer.determineCauseChain(ex);
			RuntimeException ase = (AuthenticationException) throwableAnalyzer
					.getFirstThrowableOfType(AuthenticationException.class, causeChain);

			if (ase == null) {
				ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(
						AccessDeniedException.class, causeChain);
			}

			if (ase != null) {
				if (response.isCommitted()) {
					throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", ex);
				}
				handleSpringSecurityException(request, response, chain, ase);
			}
			else {
				// Rethrow ServletExceptions and RuntimeExceptions as-is
				if (ex instanceof ServletException) {
					throw (ServletException) ex;
				}
				else if (ex instanceof RuntimeException) {
					throw (RuntimeException) ex;
				}

				// Wrap other Exceptions. This shouldn't actually happen
				// as we've already covered all the possibilities for doFilter
				throw new RuntimeException(ex);
			}
		}
	}
```



#### FilterSecurityInterceptor

权限验证的地方

```java
public class FilterSecurityInterceptor extends AbstractSecurityInterceptor implements
		Filter {	
public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
    // 打包为一个FilterInvocation，调用invoke
		FilterInvocation fi = new FilterInvocation(request, response, chain);
		invoke(fi);
	}

public void invoke(FilterInvocation fi) throws IOException, ServletException {
		if ((fi.getRequest() != null)
				&& (fi.getRequest().getAttribute(FILTER_APPLIED) != null)
				&& observeOncePerRequest) {
			// filter already applied to this request and user wants us to observe
			// once-per-request handling, so don't re-do security checking
			fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
		}
		else {
			// first time this request being called, so perform security checking
			if (fi.getRequest() != null && observeOncePerRequest) {
				fi.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
			}
			// 调用父类AbstractSecurityInterceptor去获获取token
			InterceptorStatusToken token = super.beforeInvocation(fi);

			try {
				fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
			}
			finally {
				super.finallyInvocation(token);
			}

			super.afterInvocation(token, null);
		}
	}
}
```



``` java
public abstract class AbstractSecurityInterceptor implements InitializingBean,
		ApplicationEventPublisherAware, MessageSourceAware {

            //
   protected InterceptorStatusToken beforeInvocation(Object object) {
       // 前面传递的是一个打包的FilterInvocation，这里是object
       // 做了很多的验证工作，一切顺利的则主要执行，API的权限访问验证
       try {
			this.accessDecisionManager.decide(authenticated, object, attributes);
		}
		catch (AccessDeniedException accessDeniedException) {
			publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated,
					accessDeniedException));

			throw accessDeniedException;
		}
       
   }
            
}
```





### TokenEndpoint解析

来到这里这表名通过dispatch的方式发送给了设置端点

通过验证来到了security设定好的@RequestMapping(value = "/oauth/token", method=RequestMethod.POST)请求端点。

在TokenEndPoint类中：做了客户端的验证，请求模式的识别（token refresh_toke implict )， 又通过getTokenGranter()

``` java
@RequestMapping(value = "/oauth/token", method=RequestMethod.POST)
	public ResponseEntity<OAuth2AccessToken> postAccessToken(Principal principal, @RequestParam
	Map<String, String> parameters) throws HttpRequestMethodNotSupportedException {

		if (!(principal instanceof Authentication)) {
			throw new InsufficientAuthenticationException(
					"There is no client authentication. Try adding an appropriate authentication filter.");
		}

		String clientId = getClientId(principal);
        ///*** 这里调用自己设定的clientService，获取检查的client用户密码信息
		ClientDetails authenticatedClient = getClientDetailsService().loadClientByClientId(clientId);

		TokenRequest tokenRequest = getOAuth2RequestFactory().createTokenRequest(parameters, authenticatedClient);

		if (clientId != null && !clientId.equals("")) {
			// Only validate the client details if a client authenticated during this
			// request.
			if (!clientId.equals(tokenRequest.getClientId())) {
// double check to make sure that the client ID in the token request is the same as that in the
// authenticated client
				throw new InvalidClientException("Given client ID does not match authenticated client");
			}
		}
		if (authenticatedClient != null) {
			oAuth2RequestValidator.validateScope(tokenRequest, authenticatedClient);
		}
		if (!StringUtils.hasText(tokenRequest.getGrantType())) {
			throw new InvalidRequestException("Missing grant type");
		}
		if (tokenRequest.getGrantType().equals("implicit")) {
			throw new InvalidGrantException("Implicit grant type not supported from token endpoint");
		}

		if (isAuthCodeRequest(parameters)) {
// The scope was requested or determined during the authorization step
			if (!tokenRequest.getScope().isEmpty()) {
				logger.debug("Clearing scope of incoming token request");
				tokenRequest.setScope(Collections.<String> emptySet());
			}
		}

		if (isRefreshTokenRequest(parameters)) {
// A refresh token has its own default scopes, so we should ignore any added by the factory here.
			tokenRequest.setScope(OAuth2Utils.parseParameterList(parameters.get(OAuth2Utils.SCOPE)));
		}

        /// **** 这里用进行token的生成，根据参数看出是根据授权方式grantType和上面生成的tokenRequest生成
		OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest);
		if (token == null) {
			throw new UnsupportedGrantTypeException("Unsupported grant type: " + tokenRequest.getGrantType());
		}

		return getResponse(token);

	}
```



OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest);

通过**AbstractTokenGranter**一系列又是复杂的代理，调用了之前的filter，以及授权验证登录中自定义配置的验证，填写token的jwt转化，ClientDetails的填写。

这里通过代理进入到**ResourceOwnerPasswordTokenGranter**中，调用

``` java
protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {

		Map<String, String> parameters = new LinkedHashMap<String, String>(tokenRequest.getRequestParameters());
		String username = parameters.get("username");
		String password = parameters.get("password");
		// Protect from downstream leaks of password
		parameters.remove("password");

		Authentication userAuth = new UsernamePasswordAuthenticationToken(username, password);
		((AbstractAuthenticationToken) userAuth).setDetails(parameters);
		try {
            ///*** 通过代理去做身份验证
			userAuth = authenticationManager.authenticate(userAuth);
		}
		catch (AccountStatusException ase) {
			//covers expired, locked, disabled cases (mentioned in section 5.2, draft 31)
			throw new InvalidGrantException(ase.getMessage());
		}
		catch (BadCredentialsException e) {
			// If the username/password are wrong the spec says we should send 400/invalid grant
			throw new InvalidGrantException(e.getMessage());
		}
		if (userAuth == null || !userAuth.isAuthenticated()) {
			throw new InvalidGrantException("Could not authenticate user: " + username);
		}
		
		OAuth2Request storedOAuth2Request = getRequestFactory().createOAuth2Request(client, tokenRequest);		
		return new OAuth2Authentication(storedOAuth2Request, userAuth);
	}
```

去调用**WebSecurityConfigurerAdapter** 配置类中定义的代理，执行authenticate

以上可见，登陆的时候只有用户密码，那么重要的部分就是身份的验证和token的生成。



### 访问Filter解析

正常每个API的访问，那么主要是token的解析身份验证和API访问权限的验证，以及数据划分的验证。

![](/企业微信截图_15784499188662.png)

前面的于登录一样，少了了一个**ClientCredentialsTokenEndpointFilter**，替换为**OAuth2AuthenticationProcessingFilter**开始不同，用于token的校验。

#### OAuth2AuthenticationProcessingFilter

``` java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException,
			ServletException {

		final boolean debug = logger.isDebugEnabled();
		final HttpServletRequest request = (HttpServletRequest) req;
		final HttpServletResponse response = (HttpServletResponse) res;

		try {
			// 获取token的字段
			Authentication authentication = tokenExtractor.extract(request);
			
			if (authentication == null) {
				if (stateless && isAuthenticated()) {
					if (debug) {
						logger.debug("Clearing security context.");
					}
					SecurityContextHolder.clearContext();
				}
				if (debug) {
					logger.debug("No token in request, will continue chain.");
				}
			}
			else {
				request.setAttribute(OAuth2AuthenticationDetails.ACCESS_TOKEN_VALUE, authentication.getPrincipal());
				if (authentication instanceof AbstractAuthenticationToken) {
					AbstractAuthenticationToken needsDetails = (AbstractAuthenticationToken) authentication;
					needsDetails.setDetails(authenticationDetailsSource.buildDetails(request));
				}
                // 解析token的字段，调用OAuth2AuthenticationManager，继续调用自定义的service
				Authentication authResult = authenticationManager.authenticate(authentication);

				if (debug) {
					logger.debug("Authentication success: " + authResult);
				}

				eventPublisher.publishAuthenticationSuccess(authResult);
				SecurityContextHolder.getContext().setAuthentication(authResult);

			}
		}
		catch (OAuth2Exception failed) {
			SecurityContextHolder.clearContext();

			if (debug) {
				logger.debug("Authentication request failed: " + failed);
			}
			eventPublisher.publishAuthenticationFailure(new BadCredentialsException(failed.getMessage(), failed),
					new PreAuthenticatedAuthenticationToken("access-token", "N/A"));
			// 主要处理token异常带来的OAuth2Exception 问题，通过自定义的异常解析
			authenticationEntryPoint.commence(request, response,
					new InsufficientAuthenticationException(failed.getMessage(), failed));

			return;
		}

		chain.doFilter(request, response);
	}
```



**OAuth2AuthenticationManager**

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		if (authentication == null) {
			throw new InvalidTokenException("Invalid token (token not found)");
		}
		String token = (String) authentication.getPrincipal();
    
    // 调用自定义的tokenService去解析，tokenService中设定的jwtTokenEnhancer
		OAuth2Authentication auth = tokenServices.loadAuthentication(token);
		if (auth == null) {
			throw new InvalidTokenException("Invalid token: " + token);
		}

		Collection<String> resourceIds = auth.getOAuth2Request().getResourceIds();
		if (resourceId != null && resourceIds != null && !resourceIds.isEmpty() && !resourceIds.contains(resourceId)) {
			throw new OAuth2AccessDeniedException("Invalid token does not contain resource id (" + resourceId + ")");
		}

		checkClientDetails(auth);

		if (authentication.getDetails() instanceof OAuth2AuthenticationDetails) {
			OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) authentication.getDetails();
			// Guard against a cached copy of the same details
			if (!details.equals(auth.getDetails())) {
				// Preserve the authentication details from the one loaded by token services
				details.setDecodedDetails(auth.getDetails());
			}
		}
		auth.setDetails(authentication.getDetails());
		auth.setAuthenticated(true);
		return auth;

	}

```

进入**DefaultTokenServices**中执行public OAuth2Authentication loadAuthentication(String accessTokenValue)方法，看到加载的accessTokenValue，解析得到信息为OAuth2AccessToken accessToken 

![](/企业微信截图_15784666426254.png)

### 异常处理机制

**身份验证问题**





#### 登录超时

 #### 统一的异常处理

在TokenEndPoint端参考TokenEndpoint用

``` java
@ExceptionHandler(HttpRequestMethodNotSupportedException.class)
	public ResponseEntity<OAuth2Exception> handleHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) throws Exception {
		if (logger.isInfoEnabled()) {
			logger.info("Handling error: " + e.getClass().getSimpleName() + ", " + e.getMessage());
		}
	    return getExceptionTranslator().translate(e);
	}
}
```



登录验证异常

身份验证异常

访问权限异常





### 涉及的组件类设计

#### 上下文的设计

每一次访问的过程，会建立一个Context用于不同的filter之间传递数据，doFilter(ServletRequest,SevletResponse,FilterChain )函数只是在不断的三个参数中过滤。获取的参数解析以及调用是需存储判断，主要是GrantedAuthority——>包含于Authentication——>包含于SecurityContext ———>包含于SercuityContextHoder中，通过持有策略实际持有SecurityContext ：

1. **ThreadLocal** (线程分割的Map)保存
2. **InheritableThreadLocal**保存
3. **global**保存，即用唯一一个static 属性静态持有

 ![](/SecurityContext.png)

```Java
// Security 中在SecurityContextPersistenceFilter中创建SecurityContext，
HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request,response);
SecurityContext contextBeforeChainExecution = repo.loadContext(holder);
try { // 持有SecurityContext，并继续下一个filter
			SecurityContextHolder.setContext(contextBeforeChainExecution);
			chain.doFilter(holder.getRequest(), holder.getResponse());
}finally { // 最后对访问标识做校验，移除等操作，try-finally构成一个调用链的关系？？
			SecurityContext contextAfterChainExecution = SecurityContextHolder
					.getContext();
			// Crucial removal of SecurityContextHolder contents - do this before anything
			// else.
			SecurityContextHolder.clearContext();
			repo.saveContext(contextAfterChainExecution, holder.getRequest(),
					holder.getResponse());
			request.removeAttribute(FILTER_APPLIED);
			if (debug) {
				logger.debug("SecurityContextHolder now cleared, as request processing completed");
			}
		}
```

实际的Authentication是只有一个继承实现，可主要的方法都是返回的Oject的值，意味着可以自定义一些验证，最主要的验证类还token的验证。

![](/AbstractAuthenticationToken.png)



#### FliterChain的链式调用

​	实际属于责任链模式，一层一层的调用，FilterChainProxy使用了嵌套类（可以实现内部的多重继承，以及对外部属性的所有访问权限）





