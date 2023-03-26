# 权限管理

不同用户登陆后台管理系统拥有不同的彩蛋权限和功能权限

权限管理包含三个功能模块：菜单管理，角色管理，用户管理

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206021518699.png" alt="07 权限管理需求" style="zoom:67%;" />

1、创建权限管理模块service-acl

2、引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.yienx</groupId>
        <artifactId>spring_security</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
    </dependency>
</dependencies>
```



## SpringSecurity

- **用户认证**(**Authentication**)

	验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

- **用户授权**(**Authorization**)

	验证某个用户是否有权限执行某个操作。

> **Spring Security**其实就是用**filter**，多请求的路径进行过滤。

1. 如果是基于Session，那么Spring-security会对cookie里的sessionid进行解析，找到服务器存储的sesion信息，然后判断当前用户是否符合请求的要求。

2. 如果是token，则是解析出token，然后将当前请求加入到Spring-security管理的权限信息中去

[参考](https://blog.csdn.net/lendsomething/article/details/119251123)

### 实现原理

如果系统的模块众多，每个模块都需要就行授权与认证，所以我们选择基于 token 的形式进行授权与认证。

1、用户根据用户名密码认证成功，然后从数据库获取当前用户角色的一系列**权限值**，

2、并以用户名为 key，权限列表为 value 的形式存入 redis 缓存中。

3、根据用户名相关信息生成 token 返回，

4、**浏览器将 token 记录到 cookie 中， 每次调用api接口都默认将 token 携带到 header 请求头中**。

5、Spring-security 从 header 中解析获取 token 信息，解析 token 获取当前用户名，根据用户名就可以从 **redis** 中获取权限列表，

6、这样 Spring-security 就能够判断当前请求是否有权限访问，给用户赋予权限

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022017607.png" alt="01 Spring Security授权过程" style="zoom:75%;" />

> 用户权限列表存入redis是因为token存不了那么多东西

### 实现

```xml
<dependencies>
    <dependency>
        <groupId>com.atguigu</groupId>
        <artifactId>common_utils</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
	<!-- Spring Security依赖 --> 
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
    </dependency>
