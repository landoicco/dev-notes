

> 1.- What is the output of the following code?
```
4: var path = Path.of("/user/./root", "../kodiacbear.txt");
5: path.normalize().relativize("/lion");
6: System.out.println(path);
```

This code does not compile. The `relativize()` method takes a Path value, not a String. For this reason line 5 does not compile. If line 5 was corrected to use a Path value, then the code would compile, but it would print the value of the Path created on line 4.

Since Path is immutable, the operations on line 5 are not saved anywhere. If the value on line 5 was assigned to path and printed on line 6, then `../../lion `would be printed.

**Note:** The method `normalize()` removes path symbols that are unnecessary. The `normalize()` method does not remove all of the path symbols; only the ones that can be reduced.

**Note:** The method `relativize()` create a relative path from one Path to another. This method takes a Path as a parameter.

> 2.- For which values of `path` sent to this method would it be possible for the following code to output Success?

```
public void removeBadFile(Path path) {
	if(Files.isDirectory(path))
		System.out.println(Files.deleteIfExists(path) ? "Success": "Try Again");
}
```

This code does not compile. The reason is that the method `Files.deleteIfExists()` declares a checked `IOException` that needs to be handled or declared. If the method was corrected to declare the appropriate exceptions, adding a `path` that refers to an empty directory on the file system, `Success` would be printed.

Adding a `path` value that refers to a symbolic link pointing to an empty directory in the file system would also print `Success`. 

In both cases, if the directory is not empty, a `DirectoryNotEmptyException` will be thrown at runtime. *To delete a directory, it must be empty*.

**Note:** Most `Files` methods declare `IOException`, especially the ones that modify a file or directrory.

>3.- What is the result of executing the following code?

```
4: var p = Paths.get("sloth.schedule");
5: var a = Files.readAttributes(p, BasicFileAttributes.class);
6: Files.mkdir(p.resolve(".backup"));
7: if(a.size() > 0 && a.isDirectory()) {
8:    a.setTimes(null, null, null);
9: }
```

The code will not compile because of lines 6 and 8. The method to create a directory in the `Files` class is `createDirectory()`, not `mkdir()`. For this reason line 6 does not compile.

The `setTimes()` method is available only on `BasicFileAttributeView`, not the read-only `BasicFileAttributes`, so line 8 will not compile.

> 4.- If the current working directory is `/user/home`, then what is the output of the following code?

```
var p = Paths.get("/zoo/animals/bear/koala/food.txt");
System.out.println(p.subpath(1, 3).getName(1).toAbsolutePath());
```

The code compiles. The `subpath()` method is applied to the absoulte path, which returns the relative path `animals/bear`. Next, the `getName()` method is applied to the relative path `animals/bear`, and since this is indexed from zero, it returns the relative path `bear`.

Finally, the `toAbsolutePath()` method is applied to the relative path `bear`, resulting in the current directory `/user/home` being incorpored to the path. The final output is `/user/home/bear`.

**Note:** For the method

```
public Path subpath(int beginIndex, int endIndex)
```

the references are inclusive for `beginIndex` and exclusive of the `endIndex`. This method starts counting from `0`, just as `getName()`.

**Note:** The method `toAbsolutePath()`, converts a relative `Path` object to an absolute `Path` object by joining it to the current working directory. If the `Path` is already absolute, then the method just return the `Path` object.

> 5.- Assume `/kang` exists as a symbolic link to the directory `/mammal/kangaroo` within the file system. Which of the following statements are correct about this code snippet?

```
var path = Paths.get("/kang");
if (Files.isDirectory(path) && Files.isSymbolicLink(path))
	Files.createDirectory(path.resolve("joey"));
```

The code snippet will attempt to create a directory if the target of the symbolic link exists and is a directory. If the directory already exists, though, it will throw an exception. In short terms, "a new directory may be created" when running this code.

The new directory will be created in `/mammal/kangaroo/joey`, and also be reachable at `/kang/joey` because of the symbolic link.

**Note:** The `createDirectory()` method will create a directory and throw an exception if it already exists or the paths leading up to the directory do not exists.

**Note:** The `resolve()` method helps to concatenate paths in a similar manner as we concatenate strings. The object on which `resolve()` method is invoked becomes the basis of the new `Path` object, with the input argument being appended onto the `Path`.

> 6.- Assume that the directory `/animals` exists and is empty. What is the result of executing the following code?

```
Path path = Path.of("/animals");
try (var z = Files.walk(path)) {
	boolean b = z
		.filter((p,a) -> a.isDirectory() && !path.equals(p))  // x
		.findFirst().isPresent();  // y
	System.out.print(b ? "No Sub": "Sub");
}
```

