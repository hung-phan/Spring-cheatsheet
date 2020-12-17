## Spring cheatsheet

### IOC and Dependency injection

### Spring MVC

### Hibernate

### AOP

### Spring security

### Spring REST

#### Handle custom exception inside class

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
import lombok.Value;

@Value
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
        YourErrorResponse error = new YourErrorResponse();

        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage(exc.getMessage());
        error.setTimestamp(System.currentTimeMillis());

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

#### Handle exception globally

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
        YourErrorResponse error = new YourErrorResponse();

        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage(exc.getMessage());
        error.setTimestamp(System.currentTimeMillis());

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```
