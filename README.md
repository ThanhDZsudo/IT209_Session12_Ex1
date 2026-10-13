# BÁO CÁO BÀI TẬP: XÂY DỰNG QUY TRÌNH CI ĐA PHIÊN BẢN JDK (MATRIX STRATEGY) CHO JAVA SPRING BOOT GRADLE

> **Môn học:** IT209 - DevOps  
> **Bài tập:** Session 12 - Bài 1: Matrix Strategy CI cho Java Spring Boot Gradle  
> **Người thực hiện:** Nguyễn Tiến Thành  
> **Đường dẫn nộp bài:** `homework/session_12/ex1/`  
> **Sản phẩm nộp bài:**  
> - File workflow: [.github/workflows/ci-matrix.yml](file:///e:/BaiTap/IT209_Devops/Session12/Ex1/.github/workflows/ci-matrix.yml)  
> - Báo cáo & tài liệu hướng dẫn: [README.md](file:///e:/BaiTap/IT209_Devops/Session12/Ex1/README.md)  
> - Mã nguồn dự án Spring Boot Gradle đầy đủ unit test  

---

## 1. Mục tiêu bài thực hành

1. **Cấu hình ma trận kiểm thử tự động (Matrix Strategy)**:
   - Tự động hóa quá trình chạy song song (parallel execution) toàn bộ test suite trên nhiều phiên bản JDK mục tiêu (**JDK 17** và **JDK 21**).
   - Đảm bảo chất lượng mã nguồn (Code Quality) và khả năng tương thích ngược/tiến (Forward/Backward Compatibility) trước khi mã nguồn được tích hợp vào nhánh chính `main`.
2. **Tối ưu hóa hiệu năng CI với Gradle Dependency Caching**:
   - Sử dụng tính năng tích hợp `cache: 'gradle'` của action `actions/setup-java@v5` để cache toàn bộ thư mục `.gradle/caches` và wrapper distributions.
   - Giảm đáng kể thời gian tải dependency từ Maven Central và giảm tiêu tốn băng thông mạng giữa các lần chạy pipeline.
3. **Chuẩn hóa quy trình CI/CD hiện đại**:
   - Sử dụng phiên bản action mới nhất (`actions/checkout@v5`, `actions/setup-java@v5`).
   - Phân quyền thực thi tự động cho script Gradle Wrapper (`chmod +x gradlew`).
   - Tự động kích hoạt khi có sự kiện `push` lên nhánh `main`, kèm hỗ trợ chạy thủ công (`workflow_dispatch`).

---

## 2. Sơ đồ luồng hoạt động (Workflow Architecture)

```mermaid
graph TD
    A[Push to main branch / workflow_dispatch] --> B[Trigger GitHub Actions Workflow]
    B --> C[Job: test-matrix]
    
    subgraph Matrix Strategy
        C --> D1["Sub-job 1: Test on Java 17 (ubuntu-latest)"]
        C --> D2["Sub-job 2: Test on Java 21 (ubuntu-latest)"]
    end
    
    subgraph Steps for Java 17
        D1 --> E1[actions/checkout@v5]
        E1 --> F1["actions/setup-java@v5<br/>JDK 17 | Temurin | cache: gradle"]
        F1 --> G1[chmod +x gradlew]
        G1 --> H1[./gradlew test]
        H1 --> I1([PASS: Green Checkmark])
    end
    
    subgraph Steps for Java 21
        D2 --> E2[actions/checkout@v5]
        E2 --> F2["actions/setup-java@v5<br/>JDK 21 | Temurin | cache: gradle"]
        F2 --> G2[chmod +x gradlew]
        G2 --> H2[./gradlew test]
        H2 --> I2([PASS: Green Checkmark])
    end
```

---

## 3. Cấu trúc thư mục dự án

```text
Session12/Ex1/
├── .github/
│   └── workflows/
│       └── ci-matrix.yml           # Kịch bản GitHub Actions Matrix CI
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar      # Gradle Wrapper binary
│       └── gradle-wrapper.properties
├── src/
│   ├── main/
│   │   ├── java/com/example/studentapi/
│   │   │   ├── controller/StudentController.java
│   │   │   ├── exception/GlobalExceptionHandler.java
│   │   │   ├── repository/StudentRepository.java
│   │   │   ├── service/StudentService.java
│   │   │   ├── Gender.java
│   │   │   ├── Student.java
│   │   │   └── StudentApiApplication.java
│   │   └── resources/
│   │       └── application.yml     # Cấu hình production/dev
│   └── test/
│       ├── java/com/example/studentapi/
│       │   ├── service/StudentServiceTest.java       # Mockito unit tests
│       │   ├── StudentApiApplicationTests.java       # Spring Context test
│       │   └── StudentTest.java                      # Entity & Validation test
│       └── resources/
│           └── application.yml     # Cấu hình test H2 in-memory
├── .gitignore
├── build.gradle                    # Cấu hình build & dependencies
├── gradlew                         # Script chạy trên Linux/macOS
├── gradlew.bat                     # Script chạy trên Windows
├── settings.gradle
└── README.md
```

---

## 4. Chi tiết tệp kịch bản `.github/workflows/ci-matrix.yml`

```yaml
name: Java Spring Boot CI Matrix

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  test-matrix:
    name: Test on Java ${{ matrix.java-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: ['17', '21']
    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v5

      - name: Set up JDK ${{ matrix.java-version }}
        uses: actions/setup-java@v5
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Run unit tests with Gradle
        run: ./gradlew test
```

### Phân tích các thành phần cấu hình chính:

| Khóa cấu hình | Ý nghĩa & Vai trò |
| :--- | :--- |
| `on.push.branches: [main]` | Kích hoạt kịch bản tự động mỗi khi có commit mới được đẩy lên nhánh `main`. |
| `strategy.matrix` | Khai báo ma trận biến thể. GitHub Actions sẽ tự động nhân bản (fan-out) job thành nhiều sub-job độc lập. |
| `java-version: ['17', '21']` | Mảng chứa các phiên bản JDK mục tiêu cần kiểm thử song song. |
| `name: Test on Java ${{ matrix.java-version }}` | Đặt tên hiển thị động cho từng sub-job: `Test on Java 17` và `Test on Java 21`. |
| `uses: actions/checkout@v5` | Kéo mã nguồn từ GitHub repository về máy ảo runner. |
| `uses: actions/setup-java@v5` | Tải và cài đặt phiên bản Java tương ứng từ nhà phân phối Eclipse Temurin (Adoptium). |
| `cache: 'gradle'` | Kích hoạt cơ chế tự động caching các file `.gradle` và dependency JARs, tăng tốc độ thực thi cho các lần chạy tiếp theo. |
| `chmod +x gradlew` | Cấp quyền thực thi cho Gradle Wrapper script trên môi trường Linux Ubuntu Runner. |
| `./gradlew test` | Thực thi các bài kiểm thử tự động của dự án thông qua Gradle. |

---

## 5. Kết quả kiểm thử cục bộ (Local Testing)

Trước khi đẩy lên GitHub, dự án đã được thực thi và kiểm thử trực tiếp bằng lệnh `./gradlew test`:

```text
> Task :compileJava
> Task :processResources
> Task :classes
> Task :compileTestJava
> Task :processTestResources
> Task :testClasses
> Task :test

BUILD SUCCESSFUL in 34s
5 actionable tasks: 5 executed
```

### Danh sách các bộ test đã vượt qua:
1. **`com.example.studentapi.StudentTest`**:
   - `testStudentBuilderAndGetters`: Kiểm thử tính đúng đắn của Lombok Builder và getter fields.
   - `testStudentSetters`: Kiểm thử các phương thức setter và gán giá trị hợp lệ.
2. **`com.example.studentapi.service.StudentServiceTest`**:
   - `testCreateStudent`: Kiểm thử tạo mới sinh viên thông qua Mockito service layer.
   - `testGetStudentFound`: Kiểm thử tìm kiếm sinh viên theo ID khi tồn tại.
   - `testGetStudentNotFound`: Kiểm thử tìm kiếm sinh viên theo ID khi không tồn tại.
   - `testDeleteStudent`: Kiểm thử xóa sinh viên theo ID.
3. **`com.example.studentapi.StudentApiApplicationTests`**:
   - `contextLoads`: Kiểm thử nạp Spring Context thành công trên cơ sở dữ liệu in-memory H2.

---

## 6. Hướng dẫn đẩy lên GitHub và kiểm tra kết quả

### Bước 1: Tạo Repository trên GitHub
1. Truy cập [https://github.com/new](https://github.com/new).
2. Đặt tên repository: `IT209_Session12_Ex1`.
3. Để chế độ **Public** và không tích chọn khởi tạo README/.gitignore (vì đã có sẵn mã nguồn cục bộ).

### Bước 2: Commit và Push mã nguồn
Mở terminal tại thư mục `E:\BaiTap\IT209_Devops\Session12\Ex1` và chạy:
```bash
git add .
git commit -m "feat: Xay dung quy trinh CI da phien ban JDK voi Matrix Strategy"
git remote add origin git@github.com:ThanhDZsudo/IT209_Session12_Ex1.git
git push -u origin main
```
*(Nếu sử dụng HTTPS: thay `git@github.com:ThanhDZsudo/IT209_Session12_Ex1.git` bằng `https://github.com/ThanhDZsudo/IT209_Session12_Ex1.git`)*

### Bước 3: Xác nhận trên GitHub Actions
1. Truy cập tab **Actions** tại repository: `https://github.com/ThanhDZsudo/IT209_Session12_Ex1/actions`.
2. Quan sát workflow `Java Spring Boot CI Matrix` tự động rẽ thành 2 sub-job chạy song song:
   - `Test on Java 17` ✅
   - `Test on Java 21` ✅
3. Chụp ảnh màn hình hiển thị cả 2 sub-job tích xanh để hoàn tất sản phẩm nộp bài.
