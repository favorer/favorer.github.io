## spring security core
### 核心概念
* Principal: 认证主体
* Authentication：认证信息。集成Principal。
* GrantedAuthority 授权信息
* SecurityContext：SecurityContextHolder持有对应上下文信息。对应全局或线程级 SecurityContextHolder.getContext().setAuthentication(anAuthentication);
* Token,TokenService
* UserDetails, AuthenticationUserDetailsService, UserDetailsService
* SessionRegistry

### 认证过程相关类
* AuthenticationManager
* AuthenticationProvider
* SecurityMetadataSource
* ConfigAttribute, PreInvocationAttribute, PostInvocationAttributePreInvocationAttribute
* AuthenticationTrustResolver
* AuthenticationEventPublisher

TODO详细待展开
### 用户权限等维护相关类
* GroupManager
* UserDetailsManager
* MutableUserDetails

### 访问控制相关类
* AccessDecisionManager 检查授权信息
  - AffirmativeBased 至少一个投票者通过
  - ConsensusBased 多数投票者通过
  - UnanimousBased 没有投出过拒绝票
* AccessDecisionVoter 具体针对每类权限的判断vote(Authentication authentication, S object,Collection<ConfigAttribute> attributes)
```
AccessDecisionVoter子类:
RoleVoter (org.springframework.security.access.vote)
    RoleHierarchyVoter (org.springframework.security.access.vote)
ScopeVoter (org.springframework.security.oauth2.provider.vote)
WebExpressionVoter (org.springframework.security.web.access.expression)
ClientScopeVoter (org.springframework.security.oauth2.provider.vote)
Jsr250Voter (org.springframework.security.access.annotation)
AuthenticatedVoter (org.springframework.security.access.vote)
AbstractAclVoter (org.springframework.security.access.vote)
PreInvocationAuthorizationAdviceVoter (org.springframework.security.access.prepost)

```
* SecurityMetadataSource 包含类似role权限信息。可以获取对象关联的权限角色
* ConfigAttribute 用字符串表示具体权限角色类型。
* PermissionEvaluator 可以用于类似ACL的细粒度的检查

## spring security web
### 主要概念
* SecurityFilterChain 包含针对一组请求包含的过滤器。
* FilterChainProxy 作为security web的filter入口。包含一组SecurityFilterChain。针对请求选择对应的一组过滤器SecurityFilterChain进行拦截调用。都是安全相关的拦截。
  比如获取token时，token的过滤器包含加载认证信息的过滤器。资源访问请求的过滤器则不包含，但是包含检查token的过滤器。
* AuthenticationEntryPoint 对某种认证模式的失败进行处理，针对响应设置对应的header等。比如针对Basic返回提示认证的信息和表单模式提交重定向到登陆首页。

```
AuthenticationEntryPoint子类:
Http401AuthenticationEntryPoint (org.springframework.boot.autoconfigure.security)
DelegatingAuthenticationEntryPoint (org.springframework.security.web.authentication)
BasicAuthenticationEntryPoint (org.springframework.security.web.authentication.www)
DigestAuthenticationEntryPoint (org.springframework.security.web.authentication.www)
Http403ForbiddenEntryPoint (org.springframework.security.web.authentication)
LoginUrlAuthenticationEntryPoint (org.springframework.security.web.authentication)
OAuth2AuthenticationEntryPoint (org.springframework.security.oauth2.provider.error)
HttpStatusEntryPoint (org.springframework.security.web.authentication)

```

* RedirectStrategy
* PortResolver
### 认证主要类
* AuthenticationSuccessHandler，AuthenticationFailureHandler
* RememberMeServices
* Basic相关
  - BasicAuthenticationFilter
  - BasicAuthenticationEntryPoint
* Digest相关
  - DigestAuthenticationFilter
  - DigestAuthenticationEntryPoint
### 访问控制相关类
* FilterSecurityInterceptor 通过实现Filter，在调用前调用AccessDecisionManager#decide判断是否允许访问

