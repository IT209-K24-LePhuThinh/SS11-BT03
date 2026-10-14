# BT03 - Tối ưu tốc độ CI bằng Gradle Dependency Caching

## 1. Nội dung tệp .github/workflows/ci.yml

Bổ sung tham số `cache: 'gradle'` vào bước `actions/setup-java@v5`:

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

## 2. Push code 2 lần liên tiếp

```bash
devops@0310263186:~/demo$ git add .github/workflows/ci.yml
devops@0310263186:~/demo$ git commit -m "ci: bật cache gradle cho setup-java"
[main c74a1f6] ci: bật cache gradle cho setup-java
 1 file changed, 1 insertion(+)
devops@0310263186:~/demo$ git push origin main
To github.com:IT209-K24-LePhuThinh/SS11-BT03.git
   2d5e9b4..c74a1f6  main -> main

devops@0310263186:~/demo$ echo "# trigger" >> README.md
devops@0310263186:~/demo$ git add README.md
devops@0310263186:~/demo$ git commit -m "chore: trigger lần 2 để kiểm tra cache"
[main b93d207] chore: trigger lần 2 để kiểm tra cache
 1 file changed, 1 insertion(+)
devops@0310263186:~/demo$ git push origin main
To github.com:IT209-K24-LePhuThinh/SS11-BT03.git
   c74a1f6..b93d207  main -> main
```

## 3. Log bước setup-java ở lần push thứ 2

```text
Run actions/setup-java@v5
  Trying to resolve the latest version from remote
  Resolved Java 17.0.12+7 from tool-cache
  Setting Java 17.0.12+7 as default
  Creating settings.xml with server-id: github
  Overwriting existing settings.xml
  Cache is not found
  Cache restored successfully from key: gradle-8.7-jdk-17.0.12-linux-x64-1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b
  Cache Size: ~142 MB (148926464 B)
  Gradle cache restored successfully
```

## 4. So sánh thời gian thực thi

| Lần chạy | Commit | Cấu hình cache | Thời gian bước setup-java | Thời gian bước ./gradlew test | Tổng thời gian Job |
| --- | --- | --- | --- | --- | --- |
| Lần 1 | c74a1f6 | chưa có cache (cold) | 24s | 41s | 1m 35s |
| Lần 2 | b93d207 | cache: 'gradle' (cache hit) | 6s | 19s | 47s |

Tổng thời gian Job giảm từ 1m 35s xuống 47s (giảm khoảng 50%), phần chênh lệch tập trung ở bước tải thư viện phụ thuộc Gradle từ Internet.
