# MySQL:

### [Data Types](https://www.w3schools.com/sql/sql_datatypes.asp)

### Creating a schema

In MySQL, a schema is synonymous with a database.
	
	create schema if not exists book_store;

### Creating a table
	
	use book_store;
	create table if not exists book (
		book_id int not null auto_increment primary key,
		isbn varchar (15) not null,
		publish_date date not null,
		author_id int not null,
		title varchar (70) not null,
		publisher_id int not null,
		price decimal(5,2) not null
	);

### Setting Foreign Keys

	/* Foreign Keys: book */
	alter table book add constraint fk_book_author foreign key (author_id) references author(author_id);
	alter table book add constraint fk_book_publisher foreign key (publisher_id) references publisher(publisher_id);

### CRUD Queries
	
* Create: `insert into book (title, isbn, price, publish_date, author_id, publisher_id) values (?, ?, ?, ?, ?, ?)`

* Read: `select * from book where book_id = ?` Or: `select * from book where author_id = ?`

* ReadAll: `select * from book`

* Update: `update book set title = ?, isbn = ?, price = ?, publish_date = ?, author_id = ?, publisher_id = ? where book_id = ?";`

* Delete: `delete from book where book_id = ?`

#### Notes:

* Instead of `select * from book` we can specify which columns to be rendered by using `select title, isbn from book`

* To rename headers in the result grid we can use `select title "Book Title", isbn "Book ISBN" from book`

* For the condition `where book_id = ?` we can also use comparators & boolean operators: `select * from book where book_id < ? or book_id > ?`

* In mySQL `<>` represents the "Not Equal" operator.

* For null values we can use `where book_id is null` or `where book_id is not null`

* `where city in ('Albany','Boston');` is equivalent to `where city = 'Albany' or city = 'Boston';`

### The LIKE operator

* To find entries with title starting with a specific letter (s, for example): `where title like 'S%';`

* To find entries with title containing a specific letter: `where title like '%S%';`

* To find entries with title ending with a specific letter: `where title like '%S';`

* We can use any compination we want. For example, `where title like 'S%' and isbn like 'G%';`

* Under_scores are used as place holders for letters. If we need to find entries where third letter is 'a' we can use 2 underscores: `where title like '__a%';`

### Sorting data in the result grid

* To sort data we can use `order by` with `asc` (ascending) or `desc` (descending). `asc` is set by default.

`select first_name, last_name from employees order by last_name;`

`select first_name, last_name from employees order by last_name desc;`

`select city, first_name, last_name from employees order by city, last_name;`

### Joins

* Inner join Example:

		select * from northwind.orders inner join northwind.employees on orders.employee_id = employees.id;

* Multiple Inner joins:

		select orders.id "Order ID", 
			customers.id "Customer ID",
			customers.first_name,
			customers.last_name,
			employees.first_name "Employee", 
			orders.order_status "Status",
			order_details.product_id "Product id",
			order_details.unit_price "Price",
			products.product_name "Product Name"
		from northwind.orders
		inner join northwind.employees on  employees.id = orders.employee_id
		inner join northwind.customers on orders.customer_id = customers.id
		inner join northwind.order_details on orders.id = order_details.order_id
		inner join northwind.products on order_details.product_id = products.id
		where orders.employee_id = 208 and orders.order_status = 'complete';
	
* Outer Left Join:

		select employees.first_name, employees.last_name, orders.order_status
		from northwind.employees left outer join northwind.orders
		on orders.employee_id = employees.id;
		
#### [Go Back](https://github.com/Ahmed3lmallah/Java-Portfolio/blob/master/README.md)