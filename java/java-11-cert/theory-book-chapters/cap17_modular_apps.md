# Cap. 17 - Modular Applications

---

> 1.- Which of the following pairs make up a service?

A service consists of the service provider interface and logic to look up implementations using a service locator. A service is composed of an interface, any classes the interface references, and a way of looking up implementations of the interface. The implementations are not part of the service.

**Note:** The service provider itself is the implementation, which is not considered as part of the service.

> 2.- A(n) ________________ module is on the classpath while a(n) _______________ module is on the module path.

The previous statemant can be completed as following:

- An unnamed module is on the classpath while an automatic module is on the module path.

- An unnamed module is on the classpath while an named module is on the module path.

Automatic modules are on the module path but do not have a `module-info` file. Named modules are on the module path and do have a `module-info` file. Unnamed modules are on the classpath.

**Note:** An unnamed module is a regular JAR that appears on the classpath. An unnamed module does not usually contain a `module-info` file. If it happens to contain one, that file will be ignored since it's on the classpath. You can think of an unnamed module as code that works the way Java worked before modules.

**Note:** A key point to remember is that code on the classpath can access the module path. By contrast, code on the module path is unable to read from the classpath.

> 3.- An automatic module name is generated if one is not supplied. Which of the following JAR filename and generated automatic module name pairs are correct?

The following pairs are correct:

- `emily-1.0.0.jar` and `emily`
- `emily-1.0.0-SNAPSHOT.jar` and `emily`
- `emily.$.jar` and `emily`

Any version information at the end of the JAR filename is removed. Underscores (_) are turned into dots(.). Other special characters like a dolar sign ($) are also turned into dots. However, adjacent dots are merged, and leading/trailing dots are removed.

> 4.- Which of the following statements are true?

The following statements are true:

- Modules with cyclic dependencies will not compile.
- A cyclic dependency always involves at least two `requires` statements.

A cyclic dependency is when a module graph forms a circle. The Java Platform Module System does not allow cyclic dependencies between modules. No such restriction exist for packages. A cyclic dependency can involve two or more modules that require each other directly or indirectly.

> 5.- Which module is available to your named module without needing a `requires` directive?

The module `java.base` is provided by default. It contains the `java.lang` package among others.

> 6.- Suppose you are creating a service provider that contains the following class. Which line of code needs to be in your `module-info.java`?

```
package dragon;
import magic.*;
public class Dragon implements Magic {
    public String getPower() {
        return "breathe fire;
    }
}
```

The line of code needed in `module-info` file is:
```
provides magic.Magic with dragon.Dragon
```

The `provides` directive takes the interface name first and implementation class name second. The `with` keywork is used.

> 7.- Which of the following modules is provided by the JDK?

The following modules are provided by the JDK:

- `java.base`
- `java.desktop`
- `java.logging`
- `jdk.compiler`

> 8.- Which of the following compiles and is equivalent to this loop?

```
List<Unicorn> all = new ArrayList<>();
for (Unicorn current : ServiceLoader.load(Unicorn.class))
    all.add(current)
```

The following code is equivalent to this loop

```
List<Unicorn> all = ServiceLoader.load(Unicorn.class)
    .stream()
    .map(Provider::get)
    .collect(Collectors.toList());
```

Note that the `stream()` method return a list of `Provider` interfaces and needs to be converted to the `Unicorn` interface we are interested in.

> 9.- Which command can you run to determine whether you have any code in your JAR file that depends on unsupported internal APIs and suggest an alternative?

The correct command is:

```
jdeps --internal-jdk
```

The `jdeps` command has an option `--internal-jdk` that lists any code using unsupported/internal APIs and prints a table with suggested alternatives.

> 10.- For a top-down migration, all modules other than named modules are ______________ modules and on the ____________________.

The correct pair is: automatic, module path

A top-down migration strategy first places all JARs on the module path. Then it migrates the top-level module to be a named module, leaving the other modules as automatic modules.

> 11.- Suppose you have separate modules for a service provider interface, service provider, service locator and consumer. If you add a second service provider module, how many of this modules do you need to recompile?

Zero modules need to be recompiled.

Since this is a new module, you only need to compile the new module. None of the existing modules needs to be recompiled. The service locator will see the new service provider simply by having a new service provider on the module path.

