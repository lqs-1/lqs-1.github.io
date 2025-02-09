---
title: SpringSecurity+vue前后端分离
top: false
cover: false
toc: true
mathjax: true
date: 2023-03-17 16:09:59
password: 5c4aaf271c7c13b2372e84982f82f2cd279026950438dfc769c95dcabdfd2a87
summary: SpringSecurity+vue前后端分离
tags: spring-security
categories: spring
---
适合单体架构

## 基本搭建

创建SpringBoot项目并且导入SpringSecurity依赖

```pom
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.6.8</version>
</dependency>
```

导入图形验证码依赖

```pom
<dependency>
    <groupId>com.github.whvcse</groupId>
    <artifactId>easy-captcha</artifactId>
    <version>1.6.2</version>
</dependency>
```


### 创建配置类
```java
@Configuration
public class SecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();

        http.cors(Customizer.withDefaults());

        //解决跨域问题。cors 预检请求放行,让Spring security 放行所有preflight request（cors 预检请求）
        http.authorizeRequests().requestMatchers(CorsUtils::isPreFlightRequest).permitAll();

        http.formLogin()

                .failureHandler(new LoginFailHandler())

                .successHandler(new LoginSuccessHandler())

                .loginProcessingUrl("/user/login");

        http.authorizeRequests()

                .antMatchers("/user/captcha", "/user/register", "/user/alterPwd").permitAll()

                .anyRequest().authenticated();

        http.exceptionHandling() // 自定义权限受限403处理

                .accessDeniedHandler(new MyAccessDeniedHandler()); // 处理器模式，自由度更广

        // .accessDeniedPage("/forbidden.html"); // 直接返回一个页面。不和上面的同时使用
        
        http.logout().logoutUrl("/user/logout");
    }
    
}
```

### 实现登录逻辑
创建登录逻辑类并实现UserDetailService,重写他的loadUserByUsername方法
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService, UserDetailsService {

    @Autowired
    private UserRoleService userRoleService;
    @Autowired
    private RoleService roleService;
    @Autowired
    BCryptPasswordEncoder passwordEncoder;
    @Autowired
    private PermissionService permissionService;
    @Autowired
    private UserPermissionService userPermissionService;

    @Override
    public R getUser(User user, HttpServletRequest request) throws Exception {

        User querUser = this.baseMapper.selectByUserName(user.getUsername());

        if (user == null){

            return R.error(REnum.USER_DOES_NOT_EXIST.getStatusCode(),
                    REnum.USER_DOES_NOT_EXIST.getStatusMsg());

        } else if (!querUser.getPassword().equals(user.getPassword())) {

            return R.error(REnum.USER_PASSWORD_ERROR.getStatusCode(),
                    REnum.USER_PASSWORD_ERROR.getStatusMsg());

        }
        request.getSession().setAttribute("user", querUser);

        return R.ok(REnum.LOGIN_SUCCESS.getStatusCode(),
                REnum.LOGIN_SUCCESS.getStatusMsg());
    }

    @Override
    public List<User> getUserList() {

        List<User> userList = this.baseMapper.selectList(null);

        return userList;
    }

    @Override
    public PageUtils getUserPage(Map<String, Object> params) {

        IPage<User> page = this.page(new QueryPage<User>().getPage(params, true),
                new LambdaQueryWrapper<User>().like(User::getUsername,
                        (String) params.get("username")));

        return new  PageUtils(page);

    }

    @Override
    public R addUser(User user) {

        User queryUser = this.baseMapper.getUserByUserName(user.getUsername());

        if (queryUser != null){

            return R.error(REnum.USER_DOES_EXIST.getStatusCode(),
                    REnum.USER_DOES_EXIST.getStatusMsg());

        }

        String password = user.getPassword();

        String encodePassword = passwordEncoder.encode(password);

        user.setPassword(encodePassword);

        this.baseMapper.insert(user);

        return R.ok(REnum.REGISTER_SUCCESS.getStatusCode(),
                REnum.REGISTER_SUCCESS.getStatusMsg());

    }

    @Override
    public void editUser(User user) {

        this.baseMapper.updateById(user);

    }

    @Override
    public void deleteUserById(Long id) {

        this.baseMapper.deleteById(id);

    }

    @Override
    @Transactional
    public R alterPwdByUserName(User user) {

        User userByUserName = this.baseMapper.getUserByUserName(user.getUsername());

        if(userByUserName == null){

            return R.error(REnum.USER_DOES_NOT_EXIST.getStatusCode(),
                    REnum.USER_DOES_NOT_EXIST.getStatusMsg());

        }

        String password = user.getPassword();

        String encodePassword = passwordEncoder.encode(password);

        user.setPassword(encodePassword);

        this.baseMapper.alterPwdByUserName(user);

        return R.ok(REnum.ALTER_PASSWORD_SUCCESS.getStatusCode(),
                REnum.ALTER_PASSWORD_SUCCESS.getStatusMsg());
    }

    @Override
    public User selectUserByName(String username) {

        User user = this.baseMapper.getUserByUserName(username);

        return user;

    }


    // 登录逻辑
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        List<SimpleGrantedAuthority> simpleGrantedAuthorities = new ArrayList<>();

        System.out.println(username);

        try {

            User user = this.baseMapper.getUserByUserName(username);
            // System.out.println(user);

            if (user != null) {

                List<Long> roleIdList = userRoleService.selectByUserId(user.getId());

                for (Long roleId : roleIdList) {

                    Role role = roleService.getById(roleId);

                    simpleGrantedAuthorities.add(new SimpleGrantedAuthority("ROLE_" + role.getRoleName()));

                }

                List<Long> permissionIdList = userPermissionService.selectByUserId(user.getId());

                for (Long permissionId : permissionIdList) {

                    Permission permission = permissionService.getById(permissionId);

                    simpleGrantedAuthorities.add(new SimpleGrantedAuthority(permission.getPermissionName()));

                }


            }

            return new MyUser(username, user.getPassword(), simpleGrantedAuthorities);

        }catch (Exception e){

            e.printStackTrace();

            return new MyUser(username, "", simpleGrantedAuthorities);
        }
    }
}
```

### 创建自己的User认证类
实现UserDetial接口
```java
public class MyUser implements UserDetails {
    private String username;
    private String password;
    private Collection<? extends GrantedAuthority> grantedAuthorities;

