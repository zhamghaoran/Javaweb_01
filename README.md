# Javaweb_01
## 1.获取form表单的信息

```java
@Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");  // 设置编码，防止出现乱码
        String name = req.getParameter("name");
        String price = req.getParameter("price");
        System.out.println(name);
    }
```

## 2.打印信息到屏幕上

```java
 @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer = resp.getWriter();
        writer.print("hello_servlet");
    }

```

可以试着servlet的启动时间(越小越好)，这样做可以减少第一次请求的时间，在启动服务器的时候进行实例化和初始化

```xml
<load-on-startup>0</load-on-startup>
```

servlet在容器中是单例的，线程不安全(一个线程需要根据实例中的某个成员变量去做逻辑判断的时候，但是中间某个时间，另一个线程改变了这个变量)

所以最好不要再servlet中定义成员变量，如果不得不定义成员变量，那么不要根据成员变量的值去做逻辑判断。

## 3.HHTP无状态

HTTP 无状态：服务器无法区分这两个请求是同一个客户端发来的还是不同客户端发来的。

无状态的问题: 第一次请求是添加商品，第二次请求是结账，如果客户端无法区分是否是同一个客户端发来的，那么就会导致混乱。

解决方法:跟踪回话（每次回话的时候进行id分发）

1.客户端第一次发请求给服务器，服务器获取session，获取不到，则创建新的，然后响应给客户端

2.那么下次发送请求的时候就会把session带给服务器，这样服务器就可以分清客户端了

> request.getSession(false) 这个方法会获取当前回话，如果没有则不会创建。 
>
> session.getId()   获取session id
>
> session.isNew() 判断当前的session是不是新的
>
> session.getMaxInactiveInterval() session的非激活间隔时长 默认1800秒。
>
> session.setMaxInactiveInterval() 更改非激活间隔时长
>
> session.invalidate() 强制让会话失效

## 4.session 保存作用域：

是和session绑定的，换了客户端就没有了。

## 5.服务器内部转发和重定向

1.内部转发：客户端给服务器A发送请求之后，服务器A会在服务器的中间进行请求转发，最后由某个服务器给客户端，回复信息。

```java
req.getRequestDispatcher("demo2").forward(req,resp);  // 向demo2内部转发
```

2.重定向：客户端给服务器A发送请求之后，服务器A会让客户端去访问服务器B，然后客户端给服务器B发送请求。

```java
resp.sendRedirect("demo2");
```

## 6.ServletContext

web容器在启动的时候，它会为每个web程序创建一个对应的ServletContext对象，它代表了当前的web应用

### 1.共享数据

> 在这个Servlet中保存的数据，可以在另一个Servlet选中


```java
@Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String username = "123";
        servletContext.setAttribute("username",username);
    }
```

```java
@Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        Object username = servletContext.getAttribute("username");
        String name = (String) username;
        resp.setCharacterEncoding("UTF-8");
        resp.setContentType("text/html");
        resp.getWriter().println("名字" + name);
    }
```

### 2.获取初始化参数

```java
// web.xml 中的配置信息 可以从ServletContext中获取
<context-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost::3306</param-value>
</context-param>
```

```java
ServletContext servletContext = this.getServletContext();
String url = servletContext.getInitParameter("url");
```

### 3.请求转发

> 内部转发（url路径不会发生改变）

```java
ServletContext servletContext = this.getServletContext();  // 创建对象
servletContext.getRequestDispatcher("/test2").forward(req,resp);  // 进行转发
```

### 4.读取资源

## 7.下载文件

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    
    String realPath = this.getServletContext().getRealPath("/1.png");
    System.out.println("下载文件的路径" + realPath);
    // 获取文件的绝对路径
    realPath = "C:\\Users\\20179\\IdeaProjects\\demo6\\target\\classes\\1.png"; 
    // 获取文件名
    String FileName = realPath.substring(realPath.lastIndexOf("//") + 1);
    // 设置http响应头类型
    resp.setHeader("Content-Disposition","attachment;filename="+ URLEncoder.encode(FileName,"UTF-8"));
    // 设置文件流
    FileInputStream in = new FileInputStream(realPath);
    int len = 0;
    byte[] buffers = new byte[1024];
    // 获取流
    ServletOutputStream outputStream = resp.getOutputStream();
    while ((len = in.read(buffers)) > 0) {
        outputStream.write(buffers,0,len);
    }
    outputStream.close();
    in.close();
}

```

## 8.实现验证码

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    
    // 如何让浏览器五秒刷新一次
    resp.setHeader("refresh","3");
    
    // 创建图片
    BufferedImage Image = new BufferedImage(80,20,BufferedImage.TYPE_INT_RGB);
    
    //得到图片
    Graphics2D graphics = (Graphics2D) Image.getGraphics();  // 创建一支笔
    graphics.setColor(Color.white);
    graphics.fillRect(0,0,80,20);
    
    // 给图片写数据
    graphics.setColor(Color.blue);
    graphics.setFont(new Font(null,Font.BOLD,20));
    graphics.drawString(makeNum(),0,20);
    
    //告诉浏览器这个请求用图片的方式打开
    resp.setContentType("image/png");
    
    // 网站存在缓存，浏览器不缓存
    
    resp.setDateHeader("expries",-1);
    resp.setHeader("Cache-Control","no-cache");
    resp.setHeader("Pragma","no-cache");
    
    // 把图片写给浏览器
    ImageIO.write(Image,"jpg",resp.getOutputStream());
}
private String makeNum() {
    Random random = new Random();
    String s = random.nextInt(999999) + "";
    StringBuffer sb = new StringBuffer();
    for(int i = 0;i < 7-s.length();i ++) {
        sb.append("0");
    }
    String s1 = sb.toString() + s;
    return s1;
}
```

## 9.重定向

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         resp.sendRedirect("/test1");
    }
```

### 1.重定向和转发的区别:

不通点：

> 请求转发的时候url不会发生变化

> 重定向的时候url会发生变化

相同点：

> 都会跳转网页

## 10.HttpServletRequest

​	代表客户端的一个请求，用户通过该http协议访问服务器，http请求中的所有信息会被封装到request中，通过这个方法可以获取客户端的所有信息。

### 1.获取前端的参数

> ```
> String[] hobbies = req.getParameterValues("hobby");
> ```

### 2.请求转发

```java
req.getRequestDispatcher("/success.html").forward(req,resp);
```

code:

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String[] hobbies = req.getParameterValues("hobby");
    String name = req.getParameter("name");
    String password = req.getParameter("password");
    System.out.println(Arrays.toString(hobbies));
    req.getRequestDispatcher("/success.html").forward(req,resp);
}
```