</dependencies>
```

代码结构

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022014018.png" alt="image-20220602201435579" style="zoom:50%;" />

Spring Security 的核心配置就是继承 WebSecurityConfigurerAdapter 并注解 @EnableWebSecurity 的配置。

这个配置指明了用户名密码的处理方式、请求路径的开合、登录登出控制等和安全相关的配置

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202206022118689.png" alt="02 Spring Security代码执行过程" style="zoom:75%;" />

# 重要接口和过滤器

## UsernamepasswordAuthenticationFilter

拦截登录请求并获取用户输入的账号和密码，然后把账号密码封装到 `UsernamePasswordAuthenticationToken`（未受信任的认证凭据）中，然后将 “认证凭据" 交给 **我们配置** 的`AuthenticationManager`去作认证

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207271423656.png" alt="image-20220727142255500" style="zoom:67%;" />

可以实现：

- 定制我们的登录请求URI和请求方式。
- 登录请求参数的格式定制化。
- 拦截登录。
- 登录成功后调用的方法。
- 登录失败后调用的方法。

详见：https://felord.cn/usernamePasswordAuthenticationFilter.html

## UserDetailsService

查询数据库用户名和密码的过程。

* 创建类实现UserDetailService，编写查询数据库过程，返回UserDetail对象，这个对象是安全框架提供的。

## PasswordEncoder

密码编码过程

## AuthenticationManager

认证管理器。`WebSecurityConfigurerAdapter`（配置SpringSecurity的类）中的`void configure(AuthenticationManagerBuilder auth)`是配置`AuthenticationManager` 的地方，里面只有一个`Authentication authenticate(Authentication authentication)`方法，他的作用是对 **未受信任的认证凭据** 做认证，**认证**成功则返回 授信状态的认证凭据 ，否则将抛出认证异常`AuthenticationException`

详见：https://www.jianshu.com/p/56e53d4ec1a9

## Authentication

认证对象，用于封装用户的信息。认证凭据 `UsernamePasswordAuthenticationToken`实现了该接口。属性如下：

`Boolean authenticated`：确定当前用户是否受信，为true时受信，为false时不受信。上述的返回 授信状态的认证凭据，就是将该值设置为true。

`Object principal`：主体。在未受信的情况是，这是用户名；在已受信的情况下，这是UserDetails的实现类。

`Object credentials`：在未受信的情况是，这是密码；在已受信的情况下，这是null，所以这里也可以设置JWT。

`Collection authorities`：主体的权限集合。由`AuthenticationManager`设置的，所以刚登录的时候，这里没有值；登录成功后主体中的权限会设置在这里。

`Object details`：存储关于身份验证请求的其他详细信息。这些可能是IP地址、证书编号等。未使用返回null。

详见：https://felord.cn/securityContext.html

## SecurityContextHolder

这是一个工具类，用于设置、获取、清理`SecurityContext`，这个`SecurityContext`是存储`Authentication`的容器，也就是说可以将**受信用户**存放到`SecurityContext`中，允许该请求的后续访问。

详见：https://felord.cn/spring-security-securitycontext.html

## BasicAuthenticationFilter

负责处理 HTTP 头中显示的基本身份验证凭据。

重写`void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)` 即可从 HTTP 头获取JWT

详见：https://felord.cn/spring-security-filters.html#3-24-BasicAuthenticationFilter

## FilterSecurityIntercepter过滤器

是最后一层过滤器，根据资源配置判断当前请求是否有权限访问对应的资源

# 基于数据库的记住我机制

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207271013318.png" alt="11-web权限方案-记住用户流程" style="zoom:67%;" />

# 流程

## 认证

UsernamePasswordAuthenticationFilter的父类Filter中的过滤方法doFilter中，调用子类方法attemptAuthentication进行身份认证

attemptAuthentication中判断是否post提交，然后获取表单数据，使用数据构造成对象，标记为未认证，然后调用authenticate方法进行认证，见pdf

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207272255413.png" alt="23-SpringSecurity原理（认证流程）" style="zoom:67%;" />

## 授权

**ExceptionTranslationFilter** **过滤器**

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207272259985.png" alt="image-20220727225905614" style="zoom:50%;" />

**FilterSecurityInterceptor** **过滤器**

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202207272259953.png" alt="image-20220727225955501" style="zoom:50%;" />

需要注意，**Spring Security** 的过滤器链是配置在 SpringMVC 的核心组件 DispatcherServlet 运行之前。也就是说，请求通过 **Spring Security** 的所有过滤器， 不意味着能够正常访问资源，该请求还需要通过 SpringMVC 的拦截器链。

## 请求见共享认证信息

一般认证成功后的用户信息是通过 Session 在多个请求之间共享，那么 **Spring Security** 中是如何实现将已认证的用户信息对象 Authentication 与 Session 绑定的进行 具体分析。

查看 SecurityContext 接口及其实现类 SecurityContextImpl，该类其实就是对 Authentication 的封装；

查看 SecurityContextHolder 类，该类其实是对 ThreadLocal 的封装，存储 SecurityContext 对象；

在 **UsernamePasswordAuthenticationFilter** 过滤器认证成功之后，会在认证成功的处理方法中将已认证的用户信息对象 Authentication 封装进 SecurityContext，并存入 SecurityContextHolder。

之后，响应会通过 **SecurityContextPersistenceFilter** 过滤器，该过滤器的位置在所有过滤器的最前面，请求到来先进它，响应返回最后一个通过它，所以在该过滤器中处理已认证的用户信息对象 Authentication 与 Session 绑定。

认证成功的响应通过 **SecurityContextPersistenceFilter** 过滤器时，会从 SecurityContextHolder 中取出封装了已认证用户信息对象 Authentication 的 SecurityContext，放进 Session 中。当请求再次到来时，请求首先经过该过滤器，该过滤 器会判断当前请求的 Session 是否存有 SecurityContext 对象，如果有则将该对象取出再次 放入 SecurityContextHolder 中，之后该请求所在的线程获得认证用户信息，后续的资源访问不需要进行身份认证；当响应再次返回时，该过滤器同样从 SecurityContextHolder 取出 SecurityContext 对象，放入 Session 中。

# 实现

## 核心配置类

Spring Security 的核心配置就是继承 WebSecurityConfigurerAdapter 并注解 @EnableWebSecurity 的配置。这个配置指明了用户名密码的处理方式、请求路径、登录登出控制等和安全相关的配置。

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class TokenWebSecurityConfig extends WebSecurityConfigurerAdapter {
	// 自定义查询数据库用户名密码和权限信息
    private UserDetailsService userDetailsService;
    // token 管理工具类(生成 token)
    private TokenManager tokenManager;
    // 密码管理工具类
    private DefaultPasswordEncoder defaultPasswordEncoder;
    // redis操作工具类
    private RedisTemplate redisTemplate;

    @Autowired
    public TokenWebSecurityConfig(UserDetailsService userDetailsService, DefaultPasswordEncoder defaultPasswordEncoder,
                                  TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.userDetailsService = userDetailsService;
        this.defaultPasswordEncoder = defaultPasswordEncoder;
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }

    /**
     * 配置设置
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling()
            // 无权限访问时，做的处理
            .authenticationEntryPoint(new UnauthorizedEntryPoint())
            .and().csrf().disable()
            .authorizeRequests()
            .anyRequest().authenticated()
            // 设置退出的地址
            .and().logout().logoutUrl("/admin/acl/index/logout")
            // 退出 
            .addLogoutHandler(new TokenLogoutHandler(tokenManager,redisTemplate)).and()
            // 认证
            .addFilter(new TokenLoginFilter(authenticationManager(), tokenManager, redisTemplate))
            // 授权
            .addFilter(new TokenAuthenticationFilter(authenticationManager(), tokenManager, redisTemplate)).httpBasic();
    }

    /**
     * 密码处理
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 这里需要去实现UserDetailsService类
        auth.userDetailsService(userDetailsService).passwordEncoder(defaultPasswordEncoder);
    }

    /**
     * 配置哪些请求不拦截
     * @param web
     * @throws Exception
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/api/**",
                "/swagger-resources/**", "/webjars/**", "/v2/**", "/swagger-ui.html/**"
               );
    }
}
```

