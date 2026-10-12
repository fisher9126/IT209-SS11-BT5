# Bài 5: Nối Job Build & Push Docker Image sau khi Test Gradle xanh

## Thiết kế

Workflow `ci.yml` có 2 job:

| Job | Nhiệm vụ |
| --- | --- |
| `test` | Cài JDK 17 (có cache Gradle), chạy `./gradlew test`, đóng gói JAR bằng `./gradlew bootJar -x test` |
| `docker-build` | Đăng nhập `ghcr.io`, build và push Docker image bằng `docker/build-push-action@v5` |

`docker-build` khai báo `needs: test`, nên chỉ chạy khi `test` thành công. Nếu `test` đỏ, `docker-build` tự chuyển sang `Skipped`.

```yaml
docker-build:
  needs: test
  runs-on: ubuntu-latest
```

## Giải thích

- Image được đặt tên `ghcr.io/<owner>/<repo>`, tên được chuyển về chữ thường vì registry không nhận chữ hoa.
- Hai tag: `latest` và `${{ github.sha }}` (truy vết được image ứng với commit nào).
- Đăng nhập dùng secret `REGISTRY_TOKEN` (đã tạo ở Bài 4).
- Dockerfile dạng multi-stage tự build JAR bên trong, nên `docker-build` không cần lấy file từ job `test` (hai job không dùng chung filesystem).

## Kiểm tra

1. Push lên `main`, mở tab Actions. Biểu đồ Workflow hiển thị mũi tên từ `test` sang `docker-build`. Chụp ảnh lại.
2. Cố ý làm hỏng một JUnit test, push lại. Job `test` đỏ, job `docker-build` hiện `Skipped`.
3. Sửa lại test, push để trả workflow về xanh.

## Sản phẩm nộp

- `ci.yml` (2 job liên kết qua `needs: test`)
- Ảnh chụp biểu đồ Workflow hiển thị 2 job nối nhau
- Ảnh job `docker-build` ở trạng thái Skipped (tuỳ chọn, làm rõ hơn)
