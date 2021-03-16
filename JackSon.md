> Spring默认使用的序列化工具（ 对象  <==> JSON字符）

### 常用注解

```
@JsonIgnore 此注解用于属性上，作用是进行JSON操作（如序列化）时忽略该属性。
@JsonFormat 此注解用于属性上，作用是把Date类型直接转化为想要的格式，
		如@JsonFormat(pattern = "yyyy-MM-dd HH-mm-ss"，timezone = "Asia/Shanghai")。
@JsonProperty 此注解用于属性上，作用是把该属性的名称序列化为另外一个名称，
		如把trueName属性序列化为name，@JsonProperty("name")
```

### **序列化和反序列化**

>  ObjectMapper是JSON操作的核心，Jackson的所有JSON操作都是在ObjectMapper中实现。 
>          * ObjectMapper有多个JSON序列化的方法，可以把JSON字符串保存File、OutputStream等不同的介质中。 
>                   * wri   * writeValue(File arg0, Object arg1)把arg1转成json序列，并保存到arg0文件中。 
>             * writeValue(OutputStream arg0, Object arg1)把arg1转成json序列，并保存到arg0输出流中。
>             * writeValueAsBytes(Object arg0)把arg0转成json序列，并把结果输出成字节数组。 
>             * **writeValueAsString(Object arg0)把arg0转成json序列，并把结果输出成字符串。**
>    * ObjectMapper支持从byte[]、File、InputStream、字符串等数据的JSON反序列化。
>            *  readValue(jsonString, type)

```java
package com.st.json;

import java.io.IOException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import com.fasterxml.jackson.core.JsonParseException;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JacksonDemo {
    
    public static void main(String[] args) throws ParseException, IOException {
        jsonTest();
    }
    
    /**
     * jackson序列化的使用
     * @throws ParseException
     * @throws JsonProcessingException
     */
    public static void jackTest() throws ParseException, JsonProcessingException {
        User u = new User();
        u.setId(1);
        u.setName("curry");
        u.setAge(30);
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        u.setBirthday(dateFormat.parse("1988-9-21"));
        u.setEmail("138@163.com");
        

        ObjectMapper mapper = new ObjectMapper();
        
        //User对象转Json,
        //输出{"id":1,"name":"curry","age":30,"birthday":590774400000,"email":"138@163.com"}
        String jsonValue = mapper.writeValueAsString(u);
        System.out.println(jsonValue);
        
        User u2 = new User();
        u2.setId(2);
        u2.setName("KD");
        u2.setAge(29);
        u2.setBirthday(dateFormat.parse("1989-9-21"));
        u2.setEmail("123@qq.com");
        
        
        List<User> users = new ArrayList<>();
        users.add(u);
        users.add(u2);
        String jsonList = mapper.writeValueAsString(users);
        System.out.println(jsonList);
        
    }
    
    /**
     * JSON转Java对象[JSON反序列化]
     * @throws IOException 
     * @throws JsonMappingException 
     * @throws JsonParseException 
     */

    public static void jsonTest() throws JsonParseException, JsonMappingException, IOException {
        String json = " {\"id\":3, \"name\":\"小明\", \"age\":18, \"birthday\":590774400000, \"email\":\"xiaomin@sina.com\"} ";  
        
        /** 
         * ObjectMapper支持从byte[]、File、InputStream、字符串等数据的JSON反序列化。 
         */  
        ObjectMapper mapper = new ObjectMapper();
        User user = mapper.readValue(json, User.class);
        System.out.println(user); 
    }
}
```

