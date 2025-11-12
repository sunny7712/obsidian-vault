#books #book_sumary #spring_boot #spring_boot_in_action

## Chapter 1

- Spring boot performs four core tricks, namely
	1. **Automatic Configuration** - Spring boot can automatically configure common scenarios, like creating beans for jdbc templates etc. if it detects them in your dependencies or classpath.
	2. **Starter dependencies** - It aggregates commonly used dependencies into a few dependencies and transitively pulls all other required dependencies without you explicitly asking for them. It also ensures that the developer won't encounter any compatibility issues among them. (Example: web starter, jpa starter etc.)
	3. **The command line interface** - Optional feature that lets you write complete applications with just application code, but no need for a traditional project build.
	4. **The Actuator** - Actuator offers the ability to inspect the inner workings of the application at runtime. For example, what beans have been configured, what decisions were made by the auto configuration etc.

## Chapter 2

- The `@SpringBootApplication` annotation enables Spring component-scanning and Spring boot auto-configuration. It combines three other useful annotations:
	1. Spring's `@Configuration` - Designates a class as a configuration class using Spring's Java based configuration.
	2. Spring's `@ComponentScan` - Enables component scanning so that controller and other component classes will be automatically discovered and registered as beans in the Spring Application Context.
	3. Spring boot's `@EnableAutoConfiguration` - Instead of you writing manual configuration of beans and everything, Spring boot handles it automatically.

- If your application requires any additional Spring configuration beyond what auto-configuration provides, then it's usually best to write out in a separate class with `@Configuration` annotation.  They'll be picked up and used by component scanning.
- `application.properties` is automatically loaded.
- By extending `JpaRepository` we inherit 18 methods for performing common persistence operations. The `JpaRepository` interface is parameterized with two parameters: the domain type(Object) that the repository will work with, and the type of its ID property.
 - You can write conditionals to activate certain beans. We have to implement the `Condition` interface and override it's `matches` method and then we can use this annotation `@Conditional` while declaring a bean. For example, ```

```java
package readinglist;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class JdbcTemplateCondition implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

	try {
		context.getClassLoader().loadClass(
    "org.springframework.jdbc.core.JdbcTemplate");
	 return true;
	
	} catch (Exception e) {
	  return false;
		}
	}
}

@Conditional(JdbcTemplateCondition.class)
public MyService myService() {
	... 
}

```
- In this case, the MyService bean will only be created if the `JdbcTemplateCondition` passes. That is to say that the MyService bean will only be created if `JdbcTemplate` is available on the **classpath**. Otherwise, the bean declaration will be ignored. There are many other conditionals which you can find in Table 2.1 of the book.

## Chapter 3
- You can override AutoConfiguration wherever you require by explicitly specifying your own configuration.
- Here's how many ways you can set your configuration properties in
	1.  Command-line arguments
	 2.  JNDI attributes from java:comp/env
    
2. 3  JVM system properties
    
3. 4  Operating system environment variables
    
4. 5  Randomly generated values for properties prefixed with random.* (referenced
    
    when setting other properties, such as `${random.long})
    
5. 6  An application.properties or application.yml file outside of the application
