# Aspect Oriented Programming
Typically, some NFRs / Quality requirements are common across multiple layers of a layered app. Some examples of the common NFRs are security, logging, monitoring. These are also called cross-cutting concerns / cross-cutting aspects.

Aspect Oriented Programming can be used to implement cross-cutting concerns following DRY (Don't Repeat Yourself) principle. It also allows to keep the layers' code free of these concerns. It makes code more maintainable & easy to reason about.

# Steps
1. Implement the cross-cutting concern as an aspect. Aspect is a separate code.
2. Define pointcuts to indicate where the aspect should be applied

# Popular AOP Frameworks
1. AspectJ is a complete AOP solution with a lot of flexibility. Examples, intercept any method call on any Java class; intercept change of values in a field 
2. spring-aop is one of the spring framework modules. Not a complete one, but popular in spring domain. It only works with spring beans. Example; intercept method calls to spring beans

# AOP Terminology
## Compile Time Terminology
### Advice
Tells what code to execute. 
Example; logging, authentication
### Pointcut
Expression that identifies method calls to be intercepted
Example; example(* in.foresthut.*.*(..)) - to intercept all method calls in in.foresthut package
### Aspect
Aspect is a combination of
1. Advice
2. Pointcut
### Weaver
It is a framework that implements AOP. It ensure that the advice is applied at configured pointcut
Examples; spring AOP, AspectJ
Entire process of execution of aspect is called weaving

## Runtime Terminology
### Join Point
When a pointcut condition is true, the advice is executed. A specific execution instance of an advice is called a Join Point.


# Using spring-aop
## Add Dependency
		```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		```
## Define Aspect
Create aspect class and tell spring that this is a configuration class and defines an aspect.
        ```java           
            @Configuration
            @Aspect
            public class TimerAspect {
                //Pointcut - when to call this method?
                //execution(* PACKAGE.*.*(..))  
                @Around("execution * in.foresthut.*.*(..)")
                public void logCall(ProceedingJoinPoint joinPoint) {
                    //Logic - what to do?                    
                }
            }
        ```

# Important Annotations
1. @Before - Do something before a method is called. Example; authenticate user and put the principal in the context
2. @After - Do something after a method is executed, irrespective of whether 1. method executes successfully or 2. method throws an error
3. @AfterReturning - Do something only when a method executes successfully
4. @AfterThrowing - Do something only when a method throws an exception. Example; Spring Data uses it to convert checked exceptions to unchecked ones
5. @Around - Do something before and after a method execution. Example; capture execution duration

# Common Pointcut Definitions
Centralized pointcut configuration - DRY
```java
    // Centrailized pointcut configs
    package in.foresthut.permaculture.aspects;
    public class CommonPointcutConfig {
        @Pointcut("execution(* in.foresthut.permaculture.services.*.*(..))")
        public void servicesPackageConfig(){}
        
        @Pointcut("execution(* in.foresthut.permaculture.repositories.*.*(..))")
        public void repositoriesPackageConfig(){}
        
        // Another way to define a common pointcut - all spring beans having "Service" in 
        // their names will be intercepted
        @Pointcut("bean(*Service*)")
        public void serviceBeans() {}
        
        // Helpful to apply the aspect to specific bean methods. TimeTrack annotation has to be created
        @Pointcut("@annotation(in.foresthut.permaculture.annotations.TimeTrack)")
	    public void timeTrackConfig() {}
    }
    
    // Aspect
    @Configuration
    @Aspect
    public class TimerAspect{
        @Around("in.foresthut.aspects.CommonPointcutConfig.servicesPackageConfig()")
        public Object elapsedTime(ProceedingJoinPoint joinPoint) {
            // Do your magic
        }
    }
    
```
# Custom Annotation
Useful to target specific methods

```java
    // Annotation
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface TimeTrack {}
    
    // Common Pointcut config
    @Pointcut("@annotation(in.foresthut.annotations.TimeTrack)")
	public void timeTrackConfig() {
	}
	
	// Aspect
	@Around("in.foresthut.aspects.CommonPointcutConfig.timeTrackConfig()")
	public Object elapsedTime(ProceedingJoinPoint joinPoint) throws Throwable {
	    // Do your magic here
	}
	
	// Method timing to track
	@Component
	public class MyService {
	    @TimeTrack
	    public void foo {
	    }
	}
    
```
