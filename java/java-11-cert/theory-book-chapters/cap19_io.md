# Chapter 19 - I/O

---

> 1.- Which class would be best to use to read a binary file into a Java object?

The best class would be one that deserialize the data. Therefore, `ObjectInputStream` is the best choice. The `ObjectInputStream` class is a high level class that deserializes primitive Java data types and graphs of Java objects from an existing `InputStream`.

> 2.- Which of the following are methods available on instances of the `java.io.File` class?

The command to move a file or directory using a `File` instance is `renameTo()`. The commands to create a directory using a `File` instance are `mkdir()` and `mkdirs()`. The method `mkdirs()` differs from `mkdir()` by creating any missing directories along the path.

> 3.- What is the value of `name` after the instance of `Eagle` created in the main method is serialized and then deserialized?

    import java.io.Serializable;
    class Bird {
    	protected transient String name;
    	public void setName(String name) {this.name = name;}
    	public String getName() {return name;}
    	public Bird() {
    		this.name = "Matt";
    	}
    }
    public class Eagle extends Bird implements Serializable {
    	{this.name = "Olivia"}
    	public Eagle() {
    		this.name = "Bridget";
    	}
    	public static void main(String[] args) {
    		var e = new Eagle();
    		e.name = "Adeline";
    	}
    }

The value of `name` is Matt after deserialization. The key here is that while `Eagle` is serializable, it's parent class, `Bird`, is not. Therefore, none of the members of `Bird` will be serialized. During deserialization, Java calls the no-arg constructor of the first nonserializable parent. In this case, the `Bird` constructor is called, with `name` being set to Matt.

Note that none of the constructors or instance initializers in `Eagle` is executed as part of deserialization.

> 4.- Which classes will allow the following to compile?

    var is = new BufferedInputStream(new FileInputStream("z.txt"));
    InputStream wrapper = new __________________(is);
    try (wrapper) {}

The reference type of `wrapper` is `InputStream`, so we need a class that inherits `InputStream`. The class also must take another stream input, so we need to choose a stream that is high level stream. `BufferedInputStream` is a high level stream. Even though the instance is already a `BufferedInputStream`, there's no rule that it can't be wrapped multiple times by a high level stream.

`ObjectInputStream` is also a high level stream that operates on other streams. This class also inherits `InputStream`, so we can use this class as well.

> 5.- Which of the following are true?

`System.console()` will return `null` if a `Console` is not available. The JVM creates one instance of the `Console` object as a singleton. If the console is unavailable, `System.console()` will return `null`. The `Console` class includes a `format()` method to write data to the output stream.

The `Console` class does not include a `println()` method. To use `println()`, the method `writer()` must be called first.

> 6.- Which statements about closing I/O streams are correct?

All I/O streams should be closed after use or a resource leak might ensue. While a try-with-resources statement is the preferred way to close an I/O stream, it can be closed with a traditional `try` statement that uses a `finally` block.

> 7.- Assume that `in` is a valid stream whose next bytes are `XYZABC`. What is the result of calling the following method on the stream, using a `count` value of 3?

    public static String pullBytes(InputSteam in, int count) throws IOException {
    	in.mark(count);
    	var sb = new StringBuilder();
    	for(int i = 0; i < count; i++) {
    		sb.append((char) in.read());
    	}
    	in.reset();
    	in.skip(1);
    	sb.append((char) in.read());
    	return sb.toString();
    }

The result cannot be determined with the information given. Not all I/O streams support the `mark()` operation; therefore, without calling `markSupported()` on the stream, the result is unknown until runtime.

If the stream does support the `mark()` operation, the result will be `XYZY`. The `reset()` operation puts the stream back in the position before the `mark()` was called, and `skip(1)` will skip `X`.

If the stream does not support `mark()` operation, a runtime exception would likely be thrown. Since we don't know if the input stream supports `mark()`, the result cannot be determined until runtime.

> 8.- Which of the following are true statements about serialization in Java?

In Java, serialization is the process of turning an object to a stream, while deserialization is the process of turning that stream back into an object. The `Serializable` interface is a marker interface that does not contain any abstract methods. The `readObject()` method of `ObjectInputStream` declares the `ClassNotFoundException` even if the class is not cast to a specific type.