This code does not compile. First, the `filter()` operation applied to a `Stream<Path>` takes only one parameter, not two, so the code does not compile because of line x. 

**Note:** The `filter()` method returns a `Stream` with elements that match a given expression. Here the method signature

```
Stream<T> filter(Predicate<? super T> predicate) 
```

> 7.- If the current working directory is `/zoo` and the path `/zoo/turkey` does not exists, then what is the result of the following code?

```
Path path = Paths.get("turkey");
if (Files.isSameFile(path, Paths.get("/zoo/turkey")))  // z1
	Files.createDirectories(path.resolve("info"));   // z2
```

The code compiles but throws an exception at runtime. First, the method `Files.isSameFile()` first checks to see whether the `Path` values are the same in terms of `equals()`. Since the first path is relative and the second path is absolute, this comparison will return `false`, forcing `isSameFile()` to check for the existence of both paths in the file system. Since `/zoo/turkey` does not exists, a `NoSuchFileException` is thrown.

**Note:** The method

```
public static boolean isSameFile(Path path, Path path2) throws IOException
```

throws a checked `IOException` exception. In most usages of `isSameFile()`, an exception will be thrown if the path does not exists. 

There is a special case in which it does not. If the two path objects are equal, in terms of `equals()`, then the method will just return `true` without checking whether the file exists. 

**Note:** This `isSameFile()` method does not compare the contents of the files. Two files may have identical names, content and attributes, but if they are different locations, then this method will return `false`.

> 8.- Which of the following correctly create `Path` instances?

The following are valid ways of creating a `Path` instance:

* `FileSystems.getDefault().getPath("puma.txt")` First, the class `FileSystems` include factory methods for file systems. The `getDefault()` method get the default file system. The working directory of the file system is the current user directory, named by the system property `user.dir`. This allows for interoperability with the `java.io.File` class.  That's why we can call the `getPath()` method.
* `new java.io.File("tiger.txt").toPath()` This call will give us a instance of `File` for the file `tiger.txt`. And from that file instance, we can call the `getPath()` method to get the string form form of this abstract pathname.
* `Path.of(Path.of(".").toUri())` First, we can call the method `toUri()` from a `Path` instance. And since `Path.of()` have an overloaded static method that takes an URI instead of a string, this code is valid.

> 9.- What is the output of the following code?

```
var path1 = Path.of("/pets/../cat.txt");
var path2 = Paths.get("./dog.txt");
System.out.println(path1.resolve(path2));
System.out.println(path2.resolve(path1));
```

This code compiles and runs without any issue, and the output will be:

```
/pets/../cat.txt/./dog.txt
/pets/../cat.txt
```

First, remember that the `resolve()` method does not normalize any path symbols. It just concatenates the second path to the first one. Also, we know that calling `resolve()` with an absolute path as a parameter return that same absolute path. Since `path1` is absolute, we get that same path on our second call to `resolve()`.

> 10.- What are some advantages of using `Files.lines()` over `Files.readAllLines()`?

First, lets review the definition for both methods.

```
public static Stream<String> lines(Path path) throws IOException
```

```
public static List<String> readAllLines(Path path) throws IOException
```

With the first method, `Files.lines()`, the contents of the file are read and processed lazily, which means that only a small portion of the file is stored in memory at any given time. *For this reason, `Files.lines()` works better on large files with limited memory available.*

The second method, `Files.readAllLines()`, read the entire file, with the resulting `List<String>` storing all of the contents of the file in memory at once. If the file is significantly large, then a `OutOfMemoryError` may be triggered trying to load all of it into memory.

Although a `List` can be converted to a stream, this require an extra step. *For this reason `Files.lines()` is useful since the returned `Stream<String>` can be chained with functional programming methods like `filter()` and `map()` directly.*

> 11.- Assume `monkey.txt` is a file that exists in the current working directory. Which statements about the following code snippet are correct?

```
Files.move(Path.of("monkey.txt"), Paths.get("/animals"),
	StandardCopyOption.ATOMIC_MOVE,
	LinkOption.NOFOLLOW_LINKS);
```

First, lets take a look at the method definition:

```
public static Path move(Path source, Path target), 
	CopyOption... options) throws IOException
```

The `NOFOLLOW_LINKS` option means that if the source is a symbolic link, the link itself and not the target will be copied at runtime. The option `ATOMIC_MOVE` means that *any process monitoring the file system will not see an incomplete file during the move.*

> 12.- What are some advantages of NIO.2 over the legacy `java.io.File` class for working with files.  // Adddd info

NIO.2 has several advantages over `File`. Some of those are the following:

