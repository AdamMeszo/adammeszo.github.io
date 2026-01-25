# Java 25

<b>[This file is available as Java Notebook as well.](./JDK25.ijnb)</b>

This is basic overview of (some) <b>final</b> features included in JDK 25. [Preview](https://docs.oracle.com/en/java/javase/22/language/preview-language-and-vm-features.html#GUID-5A82FE0E-0CA4-4F1F-B075-564874FE2823), [experimental or incubator](https://4comprehension.com/preview-experimental-and-incubating-features-in-java/) features have been intentionally left out.

It must be noted that the listed features are not the difference between JDK 25 and JDK 21 (latest LTS before 25) but between JDK 25 and JDK 24. Therefore JDK 25 in comparison to JDK 21 contains more new features than just those listed here.

Individual features are listed in specific order from the most basic to the most advanced, based on authors' opinion.

JDK 25 is LTS release of Java SE released in september 2025 with 5 years of premier and 8 years of extended support.

All .java files with "before" prefix must be compiled using Java 21 JDK. All .java files with "JEP" prefix must be compiled using Java 25 JDK.

---

## [Compact Source Files and Instance Main Methods (JEP 512)](https://openjdk.org/jeps/512)

The main goal of this feature is to make it easier to students or beginners in Java language to create simple application by omitting unnecessary keywords that are in fact not important in such use cases. Might be beneficial for quick "scripting" in Java.

### Key features
- omit 'public static' in main method declaration
	- methods called in main method declared without `static` can be declared without `static`
- omit `class X { ... } ` in .java file
- automatic import of `java.base` (using `import module java.base`, see [module imports below](#module-import-declarations-jep-511))
    - no need to `import  java.util.List;` to use the List interface
    - no need to `import java.util.Date;` to use Date interface
- `IO.println("Hello, World!");` instead of `System.out.println("Hello, World!");`
	- implementation of the `IO` based upon `System.out` and `System.in` instead of `java.  io.Console`
```java
// Compile using JDK 25
// A sample Java JDK 25 application demonstrating new features from JEP 512.
// No need to import java.util.Date and java.util.List explicitly.
import java.math.BigDecimal;

// No need to declare the class.

// No need to declare the main method as static.
void main() {
    // No need to use System.out.println; use IO.println instead.
    IO.println("Hello, World!");
    IO.println("This is a sample Java JDK 25 application.");
    BigDecimal bigDecimal = new BigDecimal("123.45");
    IO.println(bigDecimal);
    helperMethod();
    for (String item : listMethod()) {
        IO.println(item);
    }
    Date date = new Date();
    IO.println(date);
}

// No problem declaring methods without static keyword.
void helperMethod() {
    IO.println("This is a helper method.");
}

List<String> listMethod() {
    return List.of("Item1", "Item2", "Item3");
}

// Demonstrating that helper methods can't call each other if the called method was declared without static.
// private static void unusedStaticMethod() {
//     System.out.println("This is an unused static method.");
//     helperMethod();
// }
// Compile using JDK 21
// Old way of writing the same code
import java.math.BigDecimal;
import java.util.List;

public class beforeJEP512 {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
        System.out.println("This is a sample Java JDK 25 application.");
        BigDecimal bigDecimal = new BigDecimal("123.45");
        System.out.println(bigDecimal);
        helperMethod();
        for (String item : listMethod()) {
            System.out.println(item);
        }
    }

    private static List<String> listMethod() {
        // TODO Auto-generated method stub
        return List.of("Item1", "Item2", "Item3");    
    }

    private static void helperMethod() {
        System.out.println("This is a helper method.");
    }
}
```

---

## [Module Import Declarations (JEP 511)](https://openjdk.org/jeps/511)

Using module imports provides more readable way to import different classes from the same module but different package. Another advantage is that the developer does not need to know the exact package of the class.

### Key features
- instead of multiple asterisk imports such as `import java.util.*; import java.util.stream.*...` use single module import `import module java.util;`
- when importing java.sql.SQLXML using `import module java.sql.SQLXML;` no need to import both java.sql... and javax.xml... (works for other packages as well, SQLXML used just for illustration)
    - automatic on-demand import from transitive dependencies
- when importing modules that export equally named classes (java.sql.Date and java.util.Data) and using this class (e.g.: Date class), one must specify the correct package using `import java.util.Date;`
```java
// Commenting this causes and error
import java.util.Date;

import module java.base;
import module java.sql;

// In JDK 21
// import java.sql.*;
// import javax.xml.parsers.*;

public class App {

    public static void main(String[] args) throws SQLException, IOException, ParserConfigurationException, Exception {
        try (Connection con = DriverManager.getConnection("no_url"); Statement stmt = con.createStatement(
                ResultSet.TYPE_SCROLL_INSENSITIVE,
                ResultSet.CONCUR_UPDATABLE); ResultSet rs = stmt.executeQuery("SELECT a, b FROM TABLE2")) {

            if (rs.next()) {
                DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
                factory.setNamespaceAware(true);

                SQLXML updatedXml = con.createSQLXML();
                updatedXml.setString("");

                rs.updateSQLXML("b", updatedXml);
                rs.updateRow();
            }
        }

        Date date = new Date();
        List list = new ArrayList<>();
    }
}
```

---

## [Flexible Constructor Bodies (JEP 513)](https://openjdk.org/jeps/513)

This feature makes constructors more flexible by removing the neccessity of `super` and `this` calls to be declared first and also promotes "fail fast" principle.

### Key Features
- write code before `super` or `this` calls in constructors
- validate constructor parameters and fail fast
```java
class User {

    final String name;
    final String email;

    User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}

class Broker extends User {

    final String workplace;

    public Broker(String workplace, String name, String email) {
        if (StringUtils.isEmpty(name) || StringUtils.isEmpty(email)) {
            throw new IllegalArgumentException();
        }
        super(name, email);
        this.workplace = workplace;
    }
}

class StringUtils {

    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
}
```

---

# TO BE CONTINUED...

---

## Resources

Listed here are resources used during the creation of this article using no specific order.

 1. https://openjdk.org/projects/jdk/25/
 2. https://www.baeldung.com/java-25-features
 3. https://www.reddit.com/r/java/comments/1lreohw/whats_new_in_java_25_for_us_developers/
 4. https://www.jrebel.com/blog/java-25
 5. https://egahlin.github.io/2025/05/31/whats-new-in-jdk-25.html
 6. https://www.oracle.com/java/technologies/java-se-support-roadmap.html
 7. https://docs.oracle.com/en/java/javase/17/docs/api/java.base/module-summary.html