> 9.- Assuming `/` is the root directory within the sile system, which of the following are true statements?

The path `/home/parrot` is an absolute path. Paths that begin with the root directory are absolute paths. The path `/home/parrot` could be a file or directory within the file system. There is no rule that files have to end with a file extension.

It is posible to create a `File` reference to files and directories that do not exist. The `delete()` method called on a `File` instance return `false` if the file or directory cannot be deleted. Doesn't throw an exception.

> 10.- What are the requirements for a class that you want to serialize to a stream?

For a class to be serialized, it must implement the `Serializable` interface and contain instance members that are serializable or marked `transient`. Every instance member of the class should be serializable, marked `transient` or has a `null` value at the time of serialization.

Marking a class `final` does not impact its ability to be serialized. While it's a good practice for a serializable class to include a `static  serialVersionUID` variable, it's not required. Other than the `serialVersionUID`, only the instance members of a class are serialized. `static` members of the class are ignored on serialization.

> 11.- Given a directory `/storage` full of multiple files and directories, what is the result of executing the `deleteTree("/storage")` method on it?

    public static void deleteTree(File file) {
    	if(!file.isFile()) {                                                 // n1
    		for(File entry: file.listFiles()) {
    			deleteTree(entry);
    			}
    	} else {
    		file.delete();
    	}
    }

This code compiles and will delete all non-directory files within the directory tree. The line `n1` will check that the file is NOT a normal file, aka, a directory. The `isFile()` method will return `true` for any non-directory file. The `entry` variable will take an abstract pathname for each file and directory inside the file. However, this code has a bug on it, it never deletes any directories, only files. The `delete()` method is called just when the `file` variable is a non-directory file.

> 12.- What are the posible results of executing the following code?

    public static void main(String[] args) {
    	String line;
    	var c = System.console();
    	Writer w = c.writer;
    	try(w) {
    		if((line = c.readLine("Enter your name: ")) != null) {
    			w.append(line);
    		}
    		w.flush();
    	}
    }

This code does not compile. The `Writer` methods `append()` and `flush()` both throw an `IOException` that must be handled or declared. Even without those lines of code, the try-with-resources statement itself must be handled or declared, since the `close()` method throws a checked `IOException`. If the `main` method was corrected to declare `IOException`, then the code will compile. If the `Console` was not available, it would throw a `NullPointerException` on the call to `c.writer()`.

> 13.- Suppose that the absolute path `/weather/winter/snow.dat` represents a file that exists within the file system. Which of the following lines of code creates an object that represents the file?

We can create an object that represents a file using `new File("/weather/winter/snow.dat")`. This is the proper way to create a `File` instance with a single `String` parameter.

There is a constructor that takes a `File` followed by a `String`, so we could also use `new File(new File("/weather/winter"), "snow.dat")`.

> 14.- Which of the following are built-in streams in java?

The `System` class has three streams: `in` is for input, `err` is for error and `out` is for output. Therefore, `System.in`, `System.err` and `System.out` are built-in streams in Java.

> 15.- Which of the following are not `java.io`classes?

The class `PrintReader` does not exist. `PrintStream` and `PrintWriter` are I/O classes that do not have a complementary `InputStream` or `Reader` class.

> 16.- Assuming `zoo-data.txt` exists and is not empty, what statements about the following method are correct?

    private void echo() throws IOException {
    	var o = new FileWriter("new-zoo.txt");
    	try (var f = new FileReader("zoo-data.txt");
    		var b = new BufferedReader(f); o) {
    			o.write(b.readLine());
    	}
    	o.write("");
    }

The method compiles. The method creates a `new-zoo.txt` file and copies the first line from `zoo-data.txt` into it. The try-with-resources statement closes all of declared resources including the `FileWriter o`. For this reason, the `Writer` is closed when the last `o.write()` is called, resulting in a `IOException` at runtime. This implementation uses the the character stream clases, which inherit from `Reader` or `Writer`.

