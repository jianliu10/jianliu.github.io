---  
layout: post  
title:  "Application Security notes - 2018"  
date:   2018-11-01 00:00:00 -0500  
categories: tech security  
---  
  
## Authentication and Authorization in Micro-services architecture ##  
  
in Micro-services architecture, Requests arrive at the API Gateway. The API Gateway may handle, among other things, load balancing and authentication.  
"iss": "https://LCL_APIGW_DOMAIN/",  # JWT id_token is issued by IAM authentication and authorization service.  
shared cookies (session id) across sites by using API gateway.  
Switching from a session based architecture to a token based architecture may be done incrementally.  
  
## legacy web security mechanism: session ##  
session id + 100% stateful session  
stateful session, that requires shared session store  
  
## Modern web security mechanism: JWT ##  
JWT  + 100% stateless session,  
or a hybrid of JWT + session store (keep JWT token size moderate, store non-frequently used session data in a shared session store like distributed cache Redis or Infinispan)  
JWT token stores the session state. JWT token is passed between browser and micro-services.   JWT token is passed in HTTP Header.  
  
OAuth2 is used for authorization with scopes (access_token JWT),  OAuth2 access_token.   httpheader "Authorization: bearer <access_token>", has scp (scopes) claim.  
"OpenID Connect" is used for authentication and identity user details (id_token JWT)  
JWT: Json Web Token (header, claims, signature.  
  JSON Web Token (JWT) spec, along with the JSON Web Signature (JWS) and JSON Web Encryption (JWE) specs, which complement the actual data format with signatures and encryption.  
  
## JWT, com.auth0:java-jwt:3.4.0:jar ##  
two ways to secure JWT:  
- unencrypted JWT token, but with signature for CLAIMs data intergration check.  
- Encrypted JWT token without signature.  
  
Most of time, JWTs are usually just signed. However, if your JWT contains data that must not be visible to third parties, then encrypting it using JWE is your only choice.  
  
Any application (including front-end) can encode and decode JWT token.  
front-end can not verify JWT token. Only back-end knowing the Algorithm Secret key can verify JWT token.  
  
**Algorithm:**  
RSA256, which use publicKey/privateKey  
HMAC256, which use a secret  
HS256, which use a secret. HS256 is simply an HMAC + SHA256 algorithm with Base64 encoding  
  
## Spring Security authentication and Urls authorization ##  
  
@Configuration @EnableWebSecurity  
see WebSecurityConfigurer, WebSecurityConfigurerAdapter  
  
aven dependency: spring-boot-starter-security  
  
auto rediect from http->https or from https->http depending on the dest Uri requires secure channel or not.  
  
**Legacy xml config **  
  
in WEB-INFO/web.xml, add springSecurityFilterChain config,  
  
web-info/web.xml configs "springSecurityFilterChain":  

	<filter>  
		<filter-name>springSecurityFilterChain</filter-name>  
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
	</filter>  
	<filter-mapping>  
		<filter-name>springSecurityFilterChain</filter-name>  
		<url-pattern>/*</url-pattern>  
	</filter-mapping>  
  
In Spring config xml, security configs will be added to springSecurityFilterChain;  
  
	<interceptor-url pattern="/customers/**" access="isAuthenticated() and hasROle("USER") requires-channel="https"/>  
  
**new java class configuration **  
@EnableWebSecurity auto adds "springSecurityFilterChain"  
  
com.acom.cardinal.rest.security.configuration.CardinalSecurityConfigAdapter extends WebSecurityConfigurerAdapter  
  
	@EnableWebSecurity  
	class MySecurityConfig extends WebSecurityConfigurerAdapter {  
		@Override  
		protected void configure(HttpSecurity http) throws Exception {  
		  http.authorizeRequests()  
			  .antMatchers("/customers/**").hasRole("USER")  
			  .antMatchers("/public/**").permitAll()  
			  .anyRequest().authenticated()  
			  .and()  
			  //following enables https for the specified URL pattern  
			  .requiresChannel()  
			  .antMatchers("/customers/**").requiresSecure()  
			  .antMatchers("/accounts/**").requiresSecure()  
			  //.anyRequest().requiresSecure()  # for all requests to go through https  
			.and()  
			.httpBasic().disable()  
			.formLogin()  
			.loginPage("/iam/signin").permitAll()  
			.failureUrl("/error")  
			.defaultSuccessUrl("/app/dashboard")  
			.and()  
			.logout()  
			.logoutRequestMatcher(new AntPathRequestMatcher("iam/logout"))  
			.logoutSuccessUrl("/iam/signin");			  
		}  
		@Override  
		public void configure(WebSecurity web) throws Exception {  
			// these Urls do not require authentication checks.  
			web.ignoring().antMatchers(authenticationJwtUnsecuredUrls);  
		}		  
		@Override  
		protected void configure(AuthenticationManagerBuilder auth) throws Exception {  
			// Create a default account for tests  
			auth.inMemoryAuthentication()  
					.withUser("admin")  
					.password("password")  
					.roles("ADMIN");  
		}		  
	}  
	  
	  
## how to handle JWT token leak	 ##  
store blacklist for revoked tokens or set a short expiration date for tokens.  
  
## Spring security using JWT	 ##  
https://auth0.com/blog/implementing-jwt-authentication-on-spring-boot/  
  
## Spring GLobal Method Security ##  
  
    <dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-security</artifactId>  
    </dependency>  
    <dependency>  
    <groupId>org.springframework.security</groupId>  
    <artifactId>spring-security-test</artifactId>  
    </dependency>  
  
E.g.  
  
    @Configuration  
    @EnableGlobalMethodSecurity(  
      prePostEnabled = true,  
      securedEnabled = true,  
      jsr250Enabled = true)  
    public class MyMethodSecurityConfig extends GlobalMethodSecurityConfiguration {  ... }	  
	  
The prePostEnabled property enables Spring Security pre/post annotations  
The securedEnabled property determines if the @Secured annotation should be enabled  
The jsr250Enabled property allows us to use the @RoleAllowed annotation  
  
The @Secured annotation is used to specify a list of roles on a method. The @Secured annotation doesn’t support Spring Expression Language (SpEL).  
The @RoleAllowed annotation is the JSR-250’s equivalent annotation of the @Secured annotation  
Both @PreAuthorize and @PostAuthorize annotations provide expression-based access control. Hence, predicates can be written using SpEL (Spring Expression Language).  
The @PreAuthorize annotation checks the given expression before entering the method, whereas,  
the @PostAuthorize annotation verifies it after the execution of the method and could alter the result. the @PostAuthorize annotation provides the ability to access the method result.  
@PreFilter annotation to filter a collection argument before executing the method  
@PostFilter annotation to filter the returned collection of a method.  
  
Security meta-annotation  
  
By default, Spring AOP proxying is used to apply method security – if a secured method A is called by another method within the same class, security in A is ignored altogether. This means method A will execute without any security checking. The same applies to private methods  
  
Spring SecurityContext is thread-bound (thread local)  
SecurityContextHolder -> thread local SecurityContext obj -> Authentication object -> list of GrantedAuthority objects  
  
Examples:  
  
    @Secured("ROLE_VIEWER")  
    public String getUsername() {  
    SecurityContext securityContext = SecurityContextHolder.getContext();  
    return securityContext.getAuthentication().getName();  
    }  
  
    @Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })  
    @RolesAllowed("ROLE_VIEWER")  
    @RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })  
    @PreAuthorize("hasRole('ROLE_VIEWER')")  
    @PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")  
  
    @PreAuthorize("#username == authentication.principal.username")  
    public String getMyRoles(String username) {  
    //...  
    }	  
  
    @PostAuthorize("returnObject.username == authentication.principal.nickName")  
    public CustomUser loadUserDetail(String username) {  
    return userRoleRepository.loadUserByUserName(username);  
    }  
  
    @PreFilter (value = "filterObject != authentication.principal.username",  filterTarget = "usernames")  
    public String joinUsernamesAndRoles(List<String> usernames, List<String> roles) { ... }  
    @PostFilter("filterObject != authentication.principal.username")  
    public List<String> getAllUsernamesExceptCurrent() {  
    return userRoleRepository.getAllUsernames();  
    }  
  
Test:  
  
Note that it isn’t necessary to add the ROLE_ prefix in @WithMockUser, Spring Security will add that prefix automatically. If we don’t want to have that ROLE_ prefix, we can consider using authority instead of role.  
  
    @Test  
    @WithMockUser(value = "john", roles = "VIEWER")  
    @WithAnonymousUser  
    @WithUserDetails(value = "john",  userDetailsServiceBeanName = "userDetailService")  
    public void whenJohn_callLoadUserDetail_thenOK() { ... }  
  
This UserDetailsService might be a real implementation or a fake for testing purposes.  
  
  
  
## google cloud platform ##  
JWT id_token:  
	eyJhbGciOiJSUzI1NiIsImtpZCI6ImRhZDQ0NzM5NTc2NDg1ZWMzMGQyMjg4NDJlNzNhY2UwYmMzNjdiYzQifQ.eyJhenAiOiIzMjU1NTk0MDU1OS5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsImF1ZCI6IjMyNTU1OTQwNTU5LmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwic3ViIjoiMTA2MTIzNzIxOTYwMDE0MjQ2NDA0IiwiZW1haWwiOiJqYW5lLmZ6LmxpdUBnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiYXRfaGFzaCI6Im5VM3p5UW9PYVhESWNYeEJtYl9MZFEiLCJleHAiOjE1Mjk5NDg4MTMsImlzcyI6Imh0dHBzOi8vYWNjb3VudHMuZ29vZ2xlLmNvbSIsImlhdCI6MTUyOTk0NTIxM30.C9HZtuiZqNxv43A8VM5aX_D4DhtNClxd8arNkjtZm-h3zzkJh3FNDruhh9xK553CX-VPCHPs_hVu-2WEy3gqjRWpZjjQcO3_2jQR6WpU0AbnKUbO0a-NF6k4cC-fOBo2yLpSvpQUn0a_mKUplTsp7mYoblkBx0DFreVEYYMblttaHENw8M3cRsvg2qUv76AyM5ZUGPRbnOcVEwxjP8lFBj20gThEDESOucMBE-eyPO7Axhr1g71mP4JhmKW8neRDqDRaxyCPjzyOjKUxAa7J7pgMrn0hvyX1BLgCfzuCjNbxu8ZYnxoz0JjHEQp1jIR6cE1zj4PB2PTOEF_696hgDw  
  
  
Decoded Token:  
    {  
    "alg": "RS256",  
    "kid": "dad44739576485ec30d228842e73ace0bc367bc4"  
    }.{  
    "azp": "32555940559.apps.googleusercontent.com",  
    "aud": "32555940559.apps.googleusercontent.com",  
    "sub": "106123721960014246404",  
    "email": "jane.liu.test@gmail.com",  
    "email_verified": true,  
    "at_hash": "nU3zyQoOaXDIcXxBmb_LdQ",  
    "exp": 1529948813,  
    "iss": "https://accounts.google.com",  
    "iat": 1529945213  
    }.[Signature]  
  
  
## ACOM Cardinal security design ##  
cardinal-channel-common.cardinal-channel-security module is the security core. Another is cardinal-channel-authentication module.  
Cardinal micro-services do NOT use Spring-security's session id authentication, Urls authorization, or global method authorization.  
  
    @EnableWebSecurity  
    public class CardinalSecurityConfigAdapter extends WebSecurityConfigurerAdapter {  
      ...  
    }  
  

CSRF - Cross Site Request Forgery
  
in its void configure(HttpSecurity httpSecurity) method implementation, the only useful httpSecurity config is "csrf().disable()", ".addFilterBefore(jwtAuthenticationTokenFilter()". The rest httpSecurity configs are never used because jwtAuthenticationTokenFilter filter handles all the JWT authentications and Url authorizations. The filter returns/throws immediately from the SpringSecurityFilterChain chain.  
  
cardinal ThreadLocal ExecutionContext.java is used, instead of Spring's SecurityContext.  
JwtAuthenticationTokenFilter is the only filter used for JWT authentication and cardinal priopriatory Urls authorization.  
business specific finer grained method authorizations are done using cardinal @Authorize() annotation at API method level.  
  
examples:  
"ciam_username" is used as login id, "nonce" or "jti (JWT id)" is used as session id,  
"iam_role" is used as surrogate code to map to a <Algorithm, Secret> that is used to calculate signature for the claims body. the mappings are stored on server side.  
  
JWT id_token:  
	 "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjaWFtX3VzZXJuYW1lIjoiZy5zb3R0aWxlQGdtYWlsLmNvbSIsInN1YiI6ImRiZDExOGJhLTlkYmYtNDM4ZS1hZWI0LTUzMjI1NmRiNDQyMSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJsb2NrX3N0YXR1cyI6Ik4iLCJsYXN0X2xvZ2luIjoiU2F0IFNlcCAxNSAyMDoxMToxOSBFRFQgMjAxOCIsImlhbV9yb2xlIjoiQ3VzdG9tZXJfUENGIiwiaXNzIjoiaHR0cHM6Ly9MQ0xfQVBJR1dfRE9NQUlOLyIsImdpdmVuX25hbWUiOiJnLnNvdHRpbGVAZ21haWwuY29tIiwibm9uY2UiOiIgMTA4ZjY2MzgtZGNiNC00MDFhLWE3NzAtMDVkODc4MzE5YzgxIiwiaWFtX2F1dGhfYXNzdXJhbmNlX2NvZGUiOiJBbGxvdyIsImF1ZCI6ImNsaWVudF9pZD8iLCJwY2ZfY3VzdF9pZCI6IjAwMDAwNTk5MjM3NjciLCJleHAiOjE1MzcwNjM4NzgsImlhdCI6MTUzNzA1NjY3OCwiZmFtaWx5X25hbWUiOiJDdXN0b21lcl9QQ0YiLCJlbWFpbCI6Imcuc290dGlsZUBnbWFpbC5jb21AZ21haWwuY29tIiwianRpIjoiZDAyM2EzZDQtZTBmOC00MGUwLTlhZmQtYWY4MWJmNmQ5YTVjIn0.HmWOU0U5jqwUFRsNrR62SAHP7UFTlHh4T-dSJQXyUCE",  
  
Decoded Token:	  
    {  
    "typ": "JWT",  
    "alg": "HS256"  
    }.{  
    "ciam_username": "g.sottile@gmail.com",  
    "sub": "dbd118ba-9dbf-438e-aeb4-532256db4421",  
    "email_verified": true,  
    "lock_status": "N",  
    "last_login": "Sat Sep 15 20:11:19 EDT 2018",  
    "iam_role": "Customer_ACOM",  
    "iss": "https://LCL_APIGW_DOMAIN/",  
    "given_name": "g.sottile@gmail.com",  
    "nonce": " 108f6638-dcb4-401a-a770-05d878319c81",  
    "iam_auth_assurance_code": "Allow",  
    "aud": "client_id?",  
    "acom_cust_id": "0000059923767",  
    "exp": 1537063878,  
    "iat": 1537056678,  
    "family_name": "Customer_ACOM",  
    "email": "jane.liu.test@gmail.com",  
    "jti": "d023a3d4-e0f8-40e0-9afd-af81bf6d9a5c"  
    "scope": ["admin", "user"]  
    }.[Signature]  
  
## shopping cart example: ##  
Using com.Auth0 lib and JWTs for Authentication and Client Side Sessions  
  
In this example, there are multiple JWTs present. Our shopping cart will be one of them.  
One cookie id_token - JWT for the ID token, a token that carries the user's profile information, useful for the UI.  
One cookie 'access_token' in response, or bearer in request - JWT for interacting with the API backend (the access token).  
One cookie 'cart' - JWT for our client-side state: the shopping cart.  
  
Note that the frontend does not check the signature, it simply decodes the JWT so it can display its contents. The actual checks are performed by the backend. All JWTs are verified.  
When items are added, the backend constructs a new JWT with the new item in it and a new signature:  
Note that locations prefixed by /protected are also protected by the API access token. This is set up using express-jwt  
  
    app.get('/protected/add_item', idValidator, cartValidator, (req, res) = {  
      req.cart.items.push(parseInt(req.query.id));  
      const newCart = jwt.sign(req.cart,  
                               process.env.AUTH0_CART_SECRET,  
                               cartSignJwtOptions);  
      res.cookie('cart', newCart, {  
                      maxAge: 1000 * 60 * 60  
                });  
    }  
  
    app.use('/protected', expressJwt({  
      secret: jwksClient.expressJwtSecret(jwksOpts),  
      requestProperty: 'accessToken',  
      getToken: req = {  
                       return req.cookies['access_token'];  
                     }  
    }));  
  
The /protected/add_item endpoint must first pass the access token validation step before validating the cart. One token validates access (authorization) to the API and the other token validates the integrity of the client side data (the cart).  
  