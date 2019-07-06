# MySQL:

### [Data Types](https://www.w3schools.com/sql/sql_datatypes.asp)

### Creating a schema

In MySQL, physically, a schema is synonymous with a database.
	
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

Remeber to `use book_store;`
	
Create: `insert into book (title, isbn, price, publish_date, author_id, publisher_id) values (?, ?, ?, ?, ?, ?)`

Read: `select * from book where book_id = ?` Or: `select * from book where author_id = ?`

ReadAll: `select * from book`

Update: `update book set title = ?, isbn = ?, price = ?, publish_date = ?, author_id = ?, publisher_id = ? where book_id = ?";`

Delete: `delete from book where book_id = ?`

### Joins

