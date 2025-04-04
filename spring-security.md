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
## Authorization

 
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
