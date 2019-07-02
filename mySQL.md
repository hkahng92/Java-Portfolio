# SQL:

	CREATE DATABASE **database_name**;

	USE **database_name**;
	CREATE TABLE **table_name** (
	  id INT AUTO_INCREMENT NOT NULL,
	  Title VARCHAR(250) NOT NULL,
	  Author VARCHAR(250) NOT NULL,
	  In_stock BOOLEAN DEFAULT TRUE,
	  Quantity INT,
	  PRIMARY KEY (id)
	  );

	USE **database_name**;
	INSERT INTO **table_name** (title, author, quantity) VALUES ('Title 1','Author 1',19);
	UPDATE **table_name** SET quantity = 10 WHERE ID=2;
	DELETE FROM **table_name** WHERE id = 3;
	SELECT * FROM **table_name**;
	SELECT title, author FROM **table_name**;
