---
up:
  - "[[Spring6 課程描述]]"
---
# 创建注解

### 创建Bean注解

```java
// 此注解用于定义容器对象，类似于@Componet、@Service、@Controller注解等作用
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Bean {
}
```

### 创建依赖注入注解

```java
// 此注解用于完成成员属性的依赖注入，类似于@Autoware、@Resource注解等作用
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Di {
}
```

---

# 创建IOC核心逻辑

### 定义bean容器接口

```java
public interface ApplicationContext {

    Object getBean(Class clazz);
}
```

### 编写注解bean容器接口实现

```java
public class AnnotationApplicationContext implements ApplicationContext {

    //存储bean的容器
    private HashMap<Class, Object> beanFactory = new HashMap<>();
    private static String rootPath;

    @Override
    public Object getBean(Class clazz) {
        return beanFactory.get(clazz);
    }

    /**
     * 根据包扫描加载bean
     *
     * @param basePackage
     */
    public AnnotationApplicationContext(String basePackage) {
        // 这里要注意的是如果在 linux 或 mac 平台下，文件的绝对路径使用的是 / 斜杠
        // 而 windows 平台下是 反斜杠 \
        // 如果使用的错误的符号，那么就会导致 dirs 为空
        // windows 使用这个
        // String packageDirName = basePackage.replaceAll("\\.", "\\\\");
        try {
            String packageDirName = basePackage.replaceAll("\\.", "/");
            Enumeration<URL> dirs = Thread.currentThread().getContextClassLoader().getResources(packageDirName);
            while (dirs.hasMoreElements()) {
                URL url = dirs.nextElement();
                String filePath = URLDecoder.decode(url.getFile(), StandardCharsets.UTF_8);
                rootPath = filePath.substring(0, filePath.length() - packageDirName.length());
                loadBean(new File(filePath));
            }

        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        //依赖注入
        loadDi();
    }

    private void loadBean(File fileParent) {
        if (fileParent.isDirectory()) {
            File[] childrenFiles = fileParent.listFiles();
            if (childrenFiles == null || childrenFiles.length == 0) {
                return;
            }
            for (File child : childrenFiles) {
                if (child.isDirectory()) {
                    //如果是个文件夹就继续调用该方法,使用了递归
                    loadBean(child);
                } else {
                    //通过文件路径转变成全类名,第一步把绝对路径部分去掉
                    String pathWithClass = child.getAbsolutePath().substring(rootPath.length() - 1);
                    //选中class文件
                    if (pathWithClass.contains(".class")) {
                        //    com.codermast.dao.UserDao
                        //去掉.class后缀，并且把 \ 替换成 .

                        // 这里和上面同理，还是 windows 和 linux Mac 平台下的文件路径分隔符不一致的问题
                        // String fullName = pathWithClass.replaceAll("\\\\", ".").replace(".class", "");

                        String fullName = pathWithClass.replaceAll("/", ".").replace(".class", "").substring(1);
                        try {
                            Class<?> aClass = Class.forName(fullName);
                            //把非接口的类实例化放在map中
                            if (!aClass.isInterface()) {
                                Bean annotation = aClass.getAnnotation(Bean.class);
                                if (annotation != null) {
                                    Object instance = aClass.newInstance();
                                    //判断一下有没有接口
                                    if (aClass.getInterfaces().length > 0) {
                                        //如果有接口把接口的class当成key，实例对象当成value
                                        System.out.println("正在加载【" + aClass.getInterfaces()[0] + "】,实例对象是：" + instance.getClass().getName());
                                        beanFactory.put(aClass.getInterfaces()[0], instance);
                                    } else {
                                        //如果有接口把自己的class当成key，实例对象当成value
                                        System.out.println("正在加载【" + aClass.getName() + "】,实例对象是：" + instance.getClass().getName());
                                        beanFactory.put(aClass, instance);
                                    }
                                }
                            }
                        } catch (ClassNotFoundException | IllegalAccessException | InstantiationException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }

    private void loadDi() {
        for (Map.Entry<Class, Object> entry : beanFactory.entrySet()) {
            //就是咱们放在容器的对象
            Object obj = entry.getValue();
            Class<?> aClass = obj.getClass();
            Field[] declaredFields = aClass.getDeclaredFields();
            for (Field field : declaredFields) {
                Di annotation = field.getAnnotation(Di.class);
                if (annotation != null) {
                    field.setAccessible(true);
                    try {
                        System.out.println("正在给【" + obj.getClass().getName() + "】属性【" + field.getName() + "】注入值【" + beanFactory.get(field.getType()).getClass().getName() + "】");
                        field.set(obj, beanFactory.get(field.getType()));
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```


---

# 创建Dao层

### 创建UserDao接口

```java
public interface UserDao {

    public void print();
}
```

### 创建UserDaoImpl实现

```java
@Bean
public class UserDaoImpl implements UserDao {
    @Override
    public void print() {
        System.out.println("Dao层执行结束");
    }
}
```

### 說明

- 当此类实现了 UserDao 这个接口时，会将 UserDao 这个接口作为容器的 Key 值，来获取UserDaoImpl 这个实现类
    
- 这也就是为什么一个接口不能对应多个实现类，因为 HashMap 的 Key 值不能重复

```java
// 参数解释
// 参数1：实现类上实现的第一个接口或者实现类的字节码文件
// 参数2：实现类实例
beanFactory.put(aClass.getInterfaces()[0], instance);
// 此处，参数1：接口为UserDao；参数2，实现类实例为UserDaoImpl
```

---

# 创建Service层

### 创建UserService接口

```java
public interface UserService {
    public void out();
}
```

### 创建UserServiceImpl实现类

```java
@Bean
public class UserServiceImpl implements UserService {

    @DI
    private UserDao userDao;

    @Override
    public void out() {
        userDao.print();
        System.out.println("Service层执行结束");
    }
}
```

### 说明：

当此类里包含了依赖注入这个接口时，会将 UserServiceImpl 这个实现类实例作为依赖注入的 Key值，并且根据依赖注入的变量名确定所需要依赖注入的实现类实例

```java
// 参数解释
// 参数1：设置字段值的对象实例
// 参数2：需要设置的字段值
field.set(obj, beanFactory.get(field.getType()));
//此处，参数1：实现类实例UserServiceImpl；参数2：实现类实例UserDaoImpl
```

---

# 执行测试

```java
@Test
public void testIoc() {
    ApplicationContext applicationContext = new AnnotationApplicationContext("com.codermast.reflect");
    UserService userService = (UserService)applicationContext.getBean(UserService.class);
    userService.out();
    System.out.println("run success");
}
```