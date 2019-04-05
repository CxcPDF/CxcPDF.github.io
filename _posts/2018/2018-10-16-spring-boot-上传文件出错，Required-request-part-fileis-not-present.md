---
layout: post
title: Spring Boot上传文件出错，Required request part fileis not present
categories: [Spring Boot]
description: Spring Boot上传文件出错，Required request part fileis not present
keywords: Spring Boot, 文件上传


---

先上代码：

```java
@RestController
@RequestMapping("/file")
//@PreAuthorize("hasAuthority(ROLE_USER)")
public class FileController {


    /**
     * 提取文件上传的公用代码
     *
     * @param uploadDir //文件储存路径
     * @param file      //要上传的文件
     */
    private void executeUpload(String studentId, String uploadDir, MultipartFile file) {

        //文件后缀名
        String suffix = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf("."));
        //上传文件名
//        String fileName = UUID.randomUUID() + suffix;
        //文件名为当前日期，如2018101310243010（2018年10月13日10时24分。。。），再加上studentId
        String fileName = DateUtil.convert2String(new Date()) + studentId + suffix;
        //服务器端保存的文件对象
        File serverFile = new File(uploadDir + fileName);
        //将上传的文件写入到服务器端文件内
        try {
            file.transferTo(serverFile);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 单文件上传
     *
     * @param request
     * @param file    前台上传的文件对象
     * @return
     */
    @PostMapping("/fileupload")
    @ResponseBody
    public String upload(Map map, HttpServletRequest request, HttpServletResponse response, @RequestParam("file")MultipartFile file) {


        try {
            String id = (String) map.get("id");
            //上传目录地址
            String uploadDir = request.getSession().getServletContext().getRealPath("/") + "/upload/";
            //如果目录不存在，自动创建目录
            File dir = new File(uploadDir);
            if (!dir.exists()) {
                dir.mkdir();
            }

            //调用上传方法
            executeUpload(id, uploadDir, file);

        } catch (Exception e) {
            //打印错误堆栈信息
            e.printStackTrace();
            return "上传失败";
        }


        return "上传成功";
    }
}
```

今天使用postman来测试文件上传的时候，总是报“Required request part 'file' is not present”的错误，查阅了很多资料也还是无法解决这个问题。突然想到```@RequestParam("file")MultipartFile file```中RequestParam需要的参数是“file”，那我在postman中指定文件的key为“file”，来测试一下，发现竟然成功了！！到这里我就想去查一下注解RequestParam的作用。

@RequestParam是传递参数的.

@RequestParam用于将请求参数区数据映射到功能处理方法的参数上。

```
public String queryUserName(@RequestParam String userName)
```

在url中输入:localhost:8080/**/?userName=zhangsan

请求中包含username参数（如/requestparam1?userName=zhang），则自动传入。

接下来我们看一下@RequestParam注解主要有哪些参数：

value：参数名字，即入参的请求参数名字，如username表示请求的参数区中的名字为username的参数的值将传入；

required：是否必须，默认是true，表示请求中一定要有相应的参数，否则将报404错误码；

defaultValue：默认值，表示如果请求中没有同名参数时的默认值，默认值可以是SpEL表达式，如“#{systemProperties['java.vm.version']}”。





之前我在上传文件的时候指定的key与@RequestParam中要求的不一致，导致了后台不能获取到上传的文件。