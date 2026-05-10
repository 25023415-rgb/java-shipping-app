# Hướng dẫn tạo Repository, đẩy code và chạy CI/CD trên GitHub (Bài 10)

Để chạy được quy trình CI/CD cho bài 10, cần đưa toàn bộ mã nguồn lên GitHub. Dưới đây là các bước thực hiện:

**Bước 1: Tạo kho chứa (Repository) trống trên GitHub**
- Đăng nhập vào GitHub, bấm dấu **+** ở góc trên cùng bên phải và chọn **New repository**.
- Nhập tên repository (ví dụ: `java-shipping-app`), để chế độ Public.
- **Lưu ý cực kỳ quan trọng:** Để nguyên mọi thứ, không tick vào mục add `README` hay `.gitignore`. Repository phải trống hoàn toàn để tránh xung đột khi đẩy code từ máy lên. Sau đó nhấn **Create repository**.

**Bước 2: Dùng Git để đẩy mã nguồn từ máy lên GitHub**
- Mở Terminal (hoặc Git Bash / CMD) ngay tại thư mục chứa code (nơi có file `pom.xml` và thư mục `.github`).
- Gõ lần lượt các lệnh sau:
  ```bash
  # Khởi tạo git trong thư mục
  git init
  
  # Thêm toàn bộ file vào stage
  git add .
  
  # Commit lần đầu tiên
  git commit -m "Initial commit - Bài 10"
  
  # Chuyển tên nhánh chính thành main
  git branch -M main
  
  # Kết nối với repository trống vừa tạo trên Github
  # (Nhớ thay bằng link repo thực tế)
  git remote add origin https://github.com/TENTAIKHOAN/java-shipping-app.git
  
  # Đẩy code lên GitHub
  git push -u origin main
  ```

**Bước 3: Hóng kết quả Pipeline trên GitHub Actions**
- Vừa đẩy code lên xong, quay lại trang repo trên GitHub và bấm sang tab **Actions**.
- Ngay lập tức sẽ thấy GitHub nhận diện được file `.github/workflows/ci.yml` và tự động chạy một job có tên là "build".
- Click vào workflow đang chạy để xem trực tiếp từng dòng log (Execution log). Cứ mỗi khi pipeline báo đỏ ❌, vào đọc log xem báo lỗi ở dòng nào để tìm cách bắt bệnh và sửa lỗi.
