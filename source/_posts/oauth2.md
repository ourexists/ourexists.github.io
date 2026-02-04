---
title: Springboot3生态下OAuth2的生态搭建
tags:
  - Springboot3
  - OAuth2
categories:
  - [ 后端技术, Spring生态 ]
top_img: /img/top.png
cover: /img/springboot.png
date: 2025-08-25 00:00:00
---

### 一、环境生态

Java17+, springcloud 2024.0.1, spring-boot3.4.6, OAuth2 6.4.x

### 二、公共基础依赖

```
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.6</version>
    </parent>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2024.0.1</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

### 三、认证服务器搭建

##### 1. POM依赖

```
 <dependencies>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-authorization-server</artifactId>
        </dependency>

         <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
            <version>3.5.12</version>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-jsqlparser</artifactId>
            <version>3.5.12</version>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>8.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk18on</artifactId>
            <version>1.78.1</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcpkix-jdk18on</artifactId>
            <version>1.78.1</version>
        </dependency>
    </dependencies>
```

#### 2. 代码配置

```
@Configuration
@Order(Integer.MIN_VALUE)
public class AuthorizationServerConfigurer {

    @Value("${cus.token.jwt-privateKey}")
    private String privateKey;


    @Value("${cus.token.jwt-publicKey}")
    private String publicKey;

    @Value("${cus.token.expire:2592000}")
    private Integer tokenValiditySeconds;

    @Bean
    public RegisteredClientRepository registeredClientRepository(JdbcOperations jdbcOperations) {
        return new JdbcRegisteredClientRepository(jdbcOperations);
    }

    @Bean
    public OAuth2AuthorizationService dbAuthorizationService(JdbcOperations jdbcOperations,
                                                             RegisteredClientRepository registeredClientRepository) {
        return new JdbcOAuth2AuthorizationService(jdbcOperations, registeredClientRepository);
    }


    @Bean
    public OAuth2TokenCustomizer<JwtEncodingContext> jwtCustomizer() {
        return context -> {
            if (OAuth2TokenType.ACCESS_TOKEN.equals(context.getTokenType())) {
                Instant now = Instant.now();
                // token附加内容,也可以加其他字段
                context.getClaims().claim("copyright", "1222"); 
                context.getClaims().issuedAt(now);
                context.getClaims().expiresAt(now.plusSeconds(tokenValiditySeconds));
            }
        };
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        //自定义你的密码加密方式
        return new BCryptPasswordEncoder();
    }

    /**
     * jwt配置
     */
    @Bean
    public JWKSource<SecurityContext> jwkSource() throws JOSEException {
        JWK jwk = RSAKey.parseFromPEMEncodedObjects(this.privateKey);
        JWKSet jwkSet = new JWKSet(jwk);
        return (jwkSelector, context) -> jwkSelector.select(jwkSet);
    }

    /**
     * jwt编码器
     */
    @Bean
    public JwtEncoder jwtEncoder(JWKSource<SecurityContext> jwkSource) {
        return new NimbusJwtEncoder(jwkSource);
    }

    /**
     * jwt解码器
     */
    @Bean
    public JwtDecoder jwtDecoder() throws NoSuchAlgorithmException, InvalidKeySpecException {
        byte[] encoded = Base64.getDecoder().decode(this.publicKey);
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(encoded);
        return NimbusJwtDecoder.withPublicKey((RSAPublicKey) keyFactory.generatePublic(keySpec)).build();
    }

    @Bean
    @Order(1)
    public SecurityFilterChain authServerSecurityFilterChain(HttpSecurity http,
                                                             OAuth2AuthorizationServerConfigurer configurer,
                                                             EraAuthenticationEntryPoint eraAuthenticationEntryPoint,
                                                             EraAccessDeniedHandler eraAccessDeniedHandler) throws Exception {
        return http
                .cors(CorsConfigurer::disable)
                .csrf(CsrfConfigurer::disable)
                .securityMatcher(PathRule.OAUTH_PATHS)
                .authorizeHttpRequests(authorize -> authorize
                        .anyRequest().authenticated() // 所有其他接口需要认证
                )
                .exceptionHandling(exception -> exception
                        .authenticationEntryPoint(eraAuthenticationEntryPoint)
                        .accessDeniedHandler(eraAccessDeniedHandler)
                )
                .with(configurer, Customizer.withDefaults())
                .oauth2ResourceServer(resourceServer -> resourceServer.jwt(Customizer.withDefaults()))
                .build();
    }


