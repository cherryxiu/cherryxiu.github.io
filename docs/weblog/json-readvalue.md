## 将字符串 JSON 映射为实体类

### 背景

java 模拟接口返回数据时，总是需要将字符串数据映射到实体类中，方便后续处理

### 1. 添加依赖

```
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.8</version>
</dependency>
```

或者使用 ` Jackson`

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>
```

### 2. 定义实体类

```
{
    "name": "John Doe",
    "age": 30,
    "email": "john.doe@example.com"
}

@Data
public class User {
    private String name;
    private int age;
    private String email;
}

```

### 3.将 JSON 字符串映射为实体类

#### 使用 `Gson`

```java
import com.google.gson.Gson;

public class Main {
    public static void main(String[] args) {
        String json = "{\"name\":\"John Doe\",\"age\":30,\"email\":\"john.doe@example.com\"}";
        Gson gson = new Gson();
        User user = gson.fromJson(json, User.class);
        System.out.println(user.getName());
        System.out.println(user.getAge());
        System.out.println(user.getEmail());
    }
}
```

#### 使用 `Jackson`

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class Main {
    public static void main(String[] args) {
        String json = "{\"name\":\"John Doe\",\"age\":30,\"email\":\"john.doe@example.com\"}";
        ObjectMapper objectMapper = new ObjectMapper();
        // 配置忽略未知属性, 如果json中有字段不在实体类中，可以忽略
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        try {
            User user = objectMapper.readValue(json, User.class);
            System.out.println(user.getName());
            System.out.println(user.getAge());
            System.out.println(user.getEmail());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```