    public MyUser(String username, String password, Collection<? extends GrantedAuthority> grantedAuthorities) {
        this.username = username;
        this.password = password;
        this.grantedAuthorities = grantedAuthorities;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.grantedAuthorities;
    }

    @Override
    public String getPassword() {
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
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

### 添加处理器

#### 登录成功处理器
```java
public class LoginSuccessHandler implements AuthenticationSuccessHandler {

    // private String url;

    // public LoginSuccessHandler(String url){
    //     this.url = url;
    // }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        // 获取客户端ip
        // System.out.println(request.getRemoteAddr());

        // 获取SpringSecurity里面的User对象
        // User user = (User) authentication.getPrincipal();
        // 获取权限列表
        // System.out.println(user.getAuthorities());
        //  获取用户名
        // System.out.println(user.getUsername());
        // 获取用户密码，null，为了安全
        // System.out.println(user.getPassword());
        // response.sendRedirect(this.url);

        response.setContentType("application/json;charset=utf-8");

        response.setStatus(HttpServletResponse.SC_OK);

        PrintWriter writer = response.getWriter();

        String code = (String) request.getSession().getAttribute("code");

        System.out.println(code);
        HttpSession session = request.getSession();
        String id = session.getId();
        System.out.println(id);


        if (code.toLowerCase().equals((String) request.getParameter("code").toLowerCase())){
            // Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            // System.out.println(authentication.getPrincipal());

            MyUser principal = (MyUser) SecurityContextHolder.getContext()
                    .getAuthentication()
                    .getPrincipal();

            User user = new User();

            BeanUtils.copyProperties(principal, user);

            request.getSession().setAttribute("user", user);

            writer.write(JSON.toJSONString(R
                    .ok(REnum.LOGIN_SUCCESS.getStatusCode(),
                    REnum.LOGIN_SUCCESS.getStatusMsg())));

        }else {

            writer.write(JSON.toJSONString(R.error(REnum.VALIDATE_CODE_ERROR.getStatusCode(),
                    REnum.VALIDATE_CODE_ERROR.getStatusMsg())));
        }

        writer.flush();

    }
}
```

#### 登录失败处理器
```java
public class LoginFailHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        response.setContentType("application/json;charset=utf-8");

        response.setStatus(HttpServletResponse.SC_OK);

        PrintWriter writer = response.getWriter();

        String code = (String) request.getSession().getAttribute("code");

        writer.write(JSON.toJSONString(R.ok(REnum.LOGIN_FAIL.getStatusCode(),
                REnum.LOGIN_FAIL.getStatusMsg())));

        writer.flush();

    }
}
```

#### 登出成功处理器
```java
public class LogoutSuccessHandler implements org.springframework.security.web.authentication.logout.LogoutSuccessHandler {


    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {

        response.setContentType("application/json;charset=utf-8");

        response.setStatus(HttpServletResponse.SC_OK);

        PrintWriter writer = response.getWriter();

        writer.write(JSON.toJSONString(R.ok(REnum.LOGOUT_SUCCESS.getStatusCode(),
                REnum.LOGOUT_SUCCESS.getStatusMsg())));

        writer.flush();
    }
}
```

#### 403处理器
```java
public class MyAccessDeniedHandler implements AccessDeniedHandler {


    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {

        response.setContentType("application/json;charset=utf-8");

        response.setStatus(HttpServletResponse.SC_OK);

        PrintWriter writer = response.getWriter();

        writer.write(JSON.toJSONString(R.error(REnum.USER_PERMISSIONS_ERROR.getStatusCode(),
                REnum.USER_PERMISSIONS_ERROR.getStatusMsg())));

        writer.flush();
    }
}
```

### 跨域

#### 后端
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }

}
```

#### vue(两种方式)
```shell
axios.defaults.withCredentials = true # main.js	
```

```shell
# 创建axios实例
var instance =  axios.create({
    baseURL: 'http://localhost:8888/', # 添加基本地址
    timeout: 1000, # 所有请求在超时前等待1秒
    withCredentials: true # 请求的时候携带cookie
})
```







