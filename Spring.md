# Spring Initializer Dependencies

* Spring Web Starter : Spring Boot
* Spring HATEOAS: Necessary for Exception handling
* Spring Web Services
* JDBC API: Database Connectivity API that defines how a client may connect and query a database.
* MySQL Driver: MySQL JDBC Driver

Note: to have TOMCAT, make sure to use Jar format.

# Spring Project Compostion

Project Name: App

* main.java.com.company.artifactName (package)
	* AppApplication.java (used to run Service)
	* controller (package)
		* AppController.java
		* ControllerExceptionHandler.java
	* model (package)
		* AppModel.java - Data Transfer Objects [DTO]
	* dao (package) - Data Access Objects [DAO]
		* AppDao.java (Interface)
		* AppDaoJdbcTemplateImpl.java
* main.resources (package)
	* application.properties
* test.java.com.company.artifactName (package)
	* *test files*
* test.resources (package)
	* application.properties	
	
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
	
# Creating a Spring Project with Database access using DAOs

**Check list: Using Car Lot project example**

1. Configure the Pom.xml file by adding `[<version>5.1.46</version>]` to SQL dependency
1. Creating DTO (models)
	1. Adding properties, getters, and setters
	1. Overriding .equals method
	
			@Override
			public boolean equals(Object o) {
				if (this == o) return true;
				if (o == null || getClass() != o.getClass()) return false;
				Car that = (Car) o;
				return getId() == that.getId() &&
						Objects.equals(getMake(), that.getMake()) &&
						Objects.equals(getModel(), that.getModel()) &&
						Objects.equals(getYear(), that.getYear()) &&
						Objects.equals(getColor(), that.getColor());
			}
	
	1. Overriding .hashCode method
	
			@Override
			public int hashCode() {
				return Objects.hash(getId(), getMake(), getModel(), getYear(), getColor());
			}
	
1. Creating AppDAO interface: with basic CRUD methods to Create, Read, Update and Delete from the database.

	**Example:**
	
		public interface CarLotDao {
			Car getCar(int id);
			List<Car> getAllCars();
			List<Car> getCarsByMake(String make);
			List<Car> getCarsByColor(String color);
			Car addCar(Car car);
			void updateCar(Car car);
			void deleteCar(int id);
		}
			
1. Implementing the interface: AppDaoJdbcTemplateImpl.java - `implements CarLotDao`
	1. At this stage, we just override methods in the interface without caring about methods' body.
	1. Add `@Repository` annotation to the class
	1. Use the `@Transactional` for multi-query methods.
1. Creating a test for the interface:
	1. From the interface file, auto generate a test for all methods and include a `@Before` method
	1. Add class-level annotations:
		1. `@RunWith(SpringJUnit4ClassRunner.class)`
		1. `@SpringBootTest`
	1. Use `@Autowired` to connect the test with DAO: An example of Dependency Injection
			
			@Autowired
			protected CarLotDao dao;
	
	1. Add method-level annotations: `@Before`, `@BeforeClass`, and `@Test`
	1. Design a test implementation for each DAO method. We can also test several DAO methods in one test.
	
	**Example:**
			
		@Before
		public void setUp() throws Exception {
			// clean out the test db
			List<Car> carList = dao.getAllCars();
			for (Car t : carList) {
				dao.deleteCar(t.getId());
			}
		}

		@Test
		public void addGetDeleteACar() {

			Car car = new Car();

			car.setMake("Honda");
			car.setModel("Accord");
			car.setYear("2019");
			car.setColor("Black");

			car = dao.addCar(car);

			Car car2 = dao.getCar(car.getId());

			assertEquals(car, car2);

			dao.deleteCar(car.getId());

			car2 = dao.getCar(car.getId());

			assertNull(car2);
		}

		@Test
		public void updateCar() {

			Car car = new Car();

			car.setMake("Honda");
			car.setModel("Accord");
			car.setYear("2019");
			car.setColor("Black");

			car = dao.addCar(car);

			car.setColor("Red");

			dao.updateCar(car);

			car = dao.getCar(car.getId());

			assertEquals(car.getColor(),"Red");
		}

		@Test
		public void getAllCarsAndByMakeAndColor() {

			Car car = new Car();

			car.setMake("Honda");
			car.setModel("Accord");
			car.setYear("2019");
			car.setColor("Black");

			dao.addCar(car);

			car.setMake("Toyota");
			car.setModel("Highlander");
			car.setYear("2020");
			car.setColor("Red");

			dao.addCar(car);

			car.setMake("Toyota");
			car.setModel("Camry");
			car.setYear("2015");
			car.setColor("White");

			dao.addCar(car);

			car.setMake("Kia");
			car.setModel("Rio");
			car.setYear("2010");
			car.setColor("Gray");

			dao.addCar(car);

			car.setMake("Kia");
			car.setModel("Forte");
			car.setYear("2019");
			car.setColor("Black");

			dao.addCar(car);

			List<Car> carList = dao.getAllCars();

			assertEquals(carList.size(), 5);

			carList = dao.getCarsByMake("Toyota");

			assertEquals(carList.size(), 2);

			carList = dao.getCarsByMake("Honda");

			assertEquals(carList.size(), 1);

			carList = dao.getCarsByColor("Black");

			assertEquals(carList.size(), 2);

			carList = dao.getCarsByColor("White");

			assertEquals(carList.size(), 1);
		}
	
