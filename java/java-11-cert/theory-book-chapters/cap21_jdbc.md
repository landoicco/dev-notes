# Chapter 21 - JDBC

---

> 1.- Which interfaces or classes are in a database-specific JAR file?

The implementations for `Driver` and `PreparedStatement` interfaces is contained inside the database-specific driver JAR file.

**Note:** The `Driver` and `PreparedStatement` interfaces are part of the JDK, as well as the `DriverManager` class. The database-specific JAR file contains the implementation for `Driver` and `PreparedStatement` interfaces.

> 2.- Which are required parts of a JDBC URL?

The string `jdbc` and the vendor-specific string are required in a JDBC URL.

A JDBC URL is made of three parts

```
<protocol>:<subprotocol>:<subname>
```

where the protocol, is always the string `jdbc`. The subprotocol, is the name of the database such as `derby`, `mysql` or `postgres`. The subname is a database-specific format, which contains database-specific connection details. Colons (:) separate the three parts.

That said, the following are valid JDBC URL's

```
jdbc:postgresql://localhost/zoo
jdbc:oracle:thin:@123.123.123:1521:zoo
jdbc:mysql://localhost:3306
jdbc:mysql://localhost:3306/zoo?profileSQL=true
```

> 3.- Which of the following is a valid JDBC URL?

The URL

```
jdbc:sybase:localhost:1234/db
```

is the only correct option, since is the only one that preserves the rules defined on the previous question.

> 4.- Which of the options can fill in the blank to make the code compile and run without error?

```
var sql = "UPDATE habitat WHERE environment = ?";
try (var ps = conn.preparedStatement(sql)) {
	_____________________
	ps.executeUpdate();
}
```

The statements

```
ps.setString(1, "snow");
```

and

```
ps.setString(1, "snow"); ps.setString(1, "snow");
```

can fill in the blank and make the code compile and run without error. Since this query has only one parameter, we can not set more than one string. Otherwise we will get an error.

We can set the string for the same index twice since it sets the parameter and then immediately overwrites it with the same value.

**Note:** When setting parameter on a `PreparedStatement`, there are only options that take an index. For this reason, the following statement will cause an error

```
ps.setString("environment", "snow");
```

The indexing starts with `1`, so calling this method with an index of `0` will also cause an error.

> 5.- Suppose that you have a table named `animal` with two rows. What is the result of the following code?

```
6: var conn = new Connection(url, userName, password);
7: var ps = conn.prepareStatement(
8:	"SELECT count(*) FROM animal");
9: var rs = ps.executeQuery();
10: if (rs.next()) System.out.println(rs.getInt(1));
```

There is a compiler error on line 6. A `Connection` is created using a static method on `DriverManager`, it does not use a constructor.

**Note:** There are two main ways to get a `Connection`: `DriverManager` or `DataSource`, being `DataSource` the best way to do it. The following is a valid way to create a `Connection` instance

```
Connection conn = DriverManager.getConnection("jdbc:derby:zoo");
```

> 6.- Which of the options can fill in the blanks in order to make the code compile?

```
boolean bool = ps._______();
int num = ps._________();
ResultSet rs = ps._____();
```

The options

```
execute, executeUpdate, executeQuery
```

will make the code compile. Let's review this methods signatures, this methods are defined inside the `PreparedStatement` interface.

```
boolean execute() throws SQLException

int executeUpdate() throws SQLException

ResultSet executeQuery() throws SQLException
```

As we can see, `execute, updateUpdate, executeQuery` is the correct option since their method declarations match the white spaces.

> 7.- Which of the following are words in the CRUD acronym?

The acronym CRUD stands for "Create, Read, Update and Delete".

> 8.- Suppose that the table `animal` has five rows and the following SQL statement updates all of them. What is the result of this code?

```
public static void main(String[] args) throws SQLException {
	var sql = "UPDATE names SET name = 'Animal'";
	try (var conn = DriverManager.getConnection("jdbc:derby:zoo");
		var ps = conn.PrepareStatement(sql)) {

		var result = ps.executeUpdate();
		System.out.println(result);
		}
}
```

This code works as expected. It updates each of the five rows in the table and returns the number of rows updated. Therefore, this code prints `5`.

> 9.- Suppose `learn()` is a stored procedure that takes one `IN` parameter. What is wrong with the following code?

```
18: var sql = "call learn()";
19: try (var cs = conn.prepareCall(sql)) {
20:   cs.setString(1, "java");
21:	  try (var rs = cs.executeQuery()) {
22:		while (rs.next()) {
23:			System.out.println(rs.getString(3));
24:		}
25:	  }
26: }
```

