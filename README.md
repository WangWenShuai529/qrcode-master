# 博客地址
[https://blog.csdn.net/m0_61504367/article/details/124550280](https://blog.csdn.net/m0_61504367/article/details/124550280)
# 版本信息
JDK 1.8（这里可以在pom.xml中修改）
spring boot 2.3.2
# 项目结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/deb2027fee3c4d0b8dbe416d44e78476.png)


# 代码
## pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.qrcode</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--qrcode begin-->
        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>core</artifactId>
            <version>3.4.0</version>
        </dependency>

        <dependency>
            <groupId>com.google.zxing</groupId>
            <artifactId>javase</artifactId>
            <version>3.4.0</version>
        </dependency>
        <!--qrcode   end-->

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
## 修改JDK版本
（如果你的不是1.8），在这里设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/99b067f9ab7248d7bf664004731769a4.png)

## QrCodeUtil工具类
最重要的是生成二维码
```java
package com.qrcode.demo.util;

import java.awt.BasicStroke;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.Shape;
import java.awt.geom.RoundRectangle2D;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.OutputStream;
import java.util.Hashtable;

import javax.imageio.ImageIO;

import com.google.zxing.BarcodeFormat;
import com.google.zxing.BinaryBitmap;
import com.google.zxing.DecodeHintType;
import com.google.zxing.EncodeHintType;
import com.google.zxing.MultiFormatReader;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.Result;
import com.google.zxing.client.j2se.BufferedImageLuminanceSource;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.common.HybridBinarizer;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;

/**
 * 二维码工具类
 * by liuhongdi
 */
public class QrCodeUtil {

    //编码格式,采用utf-8
    private static final String UNICODE = "utf-8";
    //图片格式
    private static final String FORMAT = "JPG";
    //二维码宽度像素pixels数量
    private static final int QRCODE_WIDTH = 300;
    //二维码高度像素pixels数量
    private static final int QRCODE_HEIGHT = 300;
    //LOGO宽度像素pixels数量
    private static final int LOGO_WIDTH = 100;
    //LOGO高度像素pixels数量
    private static final int LOGO_HEIGHT = 100;

    //生成二维码图片
    //content 二维码内容
    //logoPath 图片地址
    private static BufferedImage createImage(String content, String logoPath) throws Exception {
        Hashtable<EncodeHintType, Object> hints = new Hashtable<EncodeHintType, Object>();
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H);
        hints.put(EncodeHintType.CHARACTER_SET, UNICODE);
        hints.put(EncodeHintType.MARGIN, 1);
        BitMatrix bitMatrix = new MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, QRCODE_WIDTH, QRCODE_HEIGHT,
                hints);
        int width = bitMatrix.getWidth();
        int height = bitMatrix.getHeight();
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        for (int x = 0; x < width; x++) {
            for (int y = 0; y < height; y++) {
//                0xFF000000黑色
//                0xFFFFFFFF 白色
                image.setRGB(x, y, bitMatrix.get(x, y) ? 0xFF000000 : 0xFFFFFFFF);
            }
        }
        if (logoPath == null || "".equals(logoPath)) {
            return image;
        }
        // 插入图片
        QrCodeUtil.insertImage(image, logoPath);
        return image;
    }

    //在图片上插入LOGO
    //source 二维码图片内容
    //logoPath LOGO图片地址
    private static void insertImage(BufferedImage source, String logoPath) throws Exception {

        File file = new File(logoPath);
        System.out.println("    //在图片上插入LOGO");
        if (!file.exists()) {
            throw new Exception("logo file not found.");
        }
        Image src = ImageIO.read(new File(logoPath));
        int width = src.getWidth(null);
        int height = src.getHeight(null);
            if (width > LOGO_WIDTH) {
                width = LOGO_WIDTH;
            }
            if (height > LOGO_HEIGHT) {
                height = LOGO_HEIGHT;
            }
            Image image = src.getScaledInstance(width, height, Image.SCALE_SMOOTH);
            BufferedImage tag = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
            Graphics g = tag.getGraphics();
            g.drawImage(image, 0, 0, null); // 绘制缩小后的图
            g.dispose();
            src = image;
        // 插入LOGO
        Graphics2D graph = source.createGraphics();
        int x = (QRCODE_WIDTH - width) / 2;
        int y = (QRCODE_HEIGHT - height) / 2;
        graph.drawImage(src, x, y, width, height, null);
        Shape shape = new RoundRectangle2D.Float(x, y, width, width, 6, 6);
        graph.setStroke(new BasicStroke(3f));
        graph.draw(shape);
        graph.dispose();
    }

    //生成带logo的二维码图片，保存到指定的路径
    // content 二维码内容
    // logoPath logo图片地址
    // destPath 生成图片的存储路径
    public static String save(String content, String logoPath, String destPath) throws Exception {
        BufferedImage image = QrCodeUtil.createImage(content, logoPath);
        File file = new File(destPath);
        String path = file.getAbsolutePath();
        File filePath = new File(path);
        if (!filePath.exists() && !filePath.isDirectory()) {
            filePath.mkdirs();
        }
        String fileName = file.getName();
        fileName = fileName.substring(0, fileName.indexOf(".")>0?fileName.indexOf("."):fileName.length())
                + "." + FORMAT.toLowerCase();
        System.out.println("destPath:"+destPath);
        ImageIO.write(image, FORMAT, new File(destPath));
        return fileName;
    }

    //生成二维码图片，直接输出到OutputStream
    public static void encode(String content, String logoPath, OutputStream output)
            throws Exception {
        BufferedImage image = QrCodeUtil.createImage(content, logoPath);
        ImageIO.write(image, FORMAT, output);
    }

    //解析二维码图片，得到包含的内容
    public static String decode(String path) throws Exception {
        File file = new File(path);
        BufferedImage image = ImageIO.read(file);
        if (image == null) {
            return null;
        }
        BufferedImageLuminanceSource source = new BufferedImageLuminanceSource(image);
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        Result result;
        Hashtable<DecodeHintType, Object> hints = new Hashtable<DecodeHintType, Object>();
        hints.put(DecodeHintType.CHARACTER_SET, UNICODE);
        result = new MultiFormatReader().decode(bitmap, hints);
        return result.getText();
    }
}

```