    //同时作为资源服务器可以加上配置
    @Bean
    @Order(2)
    public SecurityFilterChain resourceSecurityFilterChain(HttpSecurity http,
                                                           EraAuthenticationEntryPoint eraAuthenticationEntryPoint,
                                                           EraAccessDeniedHandler eraAccessDeniedHandler) throws Exception {
        return http
                .securityMatchers(requestMatcherConfigurer -> {
                            List<RequestMatcher> r = new ArrayList<>();
                            for (String oauthPath : PathRule.OAUTH_PATHS) {
                                r.add(new AntPathRequestMatcher(oauthPath));
                            }
                            //这里Negated的使用方式需要注意
                            requestMatcherConfigurer.requestMatchers(new NegatedRequestMatcher(new OrRequestMatcher(r)));
                        }
                )
                .cors(CorsConfigurer::disable)
                .csrf(CsrfConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize
                        .anyRequest().authenticated() // 所有其他接口需要认证
                )
                .exceptionHandling(exception -> exception
                        .authenticationEntryPoint(eraAuthenticationEntryPoint)
                        .accessDeniedHandler(eraAccessDeniedHandler)
                )
                .oauth2ResourceServer(resourceServer -> resourceServer.jwt(Customizer.withDefaults()))
                .build();
    }

    //配置自定义的认证类型.这里自定义包装了一层,用于防止多次载入框架中原有的认证模式
    @Bean
    @ConditionalOnMissingBean(OAuth2AuthorizationServerConfigurer.class)
    public OAuth2AuthorizationServerConfigurer oAuth2AuthorizationServerConfigurer(List<EraAuthenticationConverter> eraAuthenticationConverters,
                                                                                   List<EraAuthenticationProvider> eraAuthenticationProviders) {
        OAuth2AuthorizationServerConfigurer configurer = OAuth2AuthorizationServerConfigurer.authorizationServer();
        configurer.tokenEndpoint(tokenEndpoint -> {
            if (CollectionUtil.isNotBlank(eraAuthenticationConverters)) {
                tokenEndpoint
                        .accessTokenRequestConverters(converters -> {
                            converters.addAll(eraAuthenticationConverters);
                        });
            }
            if (CollectionUtil.isNotBlank(eraAuthenticationProviders)) {
                tokenEndpoint
                        .authenticationProviders(providers -> {
                            providers.addAll(eraAuthenticationProviders);
                        });
            }
        });
        return configurer;
    }

}
```

```
public interface EraAuthenticationConverter extends AuthenticationConverter {
}
```

```
public interface EraAuthenticationProvider extends AuthenticationProvider {
}
```

```
public class PathRule {
    public static final String[] OAUTH_PATHS = {"/oauth/**", "/.well-known/**", "/connect/register","/oauth2/**"};
}
```

#### 3. 请求接口案例

###### client_credentials 模式（机器到机器）

```
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)

grant_type=client_credentials
&scope=read
```

#### 4. 自定义授权参考（短信认证模式）

```
@Component
public class SmsVerifyAuthenticationConverter implements EraAuthenticationConverter {

    private static final String GRANT_TYPE = "sms_verify";

    private static final String PARAMETER_MOBILE = "mobile";

    private static final String PARAMETER_SMSCODE = "sms_code";


    @Override
    public Authentication convert(HttpServletRequest request) {
        String grantType = request.getParameter(OAuth2ParameterNames.GRANT_TYPE);
        if (!GRANT_TYPE.equals(grantType)) {
            return null;
        }
        String phone = request.getParameter(PARAMETER_MOBILE);
        String smscode = request.getParameter(PARAMETER_SMSCODE);
        return new SmsVerifyAuthenticationToken(phone, smscode);
    }
}
```

```
@Component
public class SmsVerifyAuthenticationProvider implements EraAuthenticationProvider {

    private final SmsVerifyDetailsService smsUserDetailsService;

