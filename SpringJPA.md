# Spring Project: Java DAO Using JPA

### Spring Initializer Dependencies

* Spring Web Starter
* Spring Data JPA
* MySQL Driver

### Spring Project Compostion

Project Name: App

* main.java.com.company.artifactName
	* controller
		* AppController `<Class>`
		* ControllerExceptionHandler `<Class>`
	* dto - Contains Data Transfer Objects [DTO]
		* AppModel `<Class>`
	* dao - Contains Data Access Objects [DAO]
		* Repository`<Interface>`
	* AppApplication `<Class>` - used to run Service
* main.resources
	* application.properties - used to setup DB connection
* test.java.com.company.artifactName
	* *test files* `<Class>`
* test.resources
	* application.properties - used to setup test DB connection
* pom.xml - used to setup dependencies

### **Check list - Using Simple CRM project example**

1. Project Creation
	1. Create project with Spring Initializr. Include web, mysql, jpa
	1. Create schema and run db creation script with MySQL Workbench: [MySQL](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/mySQL.md)
1. Configuration
	1. Configure the Pom.xml file by adding `<version>5.1.46</version>` to SQL dependency
	1. Configure application.properties (for application)

			spring.jpa.hibernate.ddl-auto=create
			spring.datasource.url=jdbc:mysql://localhost:3306/simple_crm?useSSL=false
			spring.datasource.username=root
			spring.datasource.password=rootroot
			spring.jpa.show-sql=true
			
		`spring.jpa.hibernate.ddl-auto` allows us to specify how we want Spring Data JPA to act when we start our application
		`spring.jps.show-sql=true` allows us to see the SQL statements that Spring Data JPA is executing
			
1. Creating DTO (models)
	1. Adding properties, getters, and setters
	1. Annotate DTO so it can be persisted to the schema: For JPA/Hibernate to automatically create database tables, we have to tell it what our tables should look like.
		
		* Tables: To persist the class to a specific table in DB:
		
				@Entity
				@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})
				@Table(name="customer")
				public class Customer {...
			
		`@JsonIgnoreProperties` can include any additional properties that we wish to ignore. For now, it should include `{"hibernateLazyInitializer", "handler"}`
		
		* Attributes: `@Id` is used for the property that is going to be the primary key. No annotations is needed for other properties.
			
				@Id
				@GeneratedValue(strategy = GenerationType.AUTO)
				private Integer id;
				private String firstName;
				private String lastName;
			
		`@GeneratedValue` is used to auto increment id. `GenerationType.IDENTITY` was found to behave better than `GenerationType.AUTO`.
		
		* Child Tables: We use Sets to define child tables in the Parent class. Then, use the `@OneToMany` to join using the Foriegn key.
		
				@OneToMany(mappedBy = "customerId", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
				private Set<Note> notes;
			
		`cascade = CascadeType.ALL` means changes done to Parent will affect all child entries.
		`fetch = FetchType.EAGER` means all related child entries will be fetched with the parent. To conserve memory we can use `fetch = FetchType.LAZY`
	
	1. Overriding .equals method
	
			@Override
			public boolean equals(Object o) {
				if (this == o) return true;
				if (o == null || getClass() != o.getClass()) return false;
				Customer customer = (Customer) o;
				return Objects.equals(getId(), customer.getId()) &&
						Objects.equals(getFirstName(), customer.getFirstName()) &&
						Objects.equals(getLastName(), customer.getLastName()) &&
						Objects.equals(getCompany(), customer.getCompany()) &&
						Objects.equals(getPhone(), customer.getPhone()) &&
						Objects.equals(getNotes(), customer.getNotes());
			}
	
	1. Overriding .hashCode method
	
			@Override
			public int hashCode() {
				return Objects.hash(getId(), getFirstName(), getLastName(), getCompany(), getPhone(), getNotes());
			}
			
1. Creating [JPA Repository](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html): A marker interface with basic CRUD methods included — we do not need to provide an implementation, not even method signatures, for CRUD queries.

	**Example:**
	
		@Repository
		public interface CustomerRepository extends JpaRepository<Customer, Integer> {
			List<Customer> findByLastNameAndCompany(String lastName, String company);
		}
	
	We can add custom query methods by just adding the method definition to the interface, such as `findByLastNameAndCompany`.
	
1. Creating unit/integration tests:
	1. Configure application.properties (for test)

			spring.jpa.hibernate.ddl-auto=create
			spring.datasource.url=jdbc:mysql://localhost:3306/simple_crm_test?useSSL=false
			spring.datasource.username=root
			spring.datasource.password=rootroot
			spring.jpa.show-sql=true
			
	1. Add class-level annotations:
			
			@RunWith(SpringRunner.class)
			@SpringBootTest
			public class CrmApplicationTests {...
	
	1. Use `@Autowired` to connect the test with Repositories
			
			@Autowired
			CustomerRepository customerRepo;

			@Autowired
			NoteRepository noteRepo;
	
	1. Add method-level annotations: `@Before`, `@BeforeClass`, and `@Test`
	1. Design a test implementation for each Repo method.
	
	**Example:**
			
		@Before
		public void setUp() throws Exception {
			noteRepo.deleteAll();
			customerRepo.deleteAll();
		}

		@Test
		public void createTest() {
			
			Note note = new Note();
			note.setContent("This is a test note");
			Set<Note> noteSet = new HashSet<>();
			noteSet.add(note);

			Note note2 = new Note();
			note2.setContent("This is the SECOND test note.");
			noteSet.add(note2);

			Customer customer = new Customer();
			customer.setFirstName("Joe");
			customer.setLastName("Smith");
			customer.setPhone("111-222-3456");
			customer.setCompany("BigCo");
			customerRepo.save(customer);
			note.setCustomerId(customer.getId());
			note2.setCustomerId(customer.getId());

			noteRepo.save(note);
			noteRepo.save(note2);

			List<Customer> customerList = customerRepo.findAll();
			assertEquals(1, customerList.size());

			noteSet =  customerList.get(0).getNotes();
			assertEquals(2, noteSet.size());
		}
		
1. Run the tests, re-factor...
1. To use JPA DAO in the controller, we simply `@Autowired` the Repositories in the controller class to use their methods:

		@Autowired
		private CustomerRepository customerRepo;

		@Autowired
		private NoteRepository noteRepo;

		@RequestMapping(value="/customer", method = RequestMethod.POST)
		public Customer createCustomer(@RequestBody Customer customer) {
			customerRepo.save(customer);
			return customer;
		}
		
### Common Exceptions, and how to handle them

* Not being able to connect to database
	* Solution: add `spring.datasource.driver-class-name: com.mysql.jdbc.Driver` to application.properties
* SSL-related Exception
	* Solution: Adding the following dependency might help
	
			<dependency>
				<groupId>javax.xml.bind</groupId>
				<artifactId>jaxb-api</artifactId>
				<version>2.3.0</version>
			</dependency>


#### [Go Back](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)