> 17.- Assume `reader` is a valid stream that supports `mark()` and whose next characters are `PEACOCKS`. What is the expected output of the following code snippet?

    var sb = new StringBuilder();
    sb.append((char) reader.read());
    reader.mark(10);
    for (int = 0; i < 2; i++) {
    	sb.append((char) reader.read());
    	reader.skip(2);
    }
    reader.reset();
    reader.skip(0);
    sb.append((char) reader.read());
    System.out.println(sb.toString());

The code compiles without any issue. Since the `Reader` supports `mark()`, the code also run without throwing any exception. `P` is added to the `StringBuilder` first. Next, the position in the stream is marked before `E`. The `E` is added to the `StringBuilder` with `AC` being skipped. Then, the `O` is added to the `StringBuilder`, with `CK` being skipped.

The stream is then `reset()` to the position before `E`. The call to `skip(0)` does not do anything since there are no characters to skip, so `E` is added onto the `StringBuilder` in the next `read()` call. The value `PEOE` is printed.

> 18.- Suppose that you need to write data that consists of `int`, `double`, `boolean` and `String` values to a file that maintains the data types of the original data. You also want the data to be performant on large files. Which three `java.io` stream classes can be chained together to best achieve this result?

Since we need to to write primitives and `String` values, the `OutputStream` classes are appropriate. The data should be written to the file directly using the `FileOutputStream` class, buffered with the `BufferedOutputStream` class, and automatically serialized with the `ObjectOutputStream` class.

> 19.- Given the following method, which statements are correct?

    public void  copyFile(File file1, File file2) throws Exception {
    	var reader = new InputStreamReader(
    		new FileInputStream(file1));
    	try (var writer = new FileWriter(file2)) {
    		char[] buffer = new char[10];
    		while(reader.read(buffer) != -1) {
    			writer.write(buffer);
    			// n1
    		}
    	}
    }

The method compiles. Methods to read/write `byte[]` values exist in the abstract parent of all I/O stream classes.

This method correctly copies the data between some files, not all files. This implementation is not correct as the return value of `read(buffer)` is not used properly. It will only correctly copy files whose character count is a multiple of 10.

If we check `file2` on line `n1` within the file system after five iterations of the `while` loop, it may be empty as the data may not have made it to the disk yet. If the `flush()` method was called after every write, we can be sure that the data will be writen to the disk when checking `file2` on line `n1`.

This method contains a resource leak as the the `reader` stream is never closed.

> 20.- Which values when inserted into the blank independently would allow the code to compile?

    Console console = System.console();
    String color = console.readLine("Favorite color? ");
    console.____________________("Your favorite color is %s", color);

The `format()` method can be used here. `Console` includes a `format()` method that takes a `String` along with a list of arguments and writes it directly to the output stream.

The methods `writer().print` and `writer().println` can not be used here since neither of those methods take additional arguments, just `String` values.

> 21.- What are some reasons to use a character stream, such as `Reader/Writer`, over a byte stream, such as `InputStream/OutputStream`?

Character streams classes often include built-in convenience methods for working with `String` data. They also handle character encoding automatically. This two attributes make `Reader/Writer` classes more appropiate for working with `String` values.

> 22.- Which of the following fields will be `null` after an instance of the class created on line 15 (`n1`) is serialized and then deserialized using `ObjectOutputStream` and `ObjectInputStream`?

    import java.io.Serializable;
    import java.util.List;
    public class Zebra implements Serializable {
    	private transient String name = "George";
    	private static String birthPlace = "Africa";
    	private transient Integer age;
    	List<Zebra> friends = new java.util.ArrayList<>();
    	private Object stripes = new Object();
    	{ age = 10;}
    	public Zebra() {
    		this.name = "Sophia";
    	}
    	static Zebra writeAndRead(Zebra z) {
    		// Implementation omitted
    	}                                                                                    // n1
    	public static void main(String[] args) {
    		var zebra = new Zebra();
    		zebra = writeAndRead(zebra);
    	}
    }

The code compiles but throw an exception at runtime. The instance member `stripes` is of type `Object`, which is not serializable. `Object` does not implement the `Serializable` interface by default. For this reason, the `Zebra` class is not serializable, with the program throwing an exception at runtime if serialized. If `stripes` were removed from the class, the fields `name` and `age` will be null as both are marked transient.