## controller
这里要注意路径
```java
package com.qrcode.demo.controller;

import com.qrcode.demo.util.QrCodeUtil;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.OutputStream;

@RequestMapping("/home")
@Controller
public class HomeController {

    //生成带logo的二维码到response
    @RequestMapping("/qrcode")
    public void qrcode(HttpServletRequest request, HttpServletResponse response) {
        String requestUrl = "http://www.baidu.com";
        try {
            OutputStream os = response.getOutputStream();
//            QrCodeUtil.encode(requestUrl, "/data/springboot2/logo.jpg", os);
            QrCodeUtil.encode(requestUrl, "src/main/resources/logo.jpg", os);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //生成不带logo的二维码到response
    @RequestMapping("/qrnologo")
    public void qrnologo(HttpServletRequest request, HttpServletResponse response) {
        String requestUrl = "http://www.baidu.com";
        try {
            OutputStream os = response.getOutputStream();
            QrCodeUtil.encode(requestUrl, null, os);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //把二维码保存成文件
    @RequestMapping("/qrsave")
    @ResponseBody
    public String qrsave() {
        String requestUrl = "http://www.baidu.com";
        try {
//            QrCodeUtil.save(requestUrl, "/data/springboot2/logo.jpg", "/data/springboot2/qrcode2.jpg");
            QrCodeUtil.save(requestUrl, "src/main/resources/logo.jpg", "src/main/resources/qrcode2.jpg");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "文件已保存";
    }

    //解析二维码中的文字
    @RequestMapping("/qrtext")
    @ResponseBody
    public String qrtext() {
        //String requestUrl = "http://www.baidu.com";
        String url = "";
        try {
             url = QrCodeUtil.decode("src/main/resources/qrcode2.jpg");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "解析到的url:"+url;
    }

}

```
## DemoApplication
```java
package com.qrcode.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

# 运行截图
![在这里插入图片描述](https://img-blog.csdnimg.cn/927b8c3bb2c5462b97e98373e6384f02.png)
## http://127.0.0.1:8080/home/qrnologo
![在这里插入图片描述](https://img-blog.csdnimg.cn/b23a5b13724240c0b887248063504e8f.png)
## http://127.0.0.1:8080/home/qrcode
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b0fef5d65bb4bebb684d6c175c75b54.png)
## http://127.0.0.1:8080/home/qrsave
![在这里插入图片描述](https://img-blog.csdnimg.cn/936ac5aede064505b36d5d5765698d10.png)

## http://127.0.0.1:8080/home/qrtext
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a67beae4ce24af1a9c8f131564846f8.png)
# 代码下载
[https://download.csdn.net/download/m0_61504367/85269278](https://download.csdn.net/download/m0_61504367/85269278)
