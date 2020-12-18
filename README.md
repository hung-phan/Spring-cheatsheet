# Spring boot 5.x cheatsheet

This is a recap from [Udemy course](udemy.com/course/spring-hibernate-tutorial) and combination of many learning
sources.

## Spring structure opinion

This is the example of common
![common-application-structure](/imgs/common-application-structure.png)

## Configuration

### @SpringBootApplication

@SpringBootApplication is a convenience annotation that adds all of the following:

@Configuration: Tags the class as a source of bean definitions for the application context.

@EnableAutoConfiguration: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various
property settings. For example, if spring-webmvc is on the classpath, this annotation flags the application as a web
application and activates key behaviors, such as setting up a DispatcherServlet.

@ComponentScan: Tells Spring to look for other components, configurations, and services in the com/example package,
letting it find the controllers.

Reference from [resource](https://spring.io/guides/gs/spring-boot/)

### Scan more packages

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(
        scanBasePackages = {
                "com.colorvisavn.springboot.demo",
                "org.colorvisavn.sample_app"
        }
)
class MyLittlePony {
}
```

### Load additional properties file from resource

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.annotation.PropertySources;

@SpringBootApplication
@PropertySource({
        "classpath:application.properties",
        "classpath:persistent-db.properties"
})
public class SpringCheatsheetApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCheatsheetApplication.class, args);
    }
}
```

or

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.annotation.PropertySources;

@SpringBootApplication
@PropertySources({
        @PropertySource("classpath:application.properties"),
        @PropertySource("classpath:persistent-db.properties")
})
@SpringBootApplication
public class SpringCheatsheetApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCheatsheetApplication.class, args);
    }
}
```

### Use property value

From application.properties for example

```properties
some.value=Hello world
```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MyLittlePony {
    private final String someValue;

    public MyLittlePony(@Value("${some.value}") String someValue) {
        this.someValue = someValue;
    }
}
```

### Some core configurations

```properties
# log levels severity mapping
logging.level.org.springframework=DEBUG
logging.level.org.hibernate=DEBUG
logging.level.com.my_company=INFO
# log file name
logging.file=application.log
# web properties
server.port=9090 # config application port
# default context path is "/"
server.servlet.context-path=/my-little-pony
# Default HTTP session time out (default value is 30m)
server.servlet.session.timeout=15m
# Actuator config
## Endpoints to include by name or wildcard
management.enpoints.web.exposure.include=*
## Endpoints to exclude by name or wildcard
management.enpoints.web.exposure.exclude=beans,mapping
## Base path for actuator endpoints
management.enpoints.web.exposure.base-path=/actuator
# Security properties - Check the example below
spring.security.user.name=admin
spring.security.user.password=secret
# Data sources
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=user
spring.datasource.password=pass
```

## IOC and Dependency injection

### @Component

When specifying class with `@Componenent`, you will create a bean (java object which its lifecycle is being managed by
spring).

```java
import org.springframework.stereotype.Component;

@Component
public class TennisCoach {
}
```

Then you can use it as follow:

```java
import org.springframework.beans.factory.annotation.Autowired;

public class MyLittlePony {
    private final TennisCoach tennisCoach;

    @Autowired
    public MyLittlePony(TennisCoach tennisCoach) {
        this.tennisCoach = tennisCoach;
    }
}
```

To define bean name, you can update `@Component` to do this

```java

@Component("myTennisCoach")
public class TennisCoach implements Coach {
}
```

Then you can refer to this using spring context

```java
public class Testing {
    public void myTesting() {
        Coach coach = context.getBean("myTennisCoach", Coach.class);
    }
}
```

### @Autowired

There are several injection types:

- Constructor injection (Most favour way of doing DI)
- Setter injection
- Field injection (by using reflection at runtime)

When there are multiple implementation of an interface, you must use `@Qualifier` for spring to know which
implementation you want it to use. For example:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

public class Testing {
    private final Coach coach;

    @Autowired
    public Testing(@Qualifier("myTennisCoach") Coach coach) {
        this.coach = coach;
    }
}
```

### @Scope

- singleton: This scopes the bean definition to a single instance per Spring IoC container (default).
- prototype: This scopes a single bean definition to have any number of object instances.
- request: This scopes a bean definition to an HTTP request. Only valid in the context of a web-aware Spring
  ApplicationContext.
- session: This scopes a bean definition to an HTTP session. Only valid in the context of a web-aware Spring
  ApplicationContext.
- global-session: This scopes a bean definition to a global HTTP session. Only valid in the context of a web-aware
  Spring ApplicationContext.

For example

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototye")
public class TennisCoach implements Coach {
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

public class Testing {
    public void myTesting() {
        Coach coach1 = context.getBean("tennisCoach", Coach.class);
        Coach coach2 = context.getBean("tennisCoach", Coach.class);

        assert coach1 == coach2;
    }
}
```

