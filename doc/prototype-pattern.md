## 原型模式

原型模式，用原型实例指定创建对象的种类，并且通过拷贝这些原型，创建新的对象。  
原型模式是一种创建型设计模式，其实就是从一个对象再创建另外一个可定制的对象，而且不需要知道任何创建的细节。

**示例**

有一个简历类，必须要有姓名，可以设置性别和年龄，也可以设置工作经历。最后复制三份相同简历。

```java
/**
 * Resume
 */
public class Resume {

    private String name;

    private String sex;

    private String age;

    private String timeArea;

    private String company;

    public Resume(String name) {
        this.name = name;
    }

    public void setPersonInfo(String sex, String age) {
        this.sex = sex;
        this.age = age;
    }

    public void setWorkExperience(String timeArea, String company) {
        this.timeArea = timeArea;
        this.company = company;
    }

    @Override
    public String toString() {
        return "Resume{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", age='" + age + '\'' +
                ", timeArea='" + timeArea + '\'' +
                ", company='" + company + '\'' +
                '}';
    }
}

/**
 * Main
 */
public class Main {

    public static void main(String[] args) {
        Resume a = new Resume("God.Gao");
        a.setPersonInfo("woman", "20");
        a.setWorkExperience("2014-2018", "XXXCompany");

        Resume b = new Resume("God.Gao");
        b.setPersonInfo("woman", "20");
        b.setWorkExperience("2014-2018", "XXXCompany");

        Resume c = new Resume("God.Gao");
        c.setPersonInfo("woman", "20");
        c.setWorkExperience("2014-2018", "XXXCompany");

        System.out.println(a.toString());
        System.out.println(b.toString());
        System.out.println(c.toString());
    }
}
```
此写法比较好理解，简单易操作。但是，假如需要二十份相同简历，就需要二十次实例化，假如需要修改其中一个属性的值，那就要修改二十次，这样就很麻烦，效率也低。

**使用原型模式改进**

Java 中 Object 类是所有类的基类，Object 类提供了一个 clone() 方法，该方法可以将一个 Java 对象复制一份，但是需要实现 clone() 的 Java 类必须要实现一个 Cloneable 接口，这样就可以完成原型模式了。

```java
/**
 * Resume
 */
public class Resume implements Cloneable {

    private String name;

    private String sex;

    private String age;

    private String timeArea;

    private String company;

    public Resume(String name) {
        this.name = name;
    }

    public void setPersonInfo(String sex, String age) {
        this.sex = sex;
        this.age = age;
    }

    public void setWorkExperience(String timeArea, String company) {
        this.timeArea = timeArea;
        this.company = company;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "Resume{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", age='" + age + '\'' +
                ", timeArea='" + timeArea + '\'' +
                ", company='" + company + '\'' +
                '}';
    }
}

/**
 * Main
 */
public class Main {

    public static void main(String[] args) throws CloneNotSupportedException {
        Resume a = new Resume("God.Gao");
        a.setPersonInfo("woman", "20");
        a.setWorkExperience("2014-2018", "XXXCompany");

        Resume b = (Resume) a.clone();

        Resume c = (Resume) a.clone();

        System.out.println(a.toString());
        System.out.println(b.toString());
        System.out.println(c.toString());
    }
}

```



