# 深入理解HashMap存取过程

总结：remove的判断依据：先算出元素的hash值，再去哈希桶对应的下表查找值。如果

案例一：

```java
import java.util.HashSet;

public class Main {
    public static void main(String[] args) {
        HashSet<Person> set = new HashSet<>();
        Person p1 = new Person(1);
        Person p2 = new Person(2);
        set.add(p1);
        set.add(p2);
        System.out.println(set);  // 1,2

        p1.setId(11);
        set.remove(p1);
        System.out.println(set);  // 11,2

        Person p3 = new Person(3);
        set.add(p3);
        System.out.println(set);  // 11,2,3

        Person p4 = new Person(11);
        set.add(p4);
        System.out.println(set);  // 11,2,3,11

        Person p5 = new Person(11);  // 11,2,3,11
        set.add(p5);
        System.out.println(set);

        Person p6 = new Person(1);  // 11,1,2,3,11
        set.add(p6);
        System.out.println(set);
    }
}

class Person {
    private int id;

    public Person(int id) {
        this.id = id;
    }

    public int getId() {
        return this.id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof Person)) {
            return false;
        }

        Person person = (Person) o;
        return person.getId() == this.id;
    }

    @Override
    public int hashCode() {
        return id;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                '}';
    }
}
```



```java
import java.util.HashSet;

public class Main {
    public static void main(String[] args) {
        HashSet<Person> set = new HashSet<>();
        Person p1 = new Person(1);
        Person p2 = new Person(2);
        set.add(p1);
        set.add(p2);
        System.out.println(set);  // 1,2

        p1.setId(17);  // 这里发生了变化
        set.remove(p1);
        System.out.println(set);  // 17,2

        Person p3 = new Person(3);
        set.add(p3);
        System.out.println(set);  // 17,2,3

        Person p4 = new Person(11);
        set.add(p4);
        System.out.println(set);  // 17,2,3,11
    }
}

class Person {
    private int id;

    public Person(int id) {
        this.id = id;
    }

    public int getId() {
        return this.id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof Person)) {
            return false;
        }

        Person person = (Person) o;
        return person.getId() == this.id;
    }

    @Overrideimport java.util.HashSet;

public class Main {
    public static void main(String[] args) {
        HashSet<Person> set = new HashSet<>();
        Person p1 = new Person(1);
        Person p2 = new Person(2);
        set.add(p1);
        set.add(p2);
        System.out.println(set);  // 1,2

        p1.setId(17);
        Person p3 = new Person();
        set.add(p3);
        System.out.println(set);  // 2
    }
}

class Person {
    private int id;

    public Person(int id) {
        this.id = id;
    }

    public int getId() {
        return this.id;
    }

    public void setId(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof Person)) {
            return false;
        }

        Person person = (Person) o;
        return person.getId() == this.id;
    }

    @Override
    public int hashCode() {
        return id;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                '}';
    }
}
    public int hashCode() {
        return id % 16;  // 这里发生了变化
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                '}';
    }
}
```



```java
```