The first issue is that line 18 is missing braces. For all SQL in a `CallableStatement` you are supposed to use braces (`{}`).

The second issue is that line 18 is missing a `?`. Each parameter should be passed with a question mark (`?`).

**Note:** A stored procedure is code that is compiled in advance and stored in the database. Stored procedures are commonly written in a database-specific variant of SQL, which varies among database software providers.

**Note:** A stored procedure is called by putting the word `call` and the procedure name in braces. Examples:

```
String sql = "{call read_e_names()}";
var sql = "{call read_names_by_letter(?)}";
var sql = "{?= call magic_number(?) }";
```

**Note:** An `IN` parameter is used for input, and may look like this:

```
var sql = "{call read_names_by_letter(?)}";
```

An `OUT` parameter allows stored procedures to return output. It may look like this:

```
var sql = "{?= call magic_number(?) }";
```

The two special characters (`?=`) are used to specify that the stored procedure has an output value. This is optional since we have the `OUT` parameter, but it does aid in readability.

In this example, line 18 is missing braces and a `?`. Remember that you are supposed to use braces `{}` for all SQL in a `CallableStatement`. Also, each parameter should be passed with a question mark `?`.

> 10.- Suppose that the table `enrichment` has three rows with the animals `bat`, `rat` and `snake`. How many lines does this code print?

```
var sql = "SELECT toy FROM enrichment WHERE animal = ?";
try (var ps = conn.prepareStatement(sql)) {
	ps.setString(1, "bat");

	try (var rs = ps.executeQuery(sql)) {
		while (rs.next())
			System.out.println(rs.getString(1));
	}
}
```

This code throws an `SQLException`. The code compiles because `PreparedStatement` extends `Statement` and `Statement` allows passing a `String` in the `executableQuery()` call. While `PreparedStatement` can have bind variables, `Statement` cannot. Since this code uses `executeQuery(sql)` in `Statement`, it fails at runtime.

The main issue in this code is that the method `executeQuery()` does take a parameter when called in a `Statement`, but it does not take a parameter when called in `PreparedStatement`. For this reason the variable created on the second try-with-resources block is a `Statement`, and this kind of object does not support bind variables. For this reason the code throws an exception at runtime.

> 11.- Suppose that the table `food` has five rows and this SQL statement updates all of them. What is the result of this code?

```
public static void main(String[] args) {
	var sql = "UPDATE food SET amount = amount + 1";
	try (var conn = DriverManager.getConnection("jdbc:derby:zoo");
		var ps = conn.prepareStatement(sql)) {

			var result = ps.executeUpdate();
			System.out.println(result);
		}
}
```

This code does not compile. JDBC throws a `SQLException`, which is a checked exception. The code does not handle or declare the exception, and therefore it does not compile.

If the exception were handled or declared, the answer would be `5`. Remember that `executeUpdate()` return an integer with the number of rows added/changed/removed.

> 12.- Suppose we have a JDBC program that calls a stored procedure, which returns a set of results. Which is the correct order in which to close database resources for this call?

The correct order is `ResultSet, CallableStatement, Connection`. JDBC resources should be closed in the reverse order from that in which they were opened. The order for opening is `Connection, CallableStatement, ResultSet`.

> 13.- Suppose that the table `counts` has five rows with the numbers 1 to 5. How many lines does this code print?

```
var sql = "SELECT num FROM counts WHERE num > ?";
try (var ps = conn.prepareStatement(sql)) {
	ps.setInt(1, 3);

	try (var rs = ps.executeQuery()) {
		while (rs.next())
			System.out.println(rs.getObject(1))
	}
	ps.setInt(1,100);

	try (var rs = ps.executeQuery()) {
		while (rs.next())
			System.out.println(rs.getObject(1));
	}
}
```

This code prints two lines and calls `PreparedStatement` twice. The first time, it gets the numbers greater than 3. Since there are only two such numbers, it prints two lines. The second time, it gets the numbers greater than 100. There are not such numbers, so the `ResultSet` is empty. A total of two lines is printed.

> 14.- Which of the following can fill in the blank correctly?

```
var rs = ps.executeQuery();
if (rs.next()) {
	____________________________;
}
```

The following can fill in the blank correctly:

```
- String s = rs.getString(1);
- Object s = rs.getObject(1);
```

In a `ResultSet`, columns are indexed starting with 1, not 0. There are methods to get the column as `String` or as a `Object`.

> 15.- Suppose `learn()` is a stored procedure that takes one `IN` parameter and one `OUT` parameter. What is wrong with the following code?