* NIO.2 supports file system-dependent attributes.
* NIO.2 includes a method to traverse a directory tree.
* NIO.2 includes methods that are aware of symbolic links.

> 13.- For the `copy()` method shown here, assume that the source exists as a regular file and that the target does not. What is the result of the following code?

```
var p1 = Path.of(".", "/", "goat.txt").normalize();  // k1
var p2 = Path.of("mule.png");
Files.copy(p1, p2, StandardCopyOption.COPY_ATTRIBUTES);   // k2
System.out.println(Files.isSameFile(p1, p2));
```

This code compiles without any issue and it will output `false`. Even though the file is copied with attributes preserved, the file is considered a separate file, so the output is `false`.

**Note:** The method `isSameFile()` returns `true` only if the files pointed to in the file system are the same, without regard to the file contents.

> 14.- Assume `/monkeys` exists as a directory containing multiple files, symbolic links, and sub-directories. Which statement about the following code is correct?

```
var f = Path.of("/monkeys");
try (var m = 
	Files.find(f, 0, (p,a) -> a.isSymbolicLink())) {  // y1
		m.map(s -> s.toString())
			.collect(Collectors.toList())
			.stream()
			.filter(s -> s.toString().endsWith(".txt"))  // y2
			.forEach(System.out::println);	
}
```

The code compiles, runs without issue and prints nothing. First, let's take a look at the method definition:

```
public static Stream<Path> find(Path start, 
	int maxDepth,
	BiPredicate<Path, BasicFileAttributes> matcher,
	FileVisitOption... options) throws IOException
```

Where `maxDepth` is the depth limit. A depth value of 0 indicates the current path itself.

In this case, `maxDepth = 0`, meaning the only record that will be searched is the top-level directory (the current path itself). Since we know that the top directory is a directory and not a symbolic link, no other paths will be visited, and nothing will be printed.

> 15.- Which NIO.2 method is most similar to the legacy `java.io.File` method `listFiles`?

Let's remember that `java.io.File` method `listFiles()` retrieves the members of the current directory without traversing any subdirectories. The most similar method in NIO.2 that does this is `Files.list()` as return a `Stream<Path>` of a single directory.

> 16.- What are some advantages of using NIO.2's `Files.readAttributes()` method rather that reading attributes individually from a file? 

Whether a path is a symbolic link, file, or directory is not relevant. Using a view to read multiple attributes leads to fewer round-trips between the process and the file system and better performance. For reading single attributes, there is little or no expected gain.

Finally, views can be used to access file system-specific attributes that are not available in `Files` methods.

> 17.- Assuming the `/fox/food-schedule.csv` file exists with the specified contents, what is the expected output of calling `printData()` on it?

**/fox/food-schedule.csv**
```
6am,Breakfast
9am,SecondBreakfast
12pm,Lunch
6pm,Dinner
```

```
void printData(Path path) throws IOException {
	Files.readAllLines(path)  // r1
		.flatMap(p -> Stream.of(p.split(",")))  // r2
		.map(q -> q.toUpperCase())  // r3
		.forEach(System.out::println);
}
```

The code does not compile because of line `r2`. The `readAllLines()` method returns a `List`, not a `Stream`. Therefore, the call to `flatMap()` is invalid.

**Note:** If the `Files.lines()` method were instead used, it would print the contents of the file one capitalized word at a time, with commas removed. Remember that `Files.lines()` method does return a `Stream`, so it will be useful in this case.

> 18 .- What are some possible results of executing the following code?

```
var x = Path.of("/animals/fluffy/..");
Files.walk(x.toRealPath().getParent())  // u1
	.map(p -> p.toAbsolutePath().toString())  // u2
	.filter(s -> s.endsWith(".java"))  // u3
	.collect(Collectors.toList())
	.forEach(System.out::println);
```

The code compiles without issue. The `toRealPath()` method will simplify the path to `/animals` and *throw an exception if it does not exists*.

If the path does exists, calling `getParent()` on it returns the root directory. Walking the root directory with the filter expression will *print all `.java` files in the root directory along with all `.java` files in the directory tree*.

**Note:** The method

```
public Path toRealPath(LinkOption... options) throws IOException
```

is similar to `normalize()`, in that it eliminates any redundant path symbols. It is also similar to `toAbsolutePath()`, in that it will join the path with the current working directory if the path is relative. 

Unlike those methods, though, `toRealPath()` will throw an exception if the path does not exists. In addition, it will follow symbolic links, with an optional varargs parameter to ignore them.

**Note:** The method 

```
public static Stream<Path> walk(Path start,
	FileVisitOption... options) throws IOException
```