## 密码处理类DefaultPasswordEncoder

```java
/**
 * <p>
 * 密码的处理方法类型
 * 有默认的，这里是改用md5加密，所以需要重写
 * </p>
 */
@Component
public class DefaultPasswordEncoder implements PasswordEncoder {

    public DefaultPasswordEncoder() {
        this(-1);
    }

    /**
     * @param strength
     *            the log rounds to use, between 4 and 31
     */
    public DefaultPasswordEncoder(int strength) {
		// 
    }
	
    public String encode(CharSequence rawPassword) {
        return MD5.encrypt(rawPassword.toString());
    }
	// 密码比对 
    // 表示验证从存储中获取的编码密码与编码后提交的原始密码是否匹配。
    // 如果密码匹 配，则返回 true;如果不匹配，则返回 false。
    // 第一个参数表示需要被解析的密码。第二个参数表示存储的密码。
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        return encodedPassword.equals(MD5.encrypt(rawPassword.toString()));
    }
}
```

## token操作的工具类TokenManager

```java
@Component
public class TokenManager {

    private long tokenExpiration = 24*60*60*1000;
    // 编码密钥
    private String tokenSignKey = "123456";
	// 根据用户名生成token
    public String createToken(String username) {
        String token = Jwts.builder().setSubject(username)
                .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))
                .signWith(SignatureAlgorithm.HS512, tokenSignKey).compressWith(CompressionCodecs.GZIP).compact();
        return token;
    }
	// 根据token的到用户名
    public String getUserFromToken(String token) {
        String user = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token).getBody().getSubject();
        return user;
    }

    public void removeToken(String token) {
        //jwttoken无需删除，客户端扔掉即可。
    }

}
```

## 退出实现TokenLogoutHandler

```java
public class TokenLogoutHandler implements LogoutHandler {

    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;

    public TokenLogoutHandler(TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        String token = request.getHeader("token");
        if (token != null) {
            // 只要前端不带token就可以了，这里写只是写个思想
            tokenManager.removeToken(token);
 
            //清空当前用户缓存中的权限数据
            String userName = tokenManager.getUserFromToken(token);
            redisTemplate.delete(userName);
        }
        ResponseUtil.out(response, R.ok());
    }

}
```

## 未授权统一处理UnauthorizedEntryPoint

```java
public class UnauthorizedEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {

        ResponseUtil.out(response, R.error());
    }
}
```

## 认证filter

```java
/**
 * <p>
 * 登录过滤器，继承UsernamePasswordAuthenticationFilter，
 * 对用户名密码进行登录校验
 * </p>
 */
public class TokenLoginFilter extends UsernamePasswordAuthenticationFilter {
	// SpringSecurity封装的
    private AuthenticationManager authenticationManager;
    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;

    public TokenLoginFilter(AuthenticationManager authenticationManager, TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.authenticationManager = authenticationManager;
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
        this.setPostOnly(false);
        // 登录路径和提交方式，会在配置类中设置
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/admin/acl/login","POST"));
    }
	
    // 获得提交过来的用户名和密码
    @Override
    public Authentication attemptAuthentication(HttpServletRequest req, HttpServletResponse res)
            throws AuthenticationException {
        try {
            // 使用请求体传递登陆参数，更安全
            User user = new ObjectMapper().readValue(req.getInputStream(), User.class);
            return authenticationManager
                .authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword()));
        	// 返回一个凭证认据
            // UsernamePasswordAuthenticationToken的构造器将其标记为未认证，其中参数不同标记不同认证程度
            // authenticationManager.authenticate将其标记为认证
            // authenticationManager有个实现类是ProviderManager，其负责进行认证 
            // authenticate方法根据认证类型进行认证；用户名密码还是邮箱密码等等
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 登录成功
     * @param req
     * @param res
     * @param chain
     * @param auth
     * @throws IOException
     * @throws ServletException
     */
    @Override
    protected void successfulAuthentication(HttpServletRequest req, HttpServletResponse res, FilterChain chain,
                                            Authentication auth) throws IOException, ServletException {
        // Principal未认证时是用户名，认证成功后是UserDeatils对象
        SecurityUser user = (SecurityUser) auth.getPrincipal();
        String token = tokenManager.createToken(user.getCurrentUserInfo().getUsername());
        redisTemplate.opsForValue().set(user.getCurrentUserInfo().getUsername(), user.getPermissionValueList());

        ResponseUtil.out(res, R.ok().data("token", token));
    }

    /**
     * 登录失败
     * @param request
     * @param response
     * @param e
     * @throws IOException
     * @throws ServletException
     */
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                                              AuthenticationException e) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}

```