### @PostConstruct and @PreDestroy

```java
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class TennisCoach implements Coach {
    @PostConstruct
    public void doSthAfterBeanCreation() {
    }

    @PreDestroy
    public void doSthToCleanup() {
    }
}
```

## Spring MVC

### Entity

```java

import javax.persistence.*;

@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "email")
    private String email;

    public Customer(String firstName, String lastName, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }

    public Customer() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return String.format("Customer [id=%d, firstName=%s, lastName=%s, email=%s]", id, firstName, lastName, email);
    }

}
```

### DAO

```java
public interface CustomerDAO {
    public List<Customer> getCustomers();

    public void saveCustomer(Customer theCustomer);

    public Customer getCustomer(int theId);

    public void deleteCustomer(int theId);
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.luv2code.springdemo.entity.Customer;

import javax.persistence.EntityManager;
import javax.persistence.Query;

@Repository
public class CustomerDAOImpl implements CustomerDAO {
    private final EntityManager entityManager;

    @Autowired
    public CustomerDAOImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    public List<Customer> getCustomers() {
        // create a query  ... sort by last name
        Query<Customer> theQuery =
                entityManager.createQuery("from Customer order by lastName",
                        Customer.class);

        // execute query and get result list
        List<Customer> customers = theQuery.getResultList();

        // return the results
        return customers;
    }

    @Override
    public void saveCustomer(Customer theCustomer) {
        // save/update the customer
        Customer customerFromDb = entityManager.merge(theCustomer);

        theCustomer.setId(customerFromDb.getId());
    }

    @Override
    public Customer getCustomer(int theId) {
        // now retrieve/read from database using the primary key
        return entityManager.find(Customer.class, theId);
    }

    @Override
    public void deleteCustomer(int theId) {
        // delete object with primary key
        Query theQuery =
                entityManager.createQuery("delete from Customer where id=:customerId");

        theQuery.setParameter("customerId", theId);

        theQuery.executeUpdate();
    }
}

```

### Service

```java
public interface CustomerService {
    public List<Customer> getCustomers();

    public void saveCustomer(Customer theCustomer);

    public Customer getCustomer(int theId);

    public void deleteCustomer(int theId);
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

@Service
public class CustomerServiceImpl implements CustomerService {
    private final CustomerDAO customerDAO;

    @Autowired
    public CustomerServiceImpl(CustomerDAO customerDAO) {
        this.customerDAO = customerDAO;
    }

    @Override
    @Transactional
    public List<Customer> getCustomers() {
        return customerDAO.getCustomers();
    }

    @Override
    @Transactional
    public void saveCustomer(Customer theCustomer) {
        customerDAO.saveCustomer(theCustomer);
    }

    @Override
    @Transactional
    public Customer getCustomer(int theId) {
        return customerDAO.getCustomer(theId);
    }

    @Override
    @Transactional
    public void deleteCustomer(int theId) {
        customerDAO.deleteCustomer(theId);
    }
}
```

### Controller

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class CustomerRestController {
    private final CustomerService customerService;

    @Autowired
    public CustomerRestController(CustomerService customerService) {
        this.customerService = customerService;
    }

    // add mapping for GET /customers
    @GetMapping("/customers")
    public List<Customer> getCustomers() {
        return customerService.getCustomers();
    }

    // add mapping for GET /customers/{customerId}

    @GetMapping("/customers/{customerId}")
    public Customer getCustomer(@PathVariable int customerId) {
        return customerService.getCustomer(customerId);
    }

    @PostMapping("/customers")
    public Customer addCustomer(@RequestBody Customer customer) {
        // also just in case someone passes id in JSON
        // set id to 0. This is the force a save of new item instead of update
        customer.setId(0);

        customerService.saveCustomer(customer);

        return customer;
    }

    @PutMapping("/customers")
    public Customer updateCustomer(@RequestBody Customer customer) {
        customerService.saveCustomer(customer);

        return customer;
    }

    @DeleteMapping("/customers/{customerId}")
    public String deleteCustomer(@PathVariable int customerId) {
        Customer customer = customerService.getCustomer(customerId);

        if (customer == null) {
            throw new CustomerNotFoundException();
        }

        customerService.deleteCustomer(customerId);

        return "Deleted customer id - %d".formatted(customerId);
    }
}
```

### Handle custom exception inside class

```java
import lombok.Value;

@Value
class YourErrorResponse {
    int status;
    String message;
    long timestamp;
}
```

```java
class YourException extends Exception {
    public YourException(String message, Throwable cause) {
        super(message, cause);
    }