```
public static Stream<Path> walk(Path start, int maxDepth,
	FileVisitOption... options) throws IOException
```

can be used to walk the directory tree using a depth first-search. The `walk()` method is different in that it does not follow symbolic links by default and requires the `FOLLOW_LINKS` options to be enabled.

**Note:** A depth-first search traverses the structure from the root to an arbitrary leaf and then navigates back up towards the root, traversing fully down any paths it skipped along the way.

> 19.- Assuming the directories and files referenced exists and are not symbolic links, what is the result of executing the following code?

```
var p1 = Path.of("/lizard", ".")
	.resolve(Path.of("walking.txt"));
var p2 = new File("/lizard/././actions/../walking.txt")
	.toPath();
System.out.print(Files.isSameFile(p1, p2));
System.out.print(" ");
System.out.print(p1.equals(p2));
System.out.print(" ");
System.out.print(p1.normalize().equals(p2.normalize()));
```

The code compiles and runs without issue. If you simplify the redundant path symbols, then `p1` and `p2` represent the same path, `/lizard/walking.txt`. Therefore, `isSameFile()` return `true`.

The second output is `false`, because `equals()` checks only if the paths values are the same, without reducing the path symbols. Finally, the normalized paths are the same, since all extra symbols have been removed, so the last line outputs `true`. For this reason, the output of this code is

```
true false true
```

**Note:** The method

```
public static boolean isSameFile(Path path, Path path2) throws IOException
```

takes two `Path` objects as input, resolves all path symbols, and follows symbolic links.

While most usages of `isSameFile()` will trigger an exception if the paths do not exists, there is a special case in which it does not. If the two path objects are equal, in terms of `equals()`, then the method will just return `true` without checking whether the file exists. 

> 20.- Assuming the current directory is `/seals/harp/food`, what is the result of executing the following code?

```
final Path path = Paths.get(".").normalize();
int count = 0;
for(int i = 0; i<path.getNameCount(); i++) {
	count++;
}
System.out.println(count);
```

This code compiles and prints `1`. The `normalize()` method does not convert a relative path into an absolute path; therefore, the path value after the first line is just the current directory symbol. The `for()` loop iterates the name values, but since there is only one entry, the loop terminates after a single iteration.

**Note:** The `Path` method

```
int getNameCount()
```

returns the number of elements in the path, or `0` if this path only represents a root component.

> 21.- Assume the `source` instance passed to the following method represents a file that exists. Also, assume `/flip/sounds.txt` exists as a file prior to executing this method. When this method is executed, which statement correctly copies the file to the path specified by `/flip/sounds.txt`?

```
void copyIntoFlipDirectory(Path source) throws IOException {
	var dolphinDir = Path.of("/flip");
	dolphinDir = Files.createDirectories(dolphinDir);
	var n = Paths.get("sounds.txt");
	________________________;
}
```

The code compiles without issues. To correctly copy the file to the path specified by `/flip/sounds.txt`, we can use the statement:

```
Files.copy(source, dolphinDir.resolve(n),
	StandardCopyOption.REPLACE_EXISTING)
```

The `copy()` method takes a target that is the path to the new file location, not the directory to be copied into. Therefore, the target path should be `/flip/sounds.txt`, not `/flip`. For this reason, we need to use the `resolve()` method, to concatenate the paths an get `/flip/sounds.txt`.

Since the question says the file already exists, the `REPLACE_EXISTING` option must be specified or an exception will be thrown at runtime.

**Note:** The method

```
public static Path copy(Path source, Path target, 
	CopyOption... options) throws IOException
```

copies a file or directory from one location to another using Path objects. When directories are copied, the copy is shallow.

A shallow copy means that the files and subdirectories within the directory are not copied. A deep copy means that the entire tree is copied, including all of its contents and subdirectories.

> 22.- Assuming the path referenced by `m` exists as a file, which statements about the following method are correct?

```
void duplicateFile(Path m, Path x) throws Exception {
	var r = Files.newBufferedReader(m);
	var w = Files.newBufferedWriter(x, 
		StandardOpenOption.APPEND);
	String currentLine = null;
	while ((currentLine = r.readLine()) != null)
		w.write(currentLine);
}
```

The code compiles without any issue. The method copies the contents of a file, but it removes all line breaks. The `while()` loop would need to include a call to `w.newLine()` to correctly copy the file. The `APPEND` option creates the file if it does not exist; otherwise, it starts writing from the end of the file.

For this example, the following statements are correct:

- If the path referenced by `x` does not exist, then a new file will be created.
- The method contains a resource leak.

The method contains a resource leak because the resources created in the method are not closed or declared inside a try-with-resources statement.