**Note:** Remember that a service provider is the implementation of a service provider interface.

Also, remember that Java allows only one service provider for a service provider interface in a module. If you wanted another service provider, you will need to create a separate module.

> 12.- Which of the following modules contains the `java.util` package?

This package lives in `java.base` module. The most commonly used packages are in the `java.base` module.

> 13.- Suppose you have separate modules for a service provider interface, service provider, service locator and consumer. Which are true about the directives you need to specify?

The following are true:

- The service provider interface must use the `exports` directive.
- The service provider must use the `requires` directive.
- The service provider must use the `provides` directive.

The service interface must specify `exports` for any other modules to reference it. The service provider needs access to the service provider interface. The service provider needs to declare that it provides the service.

> 14.- Suppose you have a project with one package named `magic.wand` and another project with one package named `magic.potion`. These projects have a circular dependency, so you decide to create a third project named `magic.helper`. The `magic.helper` module has the common code containing a package named `magic.util`. For simplicity, let's give each module the same name as the package. Which of the following need to appear in your `module-info` files?

The following need to appear in the `module-info` file:

- `exports magic.util;` in the magic helper project.
- `requires magic.util;` in the potion project.
- `requires magic.util;` in the wand project.

Since the new project extracts the common code, it must have an `exports` directive for  that code. The other two modules do not have to expose anything. They must have a `requires` directive to be able to use the exported code.

> 15.- Suppose you have separate modules for a service provider interface, service provider, service locator and consumer. Which module(s) need to specify a `requires` directive on the service provider?

No module needs to specify a `requires` directive on the service provider.

This question is tricky. The service provider must have a `uses` directive, but that is on the service provider interface. No modules need to specify `requires` on the service provider since that is the implementation. 

Remember that a *service provider* is the implementation of a service provider interface. The module declaration of the service provider module requires the module containing the interface as a dependency. We don't export the package that implements the interface since we don't want callers refering to it directly. Instead, we can use the `provides` directive. This allows us to specify that we provide an implementation of the interface with a specific implementation class. The syntax looks like this:

```
provides <interfaceName> with <className> 
```

By doing this, we have not exported the package containing the implementation. Instead we have made the implementation available to a service provider using the interface.

> 16.- Which are true statements about a package in a JAR on the classpath containing a `module-info` file?

The following statement is true: *"It is possible to make it available to all other modules in the classpath".*

Since the JAR is on the classpath, it is treated as a regular unnamed module even though it has a `module-info` file inside.

Remember from top-down migration strategy that modules on the module path are not allowed to refer to the classpath.

Since the classpath does not have a facility to restric packages, all packages in the unnamed module are available to all other modules in the classpath.

> 17.- Which are true statements?

The following statements are true:

- An automatic module exports all packages to named modules
- An unnamed module exports no packages to named modules.

An automatic module exports all packages. An unnamed module is not available to any modules on the module path. Therefore, it doesn't export any packages.

> 18.- Suppose you have separate modules for a service provider interface, service provider, service locator and consumer. Which statements are true about the directives you need to specify?

The following statements are true:

- The consumer must use the `requires` directive.
- The service locator must use must use the `requires` directive.
- The service locator must use the `uses` directive.

Both the consumer and the service locator depend on the service provider interface, so they need to have a `requires` directive. The service locator must specify that it `uses` the service provider interface to look it up.

> 19.- Which statement is true about `jdeps` command?

The following statements are true:

- It can provide information about dependencies on the class or package level.
- It can run against a regular JAR

The `jdeps` command provides information about the class or package level depending on the options passed. It is frequently used to determine what dependencies you will need when converting to modules. This make it useful to run against a regilar JAR.

> 20.- Suppose we have a JAR file named `cat-1.2.3-RC1.jar` and `Automatic-Module-Name` in the MANIFEST.MF is set to `dog`. What should an unnamed module referencing this automatic module include in the `module-info.java`.

Nothing. This is a trick question. An unnamed module does not have a `module-info` file. An unnamed module can access an automatic module since code in the classpath can access code in the module path. The unnamed module would simply treat the automatic module as a regular JAR without involving the `module-info` file.