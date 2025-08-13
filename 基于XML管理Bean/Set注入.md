---
up:
  - "[[Spring6 課程描述]]"
  - "[[依赖注入]]"
---
> 在使用 Set 注入时，需要先创建对应属性的 Set 方法，否则无法进行注入。

# 1. 创建 Student 类

```java
public class Student {

    private Integer id;

    private String name;

    private Integer age;

    private String sex;

    public Student() {
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                '}';
    }

}
```

---

# 2. 配置 bean 时为属性赋值

创建一个新配置文件，名为 beans-di.xml ，添加如下配置

```xml
<bean id="studentOne" class="com.codermast.spring6.iocxml.bean.Student">
    <!-- property标签：通过组件类的setXxx()方法给组件对象设置属性 -->
    <!-- name属性：指定属性名（这个属性名是getXxx()、setXxx()方法定义的，和成员变量无关） -->
    <!-- value属性：指定属性值 -->
    <property name="id" value="1001"/>
    <property name="name" value="张三"/>
    <property name="age" value="23"/>
    <property name="sex" value="男"/>
</bean>
```

---

# 3. 测试

```java
@Test
public void setDiTest(){
    // 1.导入 beans-di 配置文件
    ApplicationContext ac = new ClassPathXmlApplicationContext("beans-di.xml");
    // 2. 创建 Student 对象
    Student studentOne = ac.getBean("studentOne", Student.class);
    // 3. 打印 Student 对象
    System.out.println(studentOne);
}
```

![[1755055129036_uHwEJcqk4H.png]]