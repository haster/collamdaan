## Structuring source and target directories

In the previous chapter, we learned how class-files needed to be placed in a directory corresponding to their fully qualified name.
Among other things, this enables us to have multiple classes with the same simple name (but in different packages).

Let's actually create this.
We'll create two `StringUtils`-classes with different functions.
Classes which offer generic utility, such as these, typically have very similar names.
`List`, `Constants`, `Result`, various `-Util` classes, all have generic names which can all be correct for their usage, but without packages we could not have them in the same project.

In order to seperate the source files, we'll use the same directory-structure as for the class-files.

`com/example/collamdaan/a/StringUtils.java`
```java
package com.example.collamdaan.a;

public class StringUtils
{
    public boolean isEmptyOrNull(String s)
    {
        return s == null || s.isEmpty();
    }
}
```

`com/example/collamdaan/b/StringUtils.java`
```java
package com.example.collamdaan.b;

public class StringUtils
{
    public boolean isUuid(String s)
    {
        return s != null && s.length() == 36 && s.matches("[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}");
    }
}
```

Now, we will compile both files at once in a way that places them in the correct directories.

```sh
$ javac -d . com/example/collamdaan/a/StringUtils.java com/example/collamdaan/b/StringUtils.java
$ ls com/example/collamdaan/a
StringUtils.class  StringUtils.java
$ ls com/example/collamdaan/b
StringUtils.class  StringUtils.java
```

We have two classes with the same simple name, but different full names.
They get compiled to class files in different locations, corresponding to their full names.
Since we keep the source files in the same package-based location, we don't have any name conflicts between the files.

Now if we want to remove the class files so we can recompile the source files, we have to specify each individual files:

```sh
$ rm com/example/collamdaan/a/StringUtils.class com/example/collamdaan/b/StringUtils.class
$ javac -d . com/example/collamdaan/a/StringUtils.java com/example/collamdaan/b/StringUtils.java
```

We can make life a little easier on us by separating the source and the class files.
Make a folder `source` and a folder `target` and move the source files, including their directory structure.
It should look like this:

```
source/
  |- com/
    |- example/
      |- collamdaan/
        |- a/
          |- StringUtils.java
        |- b/
          |- StringUtils.java
target/
```

Now if we compile the source files:

```sh
$ javac -d target/ source/com/example/collamdaan/a/StringUtils.java source/com/example/collamdaan/b/StringUtils.java
```

And now we have the following:

```
source/
  |- com/
    |- example/
      |- collamdaan/
        |- a/
          |- StringUtils.java
        |- b/
          |- StringUtils.java
target/
  |- com/
    |- example/
      |- collamdaan/
        |- a/
          |- StringUtils.class
        |- b/
          |- StringUtils.class
```

If we now want to delete all class files and recompile the source files, we can just clear the `target` directory.

```sh
$ rm -rf target/*
$ javac -d target/ source/com/example/collamdaan/a/StringUtils.java source/com/example/collamdaan/b/StringUtils.java
```

(Note that we don't necessarily need to remove class files before recompiling, but doing so ensures no old class files remain behind in case we renamed or removed a class in a source file.)

Apart from the ease of clearing all compiled class files, this also enables us to quickly gather all class files.
We can for instance zip the target folder and hand the zip file to someone else.

It also makes it easy to upload only the source files to a version control system such as github.
We could simply tell git to ignore the target folder (by adding it to `.gitignore`) to ensure no class files get added accidentally.