## spring security oauth2
### 核心类
* TokenGranter 按照不同方式对token授权。包含五种token生成方式。AuthorizationCode, Implicit, client_credentials, refreshToken, password
* ClientDetails 认证的客户信息。包含扩展的Map字段additionalInformation
* ClientDetailsService 加载客户信息
* ClientRegistrationService 维护客户信息。包含增删改等操作。
* OAuth2RequestFactory

### token相关
* TokenStore token存储。包括内存，redis，jdbc，jwt，jwk等。
* TokenEnhancer token增强。可以扩展token附加信息。比如租户id。
* OAuth2AccessToken accessToken信息
* AuthorizationServerTokenServices 按照认证信息获取accessToken
  大体是TokenGranter调用AuthorizationServerTokenServices,AuthorizationServerTokenServices调用TokenStore
* ResourceServerTokenServices 资源服务器访问时，通过accessToken加载认证信息.
* AccessTokenConverter 类似token的序列化和反序列化

### 端点类
* TokenEndpoint 获取token端点
* CheckTokenEndpoint 检查token端点



### 消息执行过程
* 请求过滤器链。web原始的过滤器链为tomcat的ApplicationFilterChain。里面的springSecurityFilterChain作为web包的DelegatingFilterProxy会代理调用security包的FilterChainProxy。构建VirtualFilterChain调用additionalFilters中的filter。然后继续未完成的原始filter链ApplicationFilterChain。
```
chain = {FilterChainProxy$VirtualFilterChain@10122}
 originalChain = {ApplicationFilterChain@10132}
  filters = {ApplicationFilterConfig[10]@10359}
   0 = {ApplicationFilterConfig@10362} 指标统计 "ApplicationFilterConfig[name=metricsFilter, filterClass=org.springframework.boot.actuate.autoconfigure.MetricsFilter]"
   1 = {ApplicationFilterConfig@10363} 设置编码 "ApplicationFilterConfig[name=characterEncodingFilter, filterClass=org.springframework.boot.web.filter.OrderedCharacterEncodingFilter]"
   2 = {ApplicationFilterConfig@10364} sleuth消息跟踪 "ApplicationFilterConfig[name=traceFilter, filterClass=org.springframework.cloud.sleuth.instrument.web.TraceFilter]"
   3 = {ApplicationFilterConfig@10365} method转换 "ApplicationFilterConfig[name=hiddenHttpMethodFilter, filterClass=org.springframework.boot.web.filter.OrderedHiddenHttpMethodFilter]"
   4 = {ApplicationFilterConfig@10366} 支持http的put和patch获取form的参数 "ApplicationFilterConfig[name=httpPutFormContentFilter, filterClass=org.springframework.boot.web.filter.OrderedHttpPutFormContentFilter]"
   5 = {ApplicationFilterConfig@10367} 上下文设置 "ApplicationFilterConfig[name=requestContextFilter, filterClass=org.springframework.boot.web.filter.OrderedRequestContextFilter]"
   6 = {ApplicationFilterConfig@10368} 代理调用springSecurityFilterChain TODO "ApplicationFilterConfig[name=springSecurityFilterChain, filterClass=org.springframework.boot.web.servlet.DelegatingFilterProxyRegistrationBean$1]"
   7 = {ApplicationFilterConfig@10369} 消息调用记录，类似接口日志 "ApplicationFilterConfig[name=webRequestLoggingFilter, filterClass=org.springframework.boot.actuate.trace.WebRequestTraceFilter]"
   8 = {ApplicationFilterConfig@10370} 响应头加入header：X-Application-Context "ApplicationFilterConfig[name=applicationContextIdFilter, filterClass=org.springframework.boot.web.filter.ApplicationContextHeaderFilter]"
   9 = {ApplicationFilterConfig@10371} WebSocket支持 "ApplicationFilterConfig[name=Tomcat WebSocket (JSR356) Filter, filterClass=org.apache.tomcat.websocket.server.WsFilter]"
  pos = 7
  n = 10
  servlet = {DispatcherServlet@10361}
  servletSupportsAsync = true
 additionalFilters = {ArrayList@10344}  size = 11
  0 = WebAsyncManager加入SecurityContext上下文拦截处理 {WebAsyncManagerIntegrationFilter@10127}
  1 = SecurityContext获取和持久化，比如session中。{SecurityContextPersistenceFilter@10125}
  2 = 支持向response写入header {HeaderWriterFilter@10124}
  3 = 支持登出操作 {LogoutFilter@10123}
  4 = 有token则认证 {OAuth2AuthenticationProcessingFilter@10118}
  5 = 获取认证跳转前缓存的请求{RequestCacheAwareFilter@10353}
  6 = 请求对象中包装认证对象从spring security获取而不是web容器{SecurityContextHolderAwareRequestFilter@10354}
  7 = 没认证时，设置上下文为匿名用户对象{AnonymousAuthenticationFilter@10355}
  8 = 用户关联session控制 {SessionManagementFilter@10356}
  9 = filter异常处理。前面filter的异常，此时处理不了，比如认证过程 {ExceptionTranslationFilter@10357}
  10 = 安全拦截器TODO {FilterSecurityInterceptor@10358}
 firewalledRequest = {RequestWrapper@10179} "FirewalledRequest[ org.apache.catalina.connector.RequestFacade@5a96a1]"
 size = 11
 currentPosition = 5
debug = true
```