    public SmsVerifyAuthenticationProvider(SmsVerifyDetailsService smsUserDetailsService) {
        this.smsUserDetailsService = smsUserDetailsService;
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        SmsVerifyAuthenticationToken smsCodeAuthenticationToken = (SmsVerifyAuthenticationToken) authentication;
        String mobile = (String) smsCodeAuthenticationToken.getPrincipal();
        String smscode = (String) smsCodeAuthenticationToken.getCredentials();
        UserDetails userDetails = smsUserDetailsService.loadUserByMobile(mobile, smscode);
        if (!userDetails.isEnabled()) {
            throw new UsernameNotFoundException("该手机号未注册.");
        }

        if (!userDetails.isCredentialsNonExpired()) {
            throw new CredentialsExpiredException("当前手机号账户已过期！");
        }
        if (!userDetails.isAccountNonExpired()) {
            throw new AccountExpiredException("账户过期！");
        }
        if (!userDetails.isAccountNonLocked()) {
            throw new LockedException("账户被冻结！");
        }

        EraAuthenticationToken eraAuthenticationToken = new EraAuthenticationToken(userDetails, userDetails.getAuthorities());
        eraAuthenticationToken.setDetails(authentication.getDetails());
        return eraAuthenticationToken;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return SmsVerifyAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

```
public class SmsVerifyAuthenticationToken extends AbstractAuthenticationToken {

    private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;


    public SmsVerifyAuthenticationToken(Object principal) {
        super(principal);
    }

    public SmsVerifyAuthenticationToken(Object principal, Object credentials) {
        super(principal, credentials);
    }

    public SmsVerifyAuthenticationToken(Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities) {
        super(principal, credentials, authorities);
    }
}
```

```
/**
 *  具体方式自己实现
 */
public interface SmsVerifyDetailsService {

    /**
     * 通过手机号查询用户
     * @param mobile    手机号
     * @param smsCode   验证码
     * @return
     * @throws UsernameNotFoundException
     */
    UserDetails loadUserByMobile(String mobile, String smsCode) throws UsernameNotFoundException;
}
```

#### 注意

* 新版的OAuth2认证自定义的认证模式无需再额外传入`client_id`和`client_secret`

### 四、资源服务器配置

##### 1. POM依赖

```
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
    </dependencies>
```

##### 2. 代码配置

```
@Configuration
@EnableWebSecurity
public class ResourceServerConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(PermissionWhiteListProperties permissionWhiteListProperties,
                                                   HttpSecurity http) throws Exception {
        //白名单。自己定义即可
        List<String> whites = new ArrayList<>();
        http
                .cors(CorsConfigurer::disable)
                .csrf(CsrfConfigurer::disable)
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers(whites.toArray(new String[whites.size()])).permitAll()
                        .anyRequest()
                        .authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2
                        .jwt(Customizer.withDefaults()) // 启用 JWT 解码
                );
        return http.build();
    }

    //认证响应体处理。自定义即可，可以不用
    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint() {
        return new AuthenticationEntryPoint() {
            ...........
        };
    }

    //认证异常响应处理。自定义即可，可以不用
    @Bean
    public AccessDeniedHandler accessDeniedHandler() {
        return new AccessDeniedHandler() {
            ...........
        };
    }
}
```

##### 3. 配置文件

```
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: http://<资源服务器地址>/oauth2/jwks
```

### 五、客户端配置

##### 1. POM依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

##### 2. 配置文件

```
spring:
  security:
    oauth2:
      client:
        registration:
          <自定义客户端名称>:
            client-id: ylcloud
            client-secret: ylcloudxxdd
            authorization-grant-type: client_credentials
            provider: satellite-gateway
        provider:
          <自定义客户端名称,与上面对应>:
            token-uri: http://<认证服务器地址>/oauth2/token
```

##### 3. 连接器配置

```
@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient(ClientRegistrationRepository clientRepo,
                               OAuth2AuthorizedClientRepository authorizedClientRepo) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2 =
                new ServletOAuth2AuthorizedClientExchangeFilterFunction(clientRepo, authorizedClientRepo);
        oauth2.setDefaultClientRegistrationId("<配置文件对应的客户端名称>");

        return WebClient.builder()
                .apply(oauth2.oauth2Configuration())
                .build();
    }
}
```