    public YourException(String message) {
        super(message);
    }

    public YourException(Throwable cause) {
        super(cause);
    }
}
```

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RestExceptionHandler {
    @GetMapping("/data/{dataId}")
    public YourResource getResource(@PathVariable int dataId) {
        throw YourException("Cannot process this entity");
    }

    @ExceptionHandler
    public ResponseEntity<YourErrorResponse> handleException(YourException exc) {
        YourErrorResponse error = new YourErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                exc.getMessage(),
                System.currentTimeMillis()
        );

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

### Handle exception globally

```java
import lombok.Value;

@Value
class YourErrorResponse {
    int status;
    String message;
    long timestamp;
}
```

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

/**
 * Use ControllerAdvice as AOP to handle global exception
 */
@ControllerAdvice
public class RestExceptionHandler {
    @ExceptionHandler
    public ResponseEntity<YourErrorResponse> handleException(Exception exc) {
        YourErrorResponse error = new YourErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                exc.getMessage(),
                System.currentTimeMillis()
        );

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

### Getting data from client

1. For query params

```java
import org.springframework.ui.Model;

import javax.servlet.http.HttpServletRequest;

public class Testing {
    public String getDataFromClient(HttpServletRequest req, Model model) {
        System.out.println(req.getParameter("myQueryParam"));
    }
}
```

or

```java
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestParam;

import javax.servlet.http.HttpServletRequest;

public class Testing {
    public String getDataFromClient(@RequestParam("myQueryParam") String myQueryParam, Model model) {
        System.out.println(myQueryParam);
    }
}
```

2. For model param

```java
import org.springframework.web.bind.annotation.ModelAttribute;

public class Testing {
    public String processFormData(@ModelAttribute("employee") Employee employee) {
    }
}
```

### Data validation

Some annotation examples:

![data-validation](/imgs/data-validation.png)

Usage:

```java
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class Customer {
    @NotNull(message = "is required")
    @Size(min = 1)
    private String name;
}
```

Perform validation check

```java
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.ModelAttribute;

import javax.validation.Valid;

public class Testing {
  public String processFormData(@Valid @ModelAttribute("customer") Customer customer, BindingResult bindingResult) {
      if (bindingResult.hasErrors()) {
          return "customer-form"; // you can handle the error in the jsp page
      }
      
      return "customer-confirmation";
  }
}
```

## Spring security

```java
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // add our users for in memory authentication
        UserBuilder users = User.withDefaultPasswordEncoder();

        auth.inMemoryAuthentication()
                .withUser(users.username("john").password("test123").roles("EMPLOYEE"))
                .withUser(users.username("mary").password("test123").roles("EMPLOYEE", "MANAGER"))
                .withUser(users.username("susan").password("test123").roles("EMPLOYEE", "ADMIN"));

        // OR

        // use jdbc authentication
//        auth.jdbcAuthentication().dataSource(securityDataSource);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests()
                .antMatchers("/").hasRole("EMPLOYEE")
                .antMatchers("/leaders/**").hasRole("MANAGER")
                .antMatchers("/systems/**").hasRole("ADMIN")
                .and()
                .formLogin()
                .loginPage("/showMyLoginPage")
                .loginProcessingUrl("/authenticateTheUser")
                .permitAll()
                .and()
                .logout().permitAll()
                .and()
                .exceptionHandling().accessDeniedPage("/access-denied");

    }

}
```

## Spring data jpa

To reduce the repetition in creating DAOs, use spring-data-jpa as follow:

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface CustomerRepository extends JpaRepository<Customer, Integer> {
}
```

`JpaRepository` will accept EntityClass and primary key. After that, you will get all of these methods for free.

```java
public class CustomerService {
    private final CustomerRepository customerRepository;

    public void testReposistory() {
        customerReposistory.findAll();
        customerReposistory.findById();
        customerReposistory.save();
        customerReposistory.deleteBy();
        // etc
    }
}
```

When using JpaRepository implementation, you can remove `@Transactional` annotation from service class because it is
taken care of inside data jpa package.

## Spring data rest

To reduce boilerplate for REST endpoints, you can use spring-data-rest. There are some prerequisites before you can
fully utilise the feature.

```md
1. Your entity: Employee
2. JpaRepository: EmployeeRepository extends JpaRepository
3. Dependency: spring-boot-starter-data-rest
```

![spring-data-reset](/imgs/spring-data-rest.png)

Spring data rest are also HATEOAS compliant.

### Some configurations

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

/**
 * In case your want to specify the endpoint prefix
 */
@RepositoryRestResource(path = "members")
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
}
```

For application.properties

```properties
spring.data.rest.base-path=/my-funky-endpoints
spring.data.rest.default-page-size=30
```
