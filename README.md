# springboot-security-mysql
This project shows how to configure spring security with mysql in the rest implementation


### Springboot Security with mysql database ###

#### Keypoints to remember ###

* In this application, I have used three tables users, role and user_roles for authentication and authorization

* Implemented UserDetailsService provided by Spring Security for authenticate user by username in custom userservice class.

```
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Users> users = userRepository.findByUsername(username);
        users.orElseThrow(() -> new UsernameNotFoundException("User not found"));
        return users.map(CustomUserDetails::new).get();
    }

```

* Created custom user details with implements UserDetails provide by Spring Security.

    * Extends User class in CustomUserDetails for authentication/authorization.
    
    * Get the list of roles from the Granted Authority provided by Spring Security.
    
    ```
        return getRoles()
                        .stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getRole()))
                        .collect(Collectors.toList());

    ```

* Configured CustomUserDetailsService in SecurityConfig
 
```
@Configuration
@EnableJpaRepositories(basePackageClasses = UserRepository.class)
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomUserDetailsService customUserDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(customUserDetailsService).passwordEncoder(getPasswordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable().authorizeRequests().antMatchers("**/api/**").authenticated().anyRequest().permitAll().and().formLogin().permitAll();
    }

    private PasswordEncoder getPasswordEncoder() {
        return new PasswordEncoder() {
            @Override
            public String encode(CharSequence charSequence) {
                return charSequence.toString();
            }

            @Override
            public boolean matches(CharSequence charSequence, String s) {
                return true;
            }
        };
    }
}

```

* Configured authorization in rest controller

```
    @PreAuthorize("hasAnyRole('ADMIN')")
    @GetMapping("/admin")
    public String securedUser(){
        return "Welcome admin to this tutorial ";

    }

```

* When access this rest end point

    http://localhost:8085/api/admin
    
    It will redirect into login page and if you provide the user who has admin role it will navigate to respective page 
    and give the output.
   

### Happy Coding ###