### spring boot配置
* AuthorizationServerEndpointsConfiguration 加载自定义的AuthorizationServerConfigurer来设置共享的一个AuthorizationServerEndpointsConfigurer, 调用自定义的AuthorizationServerConfigurer的configure(AuthorizationServerEndpointsConfigurer endpoints)
* AuthorizationServerSecurityConfiguration加载自定义的AuthorizationServerConfigurer来设置spring 容器中的ClientDetailsServiceConfigurer。调用自定义的AuthorizationServerConfigurer的configure(ClientDetailsServiceConfigurer clients)
* WebSecurityConfiguration加载所有SecurityConfigurer配置,并配置，但未实例化构建。WebSecurityConfiguration加载springSecurityFilterChain的Bean时，构建Filter对象。此时调用前面的SecurityConfigurer列表的init，调用configure(HttpSecurity http).
   - 构建过程会创建AuthorizationServerSecurityConfigurer，
   - AuthorizationServerSecurityConfiguration作为一个SecurityConfigurer, 会调用AuthorizationServerConfigurer的configure(AuthorizationServerSecurityConfigurer oauthServer)


WebSecurityConfiguration 加载安全配置
具体springSecurityFilterChain()会将所有SecurityConfigurer 加载到WebSecurity中，进行构建
```
SecurityConfigurer子类
SecurityConfigurerAdapter (org.springframework.security.config.annotation)
    ClientDetailsServiceConfigurer (org.springframework.security.oauth2.config.annotation.configurers)
    OAuth2ClientAuthenticationConfigurer in SsoSecurityConfigurer (org.springframework.boot.autoconfigure.security.oauth2.client)
    UserDetailsAwareConfigurer (org.springframework.security.config.annotation.authentication.configurers.userdetails)
        AbstractDaoAuthenticationConfigurer (org.springframework.security.config.annotation.authentication.configurers.userdetails)
            DaoAuthenticationConfigurer (org.springframework.security.config.annotation.authentication.configurers.userdetails)
            UserDetailsServiceConfigurer (org.springframework.security.config.annotation.authentication.configurers.userdetails)
                UserDetailsManagerConfigurer (org.springframework.security.config.annotation.authentication.configurers.provisioning)
                    JdbcUserDetailsManagerConfigurer (org.springframework.security.config.annotation.authentication.configurers.provisioning)
                    InMemoryUserDetailsManagerConfigurer (org.springframework.security.config.annotation.authentication.configurers.provisioning)
                        DefaultInMemoryUserDetailsManagerConfigurer in AuthenticationManagerConfiguration (org.springframework.boot.autoconfigure.security)
    ResourceServerSecurityConfigurer (org.springframework.security.oauth2.config.annotation.web.configurers)
    AbstractHttpConfigurer (org.springframework.security.config.annotation.web.configurers)
        HttpBasicConfigurer (org.springframework.security.config.annotation.web.configurers)
        LogoutConfigurer (org.springframework.security.config.annotation.web.configurers)
        RememberMeConfigurer (org.springframework.security.config.annotation.web.configurers)
        RequestCacheConfigurer (org.springframework.security.config.annotation.web.configurers)
        ServletApiConfigurer (org.springframework.security.config.annotation.web.configurers)
        DefaultLoginPageConfigurer (org.springframework.security.config.annotation.web.configurers)
        SessionManagementConfigurer (org.springframework.security.config.annotation.web.configurers)
        PortMapperConfigurer (org.springframework.security.config.annotation.web.configurers)
        ExceptionHandlingConfigurer (org.springframework.security.config.annotation.web.configurers)
        HeadersConfigurer (org.springframework.security.config.annotation.web.configurers)
        CsrfConfigurer (org.springframework.security.config.annotation.web.configurers)
        JeeConfigurer (org.springframework.security.config.annotation.web.configurers)
        AnonymousConfigurer (org.springframework.security.config.annotation.web.configurers)
        ChannelSecurityConfigurer (org.springframework.security.config.annotation.web.configurers)
        CorsConfigurer (org.springframework.security.config.annotation.web.configurers)
        SecurityContextConfigurer (org.springframework.security.config.annotation.web.configurers)
        X509Configurer (org.springframework.security.config.annotation.web.configurers)
        AbstractAuthenticationFilterConfigurer (org.springframework.security.config.annotation.web.configurers)
            FormLoginConfigurer (org.springframework.security.config.annotation.web.configurers)
            OpenIDLoginConfigurer (org.springframework.security.config.annotation.web.configurers.openid)
        AbstractInterceptUrlConfigurer (org.springframework.security.config.annotation.web.configurers)
            UrlAuthorizationConfigurer (org.springframework.security.config.annotation.web.configurers)
            ExpressionUrlAuthorizationConfigurer (org.springframework.security.config.annotation.web.configurers)
    AuthorizationServerSecurityConfigurer (org.springframework.security.oauth2.config.annotation.web.configurers)
    ClientDetailsServiceBuilder (org.springframework.security.oauth2.config.annotation.builders)
        JdbcClientDetailsServiceBuilder (org.springframework.security.oauth2.config.annotation.builders)
        1 in ClientDetailsServiceBuilder (org.springframework.security.oauth2.config.annotation.builders)
        InMemoryClientDetailsServiceBuilder (org.springframework.security.oauth2.config.annotation.builders)
    LdapAuthenticationProviderConfigurer (org.springframework.security.config.annotation.authentication.configurers.ldap)
WebSecurityConfigurer (org.springframework.security.config.annotation.web)
    WebSecurityConfigurerAdapter (org.springframework.security.config.annotation.web.configuration)
        1 in WebSecurityConfiguration (org.springframework.security.config.annotation.web.configuration)
        ResourceServerConfiguration (org.springframework.security.oauth2.config.annotation.web.configuration)
        ApplicationNoWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)
        ManagementWebSecurityConfigurerAdapter in ManagementWebSecurityAutoConfiguration (org.springframework.boot.actuate.autoconfigure)
        AuthorizationServerSecurityConfiguration (org.springframework.security.oauth2.config.annotation.web.configuration)
        H2ConsoleSecurityConfigurer in H2ConsoleSecurityConfiguration in H2ConsoleAutoConfiguration (org.springframework.boot.autoconfigure.h2)
        OAuth2SsoDefaultConfiguration (org.springframework.boot.autoconfigure.security.oauth2.client)
        ApplicationWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)
    IgnoredPathsWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)
GlobalAuthenticationConfigurerAdapter (org.springframework.security.config.annotation.authentication.configurers)
    InitializeAuthenticationProviderBeanManagerConfigurer (org.springframework.security.config.annotation.authentication.configuration)
    InitializeUserDetailsBeanManagerConfigurer (org.springframework.security.config.annotation.authentication.configuration)
    InitializeUserDetailsManagerConfigurer in InitializeAuthenticationProviderBeanManagerConfigurer (org.springframework.security.config.annotation.authentication.configuration)
    SpringBootAuthenticationConfigurerAdapter in AuthenticationManagerConfiguration (org.springframework.boot.autoconfigure.security)
    BootGlobalAuthenticationConfigurationAdapter in BootGlobalAuthenticationConfiguration (org.springframework.boot.autoconfigure.security)
    InitializeUserDetailsManagerConfigurer in InitializeUserDetailsBeanManagerConfigurer (org.springframework.security.config.annotation.authentication.configuration)
    EnableGlobalAuthenticationAutowiredConfigurer in AuthenticationConfiguration (org.springframework.security.config.annotation.authentication.configuration)

```
```
WebSecurityConfigurer子类
WebSecurityConfigurerAdapter (org.springframework.security.config.annotation.web.configuration)
    WebSecurityConfiguration (com.huawei.billingcloud.sysmgmt.oauth)
    1 in WebSecurityConfiguration (org.springframework.security.config.annotation.web.configuration)
    ResourceServerConfiguration (org.springframework.security.oauth2.config.annotation.web.configuration)
    ApplicationNoWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)
    ManagementWebSecurityConfigurerAdapter in ManagementWebSecurityAutoConfiguration (org.springframework.boot.actuate.autoconfigure)
    AuthorizationServerSecurityConfiguration (org.springframework.security.oauth2.config.annotation.web.configuration)
    H2ConsoleSecurityConfigurer in H2ConsoleSecurityConfiguration in H2ConsoleAutoConfiguration (org.springframework.boot.autoconfigure.h2)
    OAuth2SsoDefaultConfiguration (org.springframework.boot.autoconfigure.security.oauth2.client)
    ApplicationWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)
IgnoredPathsWebSecurityConfigurerAdapter in SpringBootWebSecurityConfiguration (org.springframework.boot.autoconfigure.security)

```

