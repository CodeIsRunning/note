## 有效"缓解"对接不仔细读文档的习惯

### RestControllerAdvice结合Valid有效"缓解"对接的时候不仔细看文档的习惯

1、引入依赖

```
<dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
 </dependency>
```

2、经典学生类

```java
@Data
public class Student {


    private Integer id;

    @NotBlank
    private String name;

    private Integer sex;

    private String address;

    private LocalDateTime createTime;

    private LocalDateTime updateTime;

    private String note;
}
```

![image-20201225132313797](http://cdn.liuxf.live/20201225132313.png)

在需要控制的属性上增加响应的注解，里面内置了很多控制选项，能省略大量的if else 判定代码

3、测试接口@Valid 启动验证

```java
@RequestMapping("save")
public void test(@RequestBody @Valid Student student) {

    log.info(student.toString());

}
```

4、测试验证

![image-20201225132630215](http://cdn.liuxf.live/20201225132630.png)

由于没有传name参数已经，api请求会报4xx错，这样感觉不太人道主义，有时候还要配合他们巴拉日志看，观察后台日志看到有个警告异常。

![image-20201225132944062](http://cdn.liuxf.live/20201225132944.png)

5、友好返还

配合RestControllerAdvice全局异常捕获，捕获MethodArgumentNotValidException该异常，就可以自定义数据的返还。

6、定义全局异常捕捉

```java
@RestControllerAdvice
public class ControllerAdvice {

    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public Object methodArgumentException(MethodArgumentNotValidException e) {

        //可以自己定义统一返还格式
        
        Map map = new HashMap();

        map.put("code", "400");

        StringBuilder stringBuilder = new StringBuilder();
        for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
            stringBuilder.append(fieldError.getDefaultMessage()).append("|").append(fieldError.getField()).append("|");
        }

        map.put("msg",stringBuilder);

        return map;
    }

}
```

再次请求接口发现已经能正常返还错误。

![image-20201225133639340](http://cdn.liuxf.live/20201225133639.png)

7、到此完成，对方不用每次来问我传的什么错了，少传什么了。这一套往上一配，可以直接告诉他看返还参数，这里还可以在返还的时候不把错误一次返完，一次一个的返还，可以使对接同学记忆犹新。



## 总结

能减少代码的尽量精简，有可以用的轮子尽量用切勿自己造

古人云 "善假于物也"