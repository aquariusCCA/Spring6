---
up:
  - "[[Spring6 課程描述]]"
  - "[[数据校验]]"
---
> 使用 Bean Validation 校验方式，就是如何将 Bean Validation 需要使用的javax.validation.ValidatorFactory 和 javax.validation.Validator 注入到容器中。

Spring 默认有一个实现类 LocalValidatorFactoryBean，它实现了上面 Bean Validation 中的接口，并且也实现了 org.springframework.validation.Validator 接口。

**第一步 创建配置类，配置 LocalValidatorFactoryBean**

```java
@Configuration
@ComponentScan("com.codermast.spring")
public class ValidationConfig {

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
}
```

> [!NOTE] **說明**
> 
>  - LocalValidatorFactoryBean 是 Spring 框架提供的一个实现了 ValidatorFactory 接口的类，用于创建和管理 Validator 实例。
>  - 配置 ComponentScan 扫描规则，是为了开启包扫描，便于 Spring 框架的 Bean 注入

**第二步 创建实体类，使用注解定义校验规则**

```java
import jakarta.validation.constraints.Max;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotNull;

public class User {

    @NotNull
    private String name;

    @Min(0)
    @Max(120)
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

> [!NOTE] **常用注解说明**  
	@NotNull 限制必须不为null  
	@NotEmpty 只作用于字符串类型，字符串不为空，并且长度不为0  
	@NotBlank 只作用于字符串类型，字符串不为空，并且trim()后不为空串  
	@DecimalMax(value) 限制必须为一个不大于指定值的数字  
	@DecimalMin(value) 限制必须为一个不小于指定值的数字  
	@Max(value) 限制必须为一个不大于指定值的数字  
	@Min(value) 限制必须为一个不小于指定值的数字  
	@Pattern(value) 限制必须符合指定的正则表达式  
	@Size(max,min) 限制字符长度必须在min到max之间  
	@Email 验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式

**第三步 使用两种不同的校验器实现**

**（1）使用jakarta.validation.Validator校验**

```java
import jakarta.validation.ConstraintViolation;
import jakarta.validation.Validator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.Set;

@Service
public class MyService1 {

    @Autowired
    private Validator validator;

    public boolean validator(User user){
        Set<ConstraintViolation<User>> sets =  validator.validate(user);
        return sets.isEmpty();
    }
}
```

> [!NOTE] **說明**
> 
> 在校验过程中，Validator 会依次检查用户对象的各个属性，验证其是否满足指定的约束条件。如果某个属性的值不符合约束条件，就会生成一个 ConstraintViolation 对象，其中包含了违反约束的详细信息，如属性名称、违反的约束类型、错误消息等。最后，这些 ConstraintViolation 对象被收集到一个 Set 集合中返回。

**（2）使用org.springframework.validation.Validator校验**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.validation.BindException;
import org.springframework.validation.Validator;

@Service
public class MyService2 {

    @Autowired
    private Validator validator;

    public boolean validaPersonByValidator(User user) {
        BindException bindException = new BindException(user, user.getName());
        validator.validate(user, bindException);
        return bindException.hasErrors();
    }
}
```

> [!NOTE] **說明**
> 
> BindException 是 org.springframework.validation 包中的一个类，它继承自 org.springframework.validation.Errors 接口，用于收集校验错误信息。通过创建 BindException 对象，可以将校验结果收集到其中，方便后续对错误信息的处理。

**第四步 测试**

```java
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestMethod2 {

    @Test
    public void testMyService1() {
        ApplicationContext context = new AnnotationConfigApplicationContext(ValidationConfig.class);
        MyService1 myService = context.getBean(MyService1.class);
        User user = new User();
        user.setAge(-1);
        boolean validator = myService.validator(user);
        System.out.println(validator);
    }

    @Test
    public void testMyService2() {
        ApplicationContext context = new AnnotationConfigApplicationContext(ValidationConfig.class);
        MyService2 myService = context.getBean(MyService2.class);
        User user = new User();
        user.setName("lucy");
        user.setAge(130);
        user.setAge(-1);
        boolean validator = myService.validaPersonByValidator(user);
        System.out.println(validator);
    }
}
```