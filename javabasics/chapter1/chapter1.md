Move to the correct directory for your exercise.

```sh
$ cd collamdaan_exercises/javabasics/chapter1
```

Create a file called `SimpleJavaClass.java` with the following contents:

```java
public class SimpleJavaClass
{
    public static void main(String[] args)
    {
        System.out.println("Hello world!");
    }
}
```

Now compile this class. Observe a `.class` file has been created.

```sh
$ javac SimpleJavaClass.java
$ ls
SimpleJavaClass.class  SimpleJavaClass.java
```

Now when you run this class, it will output "Hello world!".

```sh
$ java SimpleJavaClass 
Hello world!
```

Observe that we didn't specify the class-file to run, but the class to run (we said `SimpleJavaClass`, not `SimpleJavaClass.class`).
The `java` tool will (by default) start looking in the current directory for the class file that contain the requested class.
Since a class-file can only contain one class, and the name of the class-file has to correspond to the short name of the class, `java` knows what file to look for.
In this case, the requested class was named `SimpleJavaClass`, so java expected a file named `SimpleJavaClass.class` in the current directory which contained the requested class.
Let's illustrate this.

Move the class-file one directory down:

```sh
$ mkdir test
$ mv SimpleJavaClass.class test/
$ ls
SimpleJavaClass.java  test
$ java SimpleJavaClass 
Error: Could not find or load main class SimpleJavaClass
Caused by: java.lang.ClassNotFoundException: SimpleJavaClass
```

`java` says it can't find the class `SimpleJavaClass`.
If we tell it to start looking from the `test` directory, it will work.

```sh
$ java --class-path test SimpleJavaClass 
Hello world!
```

If we move the class-file back to the current directory, it will no longer work, because `java` now starts looking from the `test` directory, and that directory no longer contains a `SimpleJavaClass.class`!

```sh
$ mv test/SimpleJavaClass.class ./
$ java --class-path test SimpleJavaClass 
Error: Could not find or load main class SimpleJavaClass
Caused by: java.lang.ClassNotFoundException: SimpleJavaClass
```

A class-path, like most path-variables, can contain a list of paths (separated by ':'-characters).
When we specify a list of paths, `java` will look in these (in order) to find the matching class.
Let's try that out.

```sh
$ java --class-path test:. SimpleJavaClass 
Hello world!
```

We now specified a `class-path` consisting of both the `test`-directory, as  well as the current directory (represented by '.').
Now, `java` can find the `SimpleJavaClass.class` file, since it is located in the current directory.
But if we move it back to the `test`-directory, `java` can still find it since we specify both directories in our class-path.

```sh
$ mv SimpleJavaClass.class test/
$ java --class-path test:. SimpleJavaClass 
Hello world!
```

The current directory, '.', is the default class-path, if you don't specify anything.