ResourceServerConfiguration 加载资源服务器配置ResourceServerConfigurer。
同时自身作为一个WebSecurityConfigurer被上面的WebSecurityConfiguration加载


```
RestTemplate 默认converter
0 = {ByteArrayHttpMessageConverter@8484}
1 = {StringHttpMessageConverter@8485}
2 = {ResourceHttpMessageConverter@8486}
3 = {SourceHttpMessageConverter@8487}
4 = {AllEncompassingFormHttpMessageConverter@8488}
5 = {Jaxb2RootElementHttpMessageConverter@8489}
6 = {MappingJackson2HttpMessageConverter@8490}
```


```
0 = {SpringBootWebSecurityConfiguration$IgnoredPathsWebSecurityConfigurerAdapter@11234}
1 = {ResourceServerConfiguration$$EnhancerBySpringCGLIB$$c6c322ec@8468}
2 = {SpringBootWebSecurityConfiguration$ApplicationNoWebSecurityConfigurerAdapter$$EnhancerBySpringCGLIB$$a64c52f7@11230}
```

### 启动配置

```
0 = {SpringBootWebSecurityConfiguration$IgnoredPathsWebSecurityConfigurerAdapter@13290}
1 = {AuthorizationServerSecurityConfiguration$$EnhancerBySpringCGLIB$$2aaaf2bf@9227}
2 = {WebSecurityConfiguration$$EnhancerBySpringCGLIB$$f14e4087@13291}
3 = {SpringBootWebSecurityConfiguration$ApplicationNoWebSecurityConfigurerAdapter$$EnhancerBySpringCGLIB$$a7a04c53@13292}
```
