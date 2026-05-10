# Báo cáo Bài tập Lập trình Java & OOP (CI/CD, Code Quality)

**Học viên:** [Điền tên]  
**Môn học:** Lập trình Java & Hướng Đối Tượng (OOP)

Dưới đây là phần bài làm và báo cáo thực hành cho các bài tập từ Bài 1 đến Bài 10. Quá trình làm bài bao gồm cài đặt môi trường ở máy cá nhân, tạo repository trên GitHub và debug lỗi của pipeline.

---

## 📌 PHẦN 1: Cấu Hình & CI/CD (Bài 1 - Bài 8)

### Bài 1: Quản lý Dependency với Maven
**Yêu cầu:** Nâng cấp file `pom.xml` cũ, thêm thư viện mới và sửa lỗi xung đột.

**Bài làm:**
Vào file `pom.xml` xóa ngay dependency của JUnit 4 cũ để code chạy hoàn toàn trên môi trường của JUnit Jupiter (JUnit 5), tránh các lỗi conflict linh tinh. Sau đó thêm các thư viện đúng version theo yêu cầu:

```xml
<dependencies>
    <!-- Thêm Logback thay cho System.out.println() -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
    
    <!-- Thêm Hibernate Core -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>6.2.0.Final</version>
    </dependency>
    
    <!-- Dùng JUnit 5 làm công cụ test duy nhất -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.9.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Bài 2: Code Quality - Checkstyle
**Yêu cầu:** Tích hợp Checkstyle Plugin và giải thích các cấp độ log.

**Bài làm:**
Cấu hình Checkstyle dùng tiêu chuẩn của Google. Code viết sai chuẩn format là Maven sẽ báo lỗi:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <configLocation>google_checks.xml</configLocation>
        <consoleOutput>true</consoleOutput>
        <failsOnError>true</failsOnError>
    </configuration>
</plugin>
```
Phân bổ các cấp độ log (Log Levels) như sau:
- **INFO:** Dùng để ghi lại các thao tác nghiệp vụ bình thường. Ví dụ: "Tài khoản A vừa nạp 50k". Ghi lại để sau này dễ tra cứu luồng hoạt động.
- **WARN:** Dùng khi thấy có dấu hiệu bất thường nhưng chưa làm sập app. Ví dụ: "Người dùng đăng nhập sai mật khẩu 3 lần".
- **ERROR:** Bắt buộc dùng khi có `Exception`, lỗi mạng, lỗi DB, để team dev nắm được và fix ngay.

### Bài 3: CI/CD Automation
**Bài làm:**
Sử dụng file `.github/workflows/ci.yml` chuẩn. Nhờ workflow này mà mỗi lần push code lên, GitHub sẽ tự mang đi test và đóng gói thành file `.jar`.

```yaml
name: Java CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Kéo code từ repo về
        uses: actions/checkout@v3
        
      - name: Cài đặt JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          
      - name: Chạy Maven Build (Test và Package)
        run: mvn clean verify
        
      - name: Lưu trữ file build xong (.jar)
        uses: actions/upload-artifact@v3
        with:
          name: ung-dung-chay-duoc
          path: target/*.jar
```

### Bài 4: Kiểm thử đa hệ điều hành (Matrix Strategy)
**Bài làm:**
Test lúc đầu chạy bình thường trên Windows (vì dùng đường dẫn `String path = "thu_muc\\file.txt";`). Nhưng khi lên CI chạy Ubuntu là báo lỗi ngay vì bên Linux dùng dấu `/`.
Nâng cấp CI để test trên cả 3 HĐH:
```yaml
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
```
**Cách fix lỗi:** Refactor lại code, không viết cứng dấu `\` hay `/`, mà xài API `java.nio.file.Paths`:
```java
Path path = Paths.get("thu_muc", "file.txt");
String duongDanChuan = path.toString();
```
Sau khi sửa, test đã pass thành công trên mọi hệ điều hành.

### Bài 5: Test Coverage (JaCoCo)
**Bài làm:**
Cấu hình JaCoCo ép buộc độ bao phủ code (Coverage) phải trên 80%, nếu không thì tự động fail bản build:
```xml
<execution>
    <id>check</id>
    <goals><goal>check</goal></goals>
    <configuration>
        <rules>
            <rule>
                <element>BUNDLE</element>
                <limits>
                    <limit>
                        <counter>INSTRUCTION</counter>
                        <value>COVEREDRATIO</value>
                        <minimum>0.80</minimum>
                    </limit>
                </limits>
            </rule>
        </rules>
    </configuration>
