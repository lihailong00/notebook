# 重写equals和hashcode

[toc]



## 如何重写

在IDEA中，可以自动生成equals方法和hashcode方法。



## 为什么要重写equals方法

当自定义对象后，Java判断两个对象是否相同的依据是：这两个引用是否指向内存空间中的同一个对象。

**重写equals方法可以自定义规则，用于比较两个对象是否相等。**

下面的案例中，我们希望s1和s2是相等的，因为这两个对象中的属性值都分别相等。

那么此时我们需要重写equals方法。

```java
public class Student {
    private String firstName;

    private String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```



```java
import java.util.Objects;

public class Main {
    public static void main(String[] args) {
        Student s1 = new Student("li", "xiaolong");
        Student s2 = new Student("li", "xiaolong");
        if (Objects.equals(s1, s2)) {
            System.out.println("same!");
        } else {
            System.out.println("not same~");
        }
        // 输出 not same~
    }
}
```

下面是重写equals方法后的情况。当再次执行Main方法，可以得出两个对象相等。

```java
public class Student {
    private String firstName;

    private String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (!Objects.equals(firstName, student.firstName)) return false;
        return Objects.equals(lastName, student.lastName);
    }
}
```



## 重写hashcode方法

当我们将对象放入set/map之类的容器时，会先调用对象的hashcode方法，得到一个hash值，再算出该对象放在hash桶中的哪个位置。

如果我们不重写hashcode方法，就会导致两个看上去一样的对象放入不同的哈希桶中。

下面案例中，由于每个对象经过hash运算，被散列到不同的位置，没有产生hash冲突，这样也不会触发equals函数判断两个元素是否相等。因此即便重写了equals，也会导致两个看上去相同的元素都被放入set中。

```java
public class Student {
    private String firstName;

    private String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (!Objects.equals(firstName, student.firstName)) return false;
        return Objects.equals(lastName, student.lastName);
    }
}
```



```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Student> set = new HashSet<>();
        for (int i = 0; i < 100; i++) {
            Student student = new Student("li", "xiaolong");
            set.add(student);
        }
        System.out.println("set中不重复的元素个数为：" + set.size());  // 100
    }
}
```



再看看下面的例子。由于元素个数非常多，因此会产生hash冲突。产生冲突后会触发equals函数，因此set中的元素并不是10000000。

```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Student> set = new HashSet<>();
        for (int i = 0; i < 10000000; i++) {
            Student student = new Student("li", "xiaolong");
            set.add(student);
        }
        System.out.println("set中不重复的元素个数为：" + set.size());  // 9976723
    }
}
```



当我们重写hashcode后，就能让看上去相等的元素被散列到hash表中同一个位置，进而出发equals函数。

```java
import java.util.Objects;

public class Student {
    private String firstName;

    private String lastName;

    public Student(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (!Objects.equals(firstName, student.firstName)) return false;
        return Objects.equals(lastName, student.lastName);
    }

    @Override
    public int hashCode() {
        int result = firstName != null ? firstName.hashCode() : 0;
        result = 31 * result + (lastName != null ? lastName.hashCode() : 0);
        return result;
    }
}
```



```java
import java.util.HashSet;
import java.util.Set;

public class Main {
    public static void main(String[] args) {
        Set<Student> set = new HashSet<>();
        for (int i = 0; i < 10000000; i++) {
            Student student = new Student("li", "xiaolong");
            set.add(student);
        }
        System.out.println("set中不重复的元素个数为：" + set.size());  // 1
    }
}
```

