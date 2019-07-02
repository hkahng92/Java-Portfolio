# Spring Initializer Dependencies

* Spring Web Starter : Spring Boot
* Spring HATEOAS: Necessary for Exception handling
* JDBC API: Database Connectivity API that defines how a client may connect and query a database.
* Spring Data JPA: Persist data in SQL stores with Java Persistence API using Spring Data and Hibernate.
* MySQL Driver: MySQL JDBC Driver

Note: to use TOMCAT as an embedded servlet container, make sure to use Jar format.	
	
# Spring Annotations

One use for annotations is **Dependency Injection**, where Spring implicitly use input coming from the request (path or body) to instantiate objects: DTO [Data Transfer Objects] 

**Example:**

	@RestController
	public class EchoRangeServiceController {
		@RequestMapping(value = "/echo/{input}", method = RequestMethod.GET)
		@ResponseStatus(value = HttpStatus.OK)
		public int echo(@PathVariable int input) {
			if (input < 1 || input > 10) {
				throw new IllegalArgumentException("Input must be between 1 and 10.");
			}
			return input;
		}
	}

## Class-level annotation

`@RestController`: Marks the class as a Rest Controller and makes Spring aware of its existence. Also, configures Spring so that it treats all returned values from methods as JSON and sends those values back to the client.

## Method-level annotation

`@RequestMapping(value="/", method=RequestMethod.GET)`: Maps an endpoint to a method that will handle requests to that endpoint

* Two parameters:
	* Value: contains the URI path for this mapping
	* Method: specifies the HTTP method for this mapping

* Must be unique across all controllers.
* Can only map a given endpoint to one method in an application.

`@ResponseStatus(value = HttpStatus.OK)`: HTTP status code that is sent back when the method successfully handles the incoming request

## Method parameter-level annotation

`@PathVariable`: Maps a path variable (marked by {} in the path) to a method parameter. Placed before the method parameter.

* If the name of the path variable is the same as the name of the method variable, no further configuration is needed.
	* If they don't match, you can specify which path variable to map to the parameter like this: @PathVariable("pathVarName")

`@RequestBody`: Maps JSON in the request body to a method parameter

### Input Validation with JSR 303

Defines a set of annotations and behaviors for that define validation rules for properties in a Java object

`@Valid` tells the Spring framework to run the validation rules. `@Valid` is a method parameter-level annotation.

In the model files, we can use `NotEmpty`, `NotNull`, `Max`, etc to enforce validation on the model properties.

## Error Handlers

Create components that control how errors are handled and reported.

Use the `@RestControllerAdvice` annotation. Class-level annotation that registers a class as the error handler for all Controllers in the application.

Also use the `@ExceptionHanlder` method-level annotation to specify which methods in the error-handler class handle which exceptions.

**Example 1: Handling Validation Errors**

	@RestControllerAdvice
	@RequestMapping(produces = "application/vnd.error+json")
	public class ControllerExceptionHandler {
		@ExceptionHandler(value = {MethodArgumentNotValidException.class})
		@ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
		public ResponseEntity<VndErrors> motorcycleValidationError(MethodArgumentNotValidException e, WebRequest request) {
			// BindingResult holds the validation errors
			BindingResult result = e.getBindingResult();
			// Validation errors are stored in FieldError objects
			List<FieldError> fieldErrors = result.getFieldErrors();
			// Translate the FieldErrors to VndErrors
			List<VndErrors.VndError> vndErrorList = new ArrayList<>();
			for (FieldError fieldError : fieldErrors) {
				VndErrors.VndError vndError = new VndErrors.VndError(request.toString(), fieldError.getDefaultMessage());
				vndErrorList.add(vndError);
			}
			// Wrap all of the VndError objects in a VndErrors object
			VndErrors vndErrors = new VndErrors(vndErrorList);
			// Create and return the ResponseEntity
			ResponseEntity<VndErrors> responseEntity = new ResponseEntity<>(vndErrors, HttpStatus.UNPROCESSABLE_ENTITY);
			return responseEntity;
		}
	}

**Example 2: Handling Other Exceptions**

Simply replace `IllegalArgumentException` with exception type. 

    @ExceptionHandler(value = {IllegalArgumentException.class})
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ResponseEntity<VndErrors> illegalArgumentException(IllegalArgumentException e, WebRequest request) {
        VndErrors error = new VndErrors(request.toString(), e.getMessage());
        ResponseEntity<VndErrors> responseEntity = new ResponseEntity<>(error, HttpStatus.UNPROCESSABLE_ENTITY);
        return responseEntity;
    }
	
	@ExceptionHandler(value = {InvalidFormatException.class})
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ResponseEntity<VndErrors> invalidFormatException(InvalidFormatException e, WebRequest request) {
        VndErrors error = new VndErrors(request.toString(), "Population must be a whole number & aCapital must be a Boolean: " + e.getMessage());
        ResponseEntity<VndErrors> responseEntity = new ResponseEntity<>(error, HttpStatus.UNPROCESSABLE_ENTITY);
        return responseEntity;
    }

    @ExceptionHandler(value = {JsonParseException.class})
    @ResponseStatus(HttpStatus.UNPROCESSABLE_ENTITY)
    public ResponseEntity<VndErrors> jsonFormatException(JsonParseException e, WebRequest request) {
        VndErrors error = new VndErrors(request.toString(), "Json file is not formatted correctly: " + e.getMessage());
        ResponseEntity<VndErrors> responseEntity = new ResponseEntity<>(error, HttpStatus.UNPROCESSABLE_ENTITY);
        return responseEntity;
    }
	
### [Creating a Spring Project with Database access using JDBC](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/SpringJDBC.md)

### [Creating a Spring Project with Database access using JPA](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/SpringJPA.md)

#### [Go Back](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)
