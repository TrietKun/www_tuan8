# www_tuan8
<h2>Spring security với database</h2>

<h3>In Memory Authentication</h3>
Cấu trúc project như hình. Thêm vào các gói config để chứa lớp cấu hình, gói controllers chứa các controller
demo. Trong thư mục static của resources, tạo một file index.html với nội dung tùy ý.

![image](https://github.com/TrietKun/www_tuan8/assets/103935961/d48b6594-c5f4-4edf-aacc-5e1f9b120ad9)

Trong lớp SecurityConfig, ta cấu hình như code sau
```
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    @Autowired
    public void globalConfig(AuthenticationManagerBuilder auth, PasswordEncoder encoder, DataSource dataSource)throws Exception{
        auth.jdbcAuthentication()
                .dataSource(dataSource)
                .withDefaultSchema()
                .withUser(User.withUsername("admin")
                        .password(encoder.encode("admin"))
                        .roles("ADMIN")
                        .build())
                .withUser(User.withUsername("teo")
                        .password(encoder.encode("teo"))
                        .roles("TEO")
                        .build())
                .withUser(User.withUsername("ty")
                        .password(encoder.encode("ty"))
                        .roles("USER")
                        .build())
        ;
    }
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth->auth
                        .requestMatchers("/","/home","/index").permitAll()
                        .requestMatchers("/api/**").hasAnyRole("ADMIN","USER","TEO")
                        .requestMatchers("/h2-console/**").permitAll()
                        .requestMatchers(("/admin/**")).hasRole("ADMIN")
                        .anyRequest().authenticated()
        );
        http.csrf(csrf->csrf.ignoringRequestMatchers("/h2-console/**"));
        http.headers(headers->headers.frameOptions(HeadersConfigurer.FrameOptionsConfig::sameOrigin));
        http.httpBasic(Customizer.withDefaults());
        return http.build();
    }
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

![image](https://github.com/TrietKun/www_tuan8/assets/103935961/aaac9a10-42c7-4e98-8fe8-3794162c421e)
![image](https://github.com/TrietKun/www_tuan8/assets/103935961/eea119ff-cce9-4703-88f9-89334d663d79)

<h2>Chú ý:</h2>
Khi chạy ứng dụng sẽ gặp lỗi
![image](https://github.com/TrietKun/www_tuan8/assets/103935961/9f3fcc3f-0bd8-4edd-bca8-889adb82e879)
Trường hợp này bạn thêm dòng config sau vào application.properties
![image](https://github.com/TrietKun/www_tuan8/assets/103935961/a00f3b36-4441-4ea1-a107-e0b5fbf91ade)

<h2>In JDBC Authentication</h2>
Với việc lưu trữ user vào database, bạn cần thêm các dependencies cho việc truy xuất đến csdl đích. Ví dụ này
dùng H2 relational database

```
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.springframework.boot:spring-boot-starter-web'implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
compileOnly 'org.projectlombok:lombok'
annotationProcessor 'org.projectlombok:lombok'
runtimeOnly 'com.h2database:h2'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.security:spring-security-test'
```

![image](https://github.com/TrietKun/www_tuan8/assets/103935961/e9a73237-e9b1-421a-8781-7b2d73076b0d)



