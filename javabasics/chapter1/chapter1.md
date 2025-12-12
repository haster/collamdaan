## Java and the class-path
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

## Full class names and the class-path

You may have noticed that our `SimpleJavaClass` didn't have a `package` declaration.
Let's add one.

Remove all `.class` files (should only be the one in `test`) and then edit the `SimpleJavaClass.java` to look as follows:

```java
package com.example.collamdaan;

public class SimpleJavaClass
{
    public static void main(String[] args)
    {
        System.out.println("Hello world!");
    }
}
```

Now recompile the class and run it again.

```sh
$ javac SimpleJavaClass.java
$ ls
SimpleJavaClass.class  SimpleJavaClass.java  test
$ java SimpleJavaClass
Error: Could not find or load main class SimpleJavaClass
Caused by: java.lang.NoClassDefFoundError: SimpleJavaClass (wrong name: com/example/collamdaan/SimpleJavaClass)
```

That's interesting!
`java` claims it cannot find `SimpleJavaClass`, due to some kind of name mismatch.
That other name looks convincingly like the package declaration we just added to our source-file.
If we check the compiled class file with the `javap` tool, we see that the full name of our class now includes the package declaration.

```sh
$ javap SimpleJavaClass.class
Compiled from "SimpleJavaClass.java"
public class com.example.collamdaan.SimpleJavaClass {
  public com.example.collamdaan.SimpleJavaClass();
  public static void main(java.lang.String[]);
}
```

That reads `public class com.example.collamdaan.SimpleJavaClass`, so apparently the full name (what is known as the 'fully qualified class name') is now `com.example.collamdaan.SimpleJavaClass`.
So let's try to run that class.

```sh
$ java com.example.collamdaan.SimpleJavaClass
Error: Could not find or load main class com.example.collamdaan.SimpleJavaClass
Caused by: java.lang.ClassNotFoundException: com.example.collamdaan.SimpleJavaClass
```

This is a little confusing.
`java` is now claiming it can't find our `com.example.collamdaan.SimpleJavaClass`, even though it could before!
The reason for this lies in how `java` loads classes.
Classes are always loaded by their fully qualified class name, to avoid name conflicts with other classes that have the same simple name.
To make it easy to find a class, `java` expects the classfile to be located on a specific path (relative to the class-path, which defaults to 'current directory').
This specific location is simple the fully qualified class name, where every part of the package-part of the name is its own subdirectory.
So for our `com.example.collamdaan.SimpleJavaClass`, it expects to find this in a file at `com/example/collamdaan/SimpleJavaClass.class`.

Let's illustrate this by moving the class-file to such a location.

```sh
$ mkdir -p com/example/collamdaan
$ mv SimpleJavaClass.class com/example/collamdaan/
$ java com.example.collamdaan.SimpleJavaClass
Hello world!
```

There's a few reasons to load classes like this, instead of just scanning the current directory and its subdirectories:

For one, this avoids having to scan potentially thousands of files in hundreds of directories, every time `java` needs to load a class.

For another, this enables there to be two classes of the same simple name, but with different fully qualified names.
Different full names means different files on different locations.
`com.example.a.StringUtils` is located at `com/example/a/StringUtils.class`, whereas `com.example.b.StringUtils` is located at `com/example/b/StringUtils.class`.
It also means there's exactly one location where the file could be (okay, one per class path-entry, but nothing is perfect).

Let's add another class to illustrate further.
Create a file called `SecondSimpleJavaClass.java` with the following contents:

```java
package com.example.collamdaan.secondbreakfast;

public class SecondSimpleJavaClass
{
    public static void main(String[] args)
    {
        System.out.println("Hello world, greetings from SecondSimpleJavaClass!");
    }
}
```

Now compile this class and add it to the correct directory.
We will utilize the `-d` option of `javac` to specify an output directory for the compilation.
As a side-effect, this also means `javac` will create the correct set of subdirectories for the generated class-file.
So in this case, it will put the class-file in `com/example/collamdaan/secondbreakfast/`, relative to the specified output directory, creating other directories as needed.

```sh
$ javac -d . SecondSimpleJavaClass.java
$ ls com/example/collamdaan/
secondbreakfast       SimpleJavaClass.class
$ ls com/example/collamdaan/secondbreakfast/
SecondSimpleJavaClass.class
```

And now we can run the second class as easily as the first:

```sh
$ java com.example.collamdaan.secondbreakfast.SecondSimpleJavaClass
Hello world, greetings from SecondSimpleJavaClass!
```