</execution>
```

### Bài 6: CI/CD Pipeline Optimization & Caching
**Bài làm:**
Bình thường chạy CI/CD sẽ phải tải lại toàn bộ thư viện từ Maven Central mất rất nhiều thời gian. Chỉ cần thêm `cache: 'maven'` vào bước setup-java:
```yaml
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'
```
Hiệu quả thấy rõ ngay từ lần push thứ 2, đọc log xuất hiện dòng `Restoring cache...`, quá trình lấy thư viện nhanh gọn hơn hẳn, chỉ tốn thêm vài giây.

### Bài 7 & Bài 8: Branch Protection và Đóng gói
**Bài làm:**
- **Bài 7:** Dùng branch protection của Github, chọn `Require status checks to pass before merging`. Nhờ đó, nếu code sai format (Checkstyle báo lỗi), nút Merge PR sẽ bị vô hiệu hóa (khoá lại). Sửa code xanh lại mới cho phép merge.
- **Bài 8:** Cấu hình `maven-jar-plugin` chỉ định class Main. Xong xuôi mở terminal gõ `java -jar target/app.jar` là app tự chạy, không cần IDE. Thư mục `target` sinh ra để chứa sản phẩm đầu ra, giữ cho source code luôn sạch sẽ gọn gàng.

---

## 📌 PHẦN 2: Bắt Bug The Broken Pipeline (Bài 9 - Bài 10)

### Bài 9: Triển khai Logging chuyên nghiệp
Bỏ thói quen dùng `System.out.println()`, chuyển sang dùng SLF4J với Logback. 

**1. Code Java (Sử dụng dấu ngoặc nhọn `{}` thay cho dấu `+` để tối ưu):**
```java
package com.lab;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AppLogService {
    // Khai báo Logger
    private static final Logger logger = LoggerFactory.getLogger(AppLogService.class);

    public void processData(String username, int amount) {
        // Tham số hóa {} thay cho cộng chuỗi
        logger.info("Bắt đầu nạp tiền cho user: {}, số tiền: {}", username, amount);
        
        try {
            if (amount < 0) {
                throw new IllegalArgumentException("Số tiền nạp không thể là số âm!");
            }
            logger.info("Giao dịch của {} đã thành công.", username);
        } catch (Exception e) {
            // Ném lỗi vào ERROR kèm stack trace để dễ điều tra
            logger.error("Xảy ra lỗi khi xử lý giao dịch cho {}: {}", username, e.getMessage(), e);
        }
    }
}
```

**2. Cấu hình `logback.xml` để vừa log ra màn hình, vừa lưu ra file:**
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Appender lưu log ra tệp vật lý -->
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/ung-dung.log</file>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

---

### Bài 10: Vạch lá tìm sâu - The Broken Pipeline

Kịch bản là lấy code Bài 10 cho sẵn đẩy lên GitHub, và pipeline bị báo lỗi đỏ. Dưới đây là hành trình đọc log và sửa 3 lỗi:

#### 🚨 Lỗi 1: Đòi build khi chưa có code
- **Vị trí có vấn đề:** Tệp `.github/workflows/ci.yml`
- **Tình trạng:** Trong tab Actions báo lỗi: `The goal you specified requires a project to execute but there is no POM in this directory (/).`
- **Nhận ra là:** Máy ảo (runner) của GitHub lúc bật lên hoàn toàn rỗng. Workflow chạy lệnh `mvn package` trong khi chưa hề kéo code (checkout) từ repo về, dẫn đến Maven không tìm thấy file `pom.xml`.
- **Cách fix:** Thêm dòng action checkout vào đầu phần steps:
  ```yaml
      steps:
        - name: Kéo code về máy ảo
          uses: actions/checkout@v3
        - name: Set up JDK 17
          # ...
  ```

#### 🚨 Lỗi 2: Dependency "ảo ảnh"
- **Vị trí có vấn đề:** Tệp `pom.xml` (khai báo thư viện Logback)
- **Tình trạng:** Log báo lỗi: `[ERROR] Could not resolve dependencies... ch.qos.logback:logback-classic:jar:9.9.9 was not found in...`
- **Nhận ra là:** Version của Logback đang là `9.9.9`. Trên Maven Central không hề có bản này. Đây là một version ảo cố tình ghi sai.
- **Cách fix:** Chỉnh lại thành phiên bản có thật, ví dụ `1.4.11`. Push code lên là Maven tải về được ngay.

#### 🚨 Lỗi 3: Surefire Plugin cũ không hiểu Java 17
- **Vị trí có vấn đề:** Tệp `pom.xml` (plugin `maven-surefire-plugin`)
- **Tình trạng:** Chạy tới phase test thì sụp đổ với lỗi: `Execution default-test of goal... failed: Unsupported class file major version 61`
- **Nhận ra là:** Plugin Surefire bản `2.12.4` phát hành từ hồi 2012. Nó quá cũ, không đọc được bytecode của Java 17 (major version 61) và cũng không chạy được code JUnit 5.
- **Cách fix:** Cập nhật plugin lên bản đời mới có hỗ trợ JUnit Jupiter:
  ```xml
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-surefire-plugin</artifactId>
                  <version>3.1.2</version>
              </plugin>
  ```
Sau khi sửa, Pipeline đã Xanh trở lại.

#### 🛠️ Lỗi thứ 4: Tự tạo thêm lỗi logic code
Sau khi pipeline đã Passed, tự tạo thêm một lỗi xem hệ thống có bắt được không.

- **Thao tác phá hoại:** Mở file `ShippingCalculator.java`, công thức tính phí EXPRESS đang là `return weight * 5000 + 20000;`. Cố tình trừ đi 10k, đổi thành `return weight * 5000 + 10000;`. Commit lên.
- **Kết quả:** Vừa đẩy lên vài giây sau pipeline báo ĐỎ! Soi log thấy báo lỗi chỗ Unit Test:
  ```text
  [ERROR] Failures: 
  [ERROR]   ShippingCalculatorTest.testExpress:18 expected: <45000.0> but was: <35000.0>
  ```
- **Phân tích:** Test case truyền 5 kg vào, tính ra đúng phải là 45.000. Do sửa sai công thức, code trả về có 35.000. Lệch khớp! Test Fail, kéo theo cả CI Pipeline Fail. Hệ thống CI/CD bắt lỗi hoàn toàn chính xác.
- **Dọn dẹp:** Sửa công thức lại đúng như cũ (`+ 20000`), push lại và pipeline tiếp tục màu Xanh. Quá trình làm bài tập hoàn tất.