```
18: var sql = "{?= call learn(?)}";
19: try (var cs = conn.prepareCall(sql)) {
20:	cs.setInt(1, 8);
21:	cs.execute();
22:	System.out.println(cs.getInt(1));
23: }
```

The issue is that since an `OUT` parameter is used, the code should call` registerOutParameter()`. Since this is missing, the code is incorrect.

We should always call `registerOutParameter()` for each `OUT` or `INOUT` parameter.

**Note:** We call `execute()` instead of `executeQuery()` since we are not returning a `ResultSet`.
The `execute()` method return a boolean to indicate the form of the first result. Return `true` if the first result is a `ResultSet` object. Returns `false` if the first result is an update count or there is no result.

> 16.- Which of the following can fill in the blank?

```
var sql = "_______________________";
try (var ps = conn.prepareStatement(sql)) {
	ps.setObject(3, "red);
	ps.setInt(2, 8);
	ps.setString(1, "ball);
	ps.executeUpdate();
}
```

The line:

```
INSERT INTO toys VALUES (?, ?, ?)
```

can fill in the blank.

First, notice that this code uses a `PreparedStatement`. This means that stored procedures can not be used here, since they are for `CallableStatement`. Next, remember that the number of parameters must be an exact match.

> 17.- Suppose that the table `counts` has five rows with the numbers 1 to 5. How many lines does this code prints?

```
var sql = "SELECT num FROM counts WHERE num > ?";
try (var ps = conn.prepareStatement(sql)) {
	ps.setInt(1, 3);

	try (var rs = ps.executeQuery()) {
	while (rs.next())
		System.out.println(rs.getObject(1));
	}

	try (var rs = ps.executeQuery()) {
		while (rs.next())
			System.out.println(rs.getObject(1));
	}
}
```

This code prints 4 lines. This code calls `PreparedStatement` twice. The first time, it gets the numbers greater than 3. Since there are two such numbers, it prints two lines. Since the parameter is not set between the first and second calls, the second attempt also prints two rows. A total of four lines are printed.

> 18.- There are currently 100 rows in the table `species` before inserting a new row. What is the output of the following code?

```
String insert = "INSERT INTO species VALUES (3, 'Ant', .05)";
String select = "SELECT count(*) FROM species";
try (var ps = conn.prepareStatement(insert)) {
	ps.executeUpdate();
}
try (var ps = conn.prepareStatement(select)) {
	var rs = ps.executeQuery();
	System.out.println(rs.getInt(1));
}
```

A `SQLException` is thrown. Before accessing the data from a `ResultSet`, the cursor needs to be positioned. The call to `rs.next()` is missing from this code.

**Note:** Remember that a `ResultSet` has a cursor, which points to the current location in the data. In order to get data from a `ResultSet`, we have to position the cursor with `rs.next()`.

> 19.- Which of the options can fill in the blank to make the code compile and run without error?

```
var sql = "UPDATE habitat WHERE environment = ?";
try (var ps = conn.prepareCall(sql)) {
	_______________________
	ps.executeUpdate();
}
```

This code throws an exception at runtime. This code should call `preparedStatement()` instead of `prepareCall()` since is not executing a stored procedure. Since we are using `var`, it does compile.

Java will happily create a `CallableStatement`. Since the compile safety is lost, the code will not cause issues until runtime. At that point, Java will complain that you are trying to execute SQL as if it were a stored procedure, throwing an exception.

> 20.- Which of the following could be true of the following code?

```
var sql = "{call transform(?)}";
try (var cs = conn.prepareCall(sql)) {
	cs.registerOutParameter(1, Types.INTEGER);
	cs.execute();
	System.out.println(cs.getInt(1));
}
```

The following is true for this code: "The stored procedure must declare an `OUT` parameter".

Since this code calls `registerOutParameter()`, the stored procedure can not use an `IN` parameter. Further, there is no `setInt()`, so it can not be a an `INOUT` parameter either. Therefore, the stored procedure must be an `OUT` parameter.

**Note:** Remember to always call `registerOutParameter()` for each `OUT` or `INOUT` parameter. It allows JDBC to retrieve the value.

> 21.- Which is the first line containing a compiler error?

```
25: String url = "jdbc:derby:zoo";
26: try (var conn = DriverManager.getConnection(url);
27:	    var ps = conn.prepareStatement();
28:     var rs = ps.executeQuery("SELECT * FROM swings")) {
29:     while (rs.next()) {
30:        System.out.println(rs.getInteger(1));
31:   }
32: }
```

The first compiler error is in line 27. The `preparedStatement()` method requires SQL be passed in. Since this parameter is omitted, line 27 does not compile. Line 30 also does not compile as the method should be `getInt(1)`.