1. Configure application.properties (for application)

		spring.datasource.url: jdbc:mysql://localhost:3306/car_lot?useSSL=false
		spring.datasource.username: root
		spring.datasource.password: rootroot
		spring.datasource.driver-class-name: com.mysql.jdbc.Driver

1. Configure application.properties (for test)

		spring.datasource.url: jdbc:mysql://localhost:3306/car_lot_test?useSSL=false
		spring.datasource.username: root
		spring.datasource.password: rootroot
		spring.datasource.driver-class-name: com.mysql.jdbc.Driver

1. Create schemas and tables: Check mySQL.md
1. We go back to the actual implementation of the DAO class: 
	1. Add the JdbcTemplate constructor
	
		**Example:**

			//Constructor
			private JdbcTemplate jdbcTemplate;

			@Autowired
			public CarLotDaoJdbcTemplateImpl(JdbcTemplate newJdbcTemplate){
				this.jdbcTemplate = newJdbcTemplate;
			}
	
	1. Add a Mapper to the implementation of DAO interface (once per model)
	
		**Example:**

			//Mappers (One per Model)
			private Car mapRowToCar(ResultSet rs, int rowNum) throws SQLException {
				Car car = new Car();
				car.setId(rs.getInt("id"));
				car.setMake(rs.getString("make"));
				car.setModel(rs.getString("model"));
				car.setYear(rs.getString("year"));
				car.setColor(rs.getString("color"));

				return car;
			}
			
	1. Add Prepared statement strings to the implementation of DAO interface
	
		**Example:**
		
			// Prepared statement strings
			private static final String INSERT_CAR_SQL =
					"insert into car (make, model, year, color) values (?, ?, ?, ?)";

			private static final String SELECT_CAR_SQL =
					"select * from car where id = ?";

			private static final String SELECT_ALL_CARS_SQL =
					"select * from car";

			private static final String DELETE_CAR_SQL =
					"delete from car where id = ?";

			private static final String UPDATE_CAR_SQL =
					"update car set make = ?, model = ?, year = ?, color = ? where id = ?";

			private static final String SELECT_CARS_BY_MAKE_SQL =
					"select * from car where make = ?";

			private static final String SELECT_CARS_BY_COLOR_SQL =
					"select * from car where color = ?";
	
	1. Implementing methods using `jdbcTemplate.query` or `jdbcTemplate.queryforObject`
	
		**Example:**
		
			@Override
			public Car getCar(int id) {
				try {

					return jdbcTemplate.queryForObject(SELECT_CAR_SQL, this::mapRowToCar, id);

				} catch (EmptyResultDataAccessException e) {
					// if nothing is returned just catch the exception and return null
					return null;
				}
			}

			@Override
			public List<Car> getAllCars() {
				try {
					return jdbcTemplate.query(SELECT_ALL_CARS_SQL, this::mapRowToCar);
				} catch (NullPointerException e) {
					return null;
				}
			}

			@Override
			public List<Car> getCarsByMake(String make) {
				return jdbcTemplate.query(SELECT_CARS_BY_MAKE_SQL, this::mapRowToCar, make);
			}

			@Override
			public List<Car> getCarsByColor(String color) {
				return jdbcTemplate.query(SELECT_CARS_BY_COLOR_SQL, this::mapRowToCar, color);
			}

			@Override
			@Transactional
			public Car addCar(Car car) {
				jdbcTemplate.update(INSERT_CAR_SQL,
						car.getMake(),
						car.getModel(),
						car.getYear(),
						car.getColor());

				int id = jdbcTemplate.queryForObject("select last_insert_id()", Integer.class);

				car.setId(id);

				return car;
			}

			@Override
			public void updateCar(Car car) {
				jdbcTemplate.update(UPDATE_CAR_SQL,
						car.getMake(),
						car.getModel(),
						car.getYear(),
						car.getColor(),
						car.getId());
			}

			@Override
			public void deleteCar(int id) {
				jdbcTemplate.update(DELETE_CAR_SQL, id);
			}
			
1. Run the tests, refactor...		