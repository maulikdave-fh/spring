# spring-security
is one of the spring projects that help secure spring app resources including REST APIs, microservices and web apps.

By default, everything is protected. 

# spring-mvc flow

```mermaid
    sequenceDiagram
        Request ->> DispatcherServlet:  
        DispatcherServlet ->> Controller(s): DispatcherServlet acts as the front controller<br/>It intercepts all the requests. <br/>DispatcherServlet routes request to an appropriate Controller<br/> based on the request URL pattern
```

# spring-security flow
```mermaid
    sequenceDiagram
        Request ->> Spring Security:  
        Spring Security ->> DispatcherServlet: Spring Security intercepts all the requests<br/>using configured filter chain
        DispatcherServlet ->> Controller(s): 
```
A filter chain will typically have multiple security filters configured.

# How spring-security works
Spring security executes a series of configured filters. Filters provides following features;
## Authentication
Example; BasicAuthenticationFilter
## Authorization
Example; AuthorizationFilter
## Others
### CORS (Cross Origing Resource Sharing)
Example; CorsFilter<br/>
Checks if AJAX calls from other domains should be allowed.
```mermaid
    sequenceDiagram
        React Application (www.frontend.com) ->> Spring Boot RESTful API (www.restapi.com):   
        Spring Boot RESTful API (www.restapi.com) ->> Database:   
```
### CSRF (Cross Site Request Forgery)
Example; CsrfFilter<br/>
A malicious web-site making use of previous authenticated session on your website. It leverages existing cookies in your browser that may be tied to an active session.
### Default Login, Logout Pages
Examples; LogoutFilter, DefaultLoginPageGeneratingFilter, DefaultLogoutPageGeneratingFilter
### Excetion Translation
Examples; ExceptionTranslationFilter<br/>
Exceptions are translated to HTTP codes. For example; 401 Access Denied, 403 Authorization Failed

# Order of filters
is important. Typical order;
## Basic Checks 
CORS, CSRF

## Authentication
Key componenents
AuthenticationManager -> AuthenticationProvider(s) -> UserDetailsService
1. AuthenticationManager - responsible for authentication. Interaction multiple AuthenticationProviders. Has ```Authentication authenticate(Authentication authentication)``` method. Before authentication, ```Authentication``` object contains ```Credentials```. After authentication, ```Authentication``` object also contains ```Principal``` and ```GrantedAuthority```.
2. AuthenticationProviders - perform specific authentication type. Loads use data using UserDetailsService. On successful authentication, AuthenticationProvider populates Principal and Authorities. Examples; JwtAuthenticationProvider
3. UserDetailsService - core interface to load user data

Authentication result is stored in SecurityContextHolder (SecurityContext ( Authentication (Principal, Credentials, GrantedAuthority))). GrantedAuthority - An authority granted to Principal (roles, scopes, ...)

## Authorization
Two Approaches;
### Global Security: authorizeHttpRequests
```.requestMatchers("/users").hasRole("USER")```
hasRole, hasAuthority, hasAnyAuthority, isAuthenticated
### Method Security ```@EnableMethodSecurity``` for Java Config Class
1. @Pre and @Post annotations <br/>
```@PreAuthorize("hasRole('USER') and #username==authentication.name")```<br/>
```@PostAuthorize("returnObject.username=='moe'")<br/>
2. jsr-250 Annotations - ```@EnableMethodSecurity (jsr250Enabled=true)```<br/>
```@RolesAllowed({"USER", "ADMIN"})```<br/>
3. ```@Secured``` annotation - old one - ```@EnableMethodSecurity(securedEnabled=true)```<br/>
```@Secured({"ROLE_ADMIN", "ROLE_USER"})```

 
# Default spring-security configuration
## Zero Trust
All requests are authenticated. It can be customized
## Form Authentication
is enabled with default form and logout features. Most of the web application use form based authentication.
1. It creates a session cookie (JSESSIONID: E50668BDDC640D88A74D288AC81287E9) on successful login and sends to a browser.
2. Browser sends this cookie in all seqsequent requests of the session to server
Form based authentication provides a default login page, logout page and /logout url.
## Basic Authentication
is  enabled. It is the most basic option to secure REST APIs. 
1. Base64 encoded username and password is sent as part of the request header - Authorization: Basic aWjluydf6lkdsaj8fouyY=
2. But it has lot of flaws - not recommended for production use. Password can be decoded, No expiry, No role / access details
## Test User
is created. Credentails printed in logs. Username is <b>user</b>
## CSRF protection
is enabled for all update requests. POST, PUT, etc.
1. You're logged into your bank's website. A cookie (say Cookie-A) is stored into your browser
2. You visit malicious website without logging out from bank's site
3. Malicious website executes a bank transfer without your knowledge using Cookie-A
To protect from CSRF
1. Synchronizer token pattern - a token created for each request
2. To make an update (POST, PUT..), you need a CSRF token from the previous request in X-CSRF-TOKEN header
3. CSRF may not be required if REST APIs are stateless
4. Another approach is SameSite cookie (Set-Cookie: SameSite=Strict) - This means that the browser will send this cookie only to the site that originated it. To enable it, add server.servlet.session.cookie.same-site=strict to application.properties

### Disable CSRF for Stateless REST APIs
1. SpringBootWebSecurityConfiguration has all the default configs of spring security
## CORS requests
are denied, by default
### Browsers don't allow AJAX calls to resources outside current origin
### CORS specs that allows you to configure which cross-domain requests are allowed
1. Global Configration - applicable to all Controllers; Configure ```addCorsMappings``` callback method in ```WebMvcConfigurer```
2. Local Configuration - can be done at each Controller or request method. @CrossOrigin - allow from all origins; @CrossOrigin(origins="https://in.foresthut") - allow from specific origins
## X-Frame-Options 
is set to 0 (Frames are disabled)

# Password Hashing
## PasswordEncoder
```PasswordEncoder``` interface has multple implementations available in Spring for various algos like Bcrypt, Argon2, etc.

# JWT Usage
## JWT Authentication
JWT Authentication using Spring Boot's OAuth2 Resource Server
### Create Keypair
Using ```openssl``` or ```java.security.KeyPairGenerator```
### Create RSA Key using the keypair
```com.nimbusds.jose.jwk.RSAKey```
### Create JWKSource (JSON Web Key Source)
1. Create ```JWKSet``` (new JSON Web Key Set) with the RSA Key
2. Create ```JWKSource``` using the ```JWKSet```
### User RSA Public Key for decoding
```NimbusJwtDecoder.withPublicKey(rsaKey().toRSAPublicKey()).build()```
### Use JWKSource for encoding
```return new NimbusJwtEncoder(jwkSource());```