## 授权filter

```java
/**
 * <p>
 * 访问过滤器
 * </p>
 */
public class TokenAuthenticationFilter extends BasicAuthenticationFilter {
    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;

    public TokenAuthenticationFilter(AuthenticationManager authManager, TokenManager tokenManager,RedisTemplate redisTemplate) {
        super(authManager);
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }

    // 对Http请求头做处理
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        logger.info("================="+req.getRequestURI());
        if(req.getRequestURI().indexOf("admin") == -1) {
            // 放行
            chain.doFilter(req, res);
            return;
        }

        UsernamePasswordAuthenticationToken authentication = null;
        try {
            // 授权
            authentication = getAuthentication(req);
        } catch (Exception e) {
            ResponseUtil.out(res, R.error());
        }
		
        if (authentication != null) {
            // 有授权，就放到权限上下文中
            // 是为了共享用的
            // SecurityContextHolder是对ThreadLocal的封装
            SecurityContextHolder.getContext().setAuthentication(authentication);
        } else {
            // 授权失败
            ResponseUtil.out(res, R.error());
        }
        chain.doFilter(req, res);
    }

    private UsernamePasswordAuthenticationToken getAuthentication(HttpServletRequest request) {
        // token置于header里
        String token = request.getHeader("token");
        if (token != null && !"".equals(token.trim())) {
            String userName = tokenManager.getUserFromToken(token);

            List<String> permissionValueList = (List<String>) redisTemplate.opsForValue().get(userName);
            Collection<GrantedAuthority> authorities = new ArrayList<>();
            for(String permissionValue : permissionValueList) {
                if(StringUtils.isEmpty(permissionValue)) continue;
                SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permissionValue);
                authorities.add(authority);
            }

            if (!StringUtils.isEmpty(userName)) {
                return new UsernamePasswordAuthenticationToken(userName, token, authorities);
            }
            return null;
        }
        return null;
    }
}
```

## 实体类

### User

```java
@Data
@ApiModel(description = "用户实体类")
public class User implements Serializable {

	private static final long serialVersionUID = 1L;

	@ApiModelProperty(value = "微信openid")
	private String username;

	@ApiModelProperty(value = "密码")
	private String password;

	@ApiModelProperty(value = "昵称")
	private String nickName;

	@ApiModelProperty(value = "用户头像")
	private String salt;

	@ApiModelProperty(value = "用户签名")
	private String token;

}
```



### SecurityUser

安全认证用户详情信息

```java
@Data
@Slf4j
public class SecurityUser implements UserDetails {

    //当前登录用户
    private transient User currentUserInfo;

    //当前权限
    private List<String> permissionValueList;

    public SecurityUser() {
    }

    public SecurityUser(User user) {
        if (user != null) {
            this.currentUserInfo = user;
        }
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        for(String permissionValue : permissionValueList) {
            if(StringUtils.isEmpty(permissionValue)) continue;
            SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permissionValue);
            authorities.add(authority);
        }

        return authorities;
    }

    @Override
    public String getPassword() {
        return currentUserInfo.getPassword();
    }

    @Override
    public String getUsername() {
        return currentUserInfo.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

## UserDetailsServiceImpl

```java
/**
 * <p>
 * 自定义userDetailsService - 认证用户详情
 * </p>
 *
 * @author qy
 * @since 2019-11-08
 */
@Service("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {
	
    // 查询User的service
    @Autowired
    private UserService userService;
	
    // 查询权限的service
    @Autowired
    private PermissionService permissionService;

    /***
     * 根据账号获取用户信息
     * @param username:
     * @return: org.springframework.security.core.userdetails.UserDetails
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 从数据库中取出用户信息
        User user = userService.selectByUsername(username);

        // 判断用户是否存在
        if (null == user){
            //throw new UsernameNotFoundException("用户名不存在！");
        }
        
        com.atguigu.serurity.entity.User curUser = new com.atguigu.serurity.entity.User();
        BeanUtils.copyProperties(user,curUser);
		// 这里是permissionService的多表连查
        List<String> authorities = permissionService.selectPermissionValueByUserId(user.getId());
        SecurityUser securityUser = new SecurityUser(curUser);
        securityUser.setPermissionValueList(authorities);
        return securityUser;
    }

}

```



