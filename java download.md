# Java文件下载功能

HTTP协议提供了文件下载功能，具体参考：rfc2616.txt

其原理为：

![](C:/Users/WH/Desktop/Linux-note/imgs/img.png)

> 只要设置Content-Disposition响应头浏览器就可以自动保存文件了。 Spring MVC Servlet Socket等技术都可以实现下载功能。

### 图片下载案例

1. 创建控制器方法处理下载请求：

   ```java
   /**
     * 图片下载功能
     */
   @RequestMapping(value="/img.do",
                   produces="image/png")
   @ResponseBody
   public byte[] image( 
       HttpServletResponse response)
       throws IOException{
       //@ResponseBody 与 返回值byte[] 配合时候
       //Spring MVC会将byte[]数组填充到响应
       //的消息正文中发送到浏览器
       String file = 
           URLEncoder.encode("演示.png", "utf-8");
       //还需要指定两个响应头 
       //Content-Type 
       //Content-Disposition: attachment; filename="fname.ext"
       //response.setContentType("image/png");
       response.setHeader(
           "Content-Disposition", 
           "attachment; filename=\""+file+"\""); 
       byte[] body = createImage();
       //response.setContentLength(body.length);
       return body;
   }
   ```

2. 创建生成图片数据的方法

   ```java
   private byte[] createImage() throws IOException{
       BufferedImage img=
           new BufferedImage(100, 50, 
                             BufferedImage.TYPE_3BYTE_BGR);
       img.setRGB(50, 25, 0xffffff);
       //out相当于酱油瓶
       ByteArrayOutputStream out=
           new ByteArrayOutputStream();
       //将图片的数据导入酱油瓶out
       ImageIO.write(img, "png", out);
       out.close();//关闭out
       //将酱油瓶out中的数据到出来
       byte[] bytes = out.toByteArray();
       return bytes;
   }
   ```

3. 配置spring-mvc.xml在拦截器上放开 URL

   <mvc:exclude-mapping path="/user/img.do"/> 

4. 编写demo.html

   <!DOCTYPE html>
   	<html>
   	<head>
   	<meta charset="UTF-8">
   	<title>Insert title here</title>
   	</head>
   	<body>
   	  <h1>下载功能</h1>
   	  <a href="user/img.do">下载测试图片</a>
   	</body>
   	</html>

5. 测试。

### 下载Excel功能

与下载图片类似，可以实现下载Excel功能，其原理为：

![](C:/Users/WH/Desktop/Linux-note/imgs/excel.png)

1. 导入Apache 提供的 Excel API

   ```xml
   <!-- POI API 用于处理Excel文件 -->
   <dependency>
       <groupId>org.apache.poi</groupId>
       <artifactId>poi-ooxml</artifactId>
       <version>3.17</version>
   </dependency>
   ```

2. 编写控制器方法

   ```java
   /*
   * 下载 Excel
   */
   @RequestMapping("/excel.do")
   @ResponseBody
   public byte[] export(
       HttpServletResponse response)
       throws IOException{
       String file=URLEncoder.encode(
       "表格.xlsx", "UTF-8");
       response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
       response.setHeader(
       "Content-Disposition",
       "attachment; filename=\""+file+"\"");
       byte[] body = createExcel(); 
       return body;
   }
   ```

   

3. 编写利用POI 生成Excel的方法

   	private byte[] createExcel() throws IOException {
   	    //Workbook 就代表着一个Excel文件
   	    XSSFWorkbook workbook = 
   	    new XSSFWorkbook();
   	    //在工作簿中创建一个工作表(sheet)
   	    XSSFSheet sheet=
   	    workbook.createSheet("第一个表");
   	    //在工作表中创建一行(row), 参数是行号：0 1 2 ...
   	    XSSFRow row = sheet.createRow(0);
   	    //在行中可以添加格子, 参数是列号：0 1 2 3 ...
   	    XSSFCell cell = row.createCell(0);
   	    cell.setCellValue("Hello World!"); 
   		ByteArrayOutputStream out=
   				new ByteArrayOutputStream();
   		workbook.write(out);
   		workbook.close();
   		out.close();
   		byte[] bytes=out.toByteArray();
   		return bytes;
   	}

4. 配置拦截器，放开URL

   <mvc:exclude-mapping path="/user/excel.do"/>

5. 编写HTML

   <!DOCTYPE html>
   	<html>
   	<head>
   	<meta charset="UTF-8">
   	<title>Insert title here</title>
   	</head>
   	<body>
   	  <h1>下载功能</h1>
   	  <a href="user/img.do">下载测试图片</a>
   	  <a href="user/excel.do">导出Excel</a>
   	</body>
   	</html>

6. 测试