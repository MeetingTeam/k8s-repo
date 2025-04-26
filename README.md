Hướng dẫn sử dụng Helm với GitHub Private Repository
Tài liệu này hướng dẫn quy trình sử dụng Helm để cài đặt các tài nguyên từ một GitHub repository private.
Quy trình cài đặt
Bước 1: Thêm repository Helm
Thêm một repository chứa chart vào Helm, sử dụng Personal Access Token (PAT) để truy cập vào GitHub repository private.
bashhelm repo add <repo-name> https://<PAT>@raw.githubusercontent.com/MeetingTeam/k8s-repo/main
Lưu ý: Thay thế <repo-name> bằng tên bạn muốn đặt cho repository và <PAT> bằng GitHub Personal Access Token của bạn.
Bước 2: Tìm kiếm các charts có sẵn
Liệt kê tất cả các charts có sẵn trong repository đã thêm:
bashhelm search repo <repo-name>
Lệnh này sẽ hiển thị danh sách các charts cùng với phiên bản và mô tả ngắn.
Bước 3: Tải cấu hình mặc định của tài nguyên
Lấy các cấu hình mặc định của chart và lưu vào file values.custom.yaml để chỉnh sửa:
bashhelm show values <repo-name>/<tài nguyên cần cài> > values.custom.yaml
Ví dụ:
bashhelm show values devops/jenkins > values.custom.yaml
Sau khi thực hiện lệnh này, bạn có thể chỉnh sửa file values.custom.yaml để tùy chỉnh cài đặt theo nhu cầu.
Bước 4: Cài đặt tài nguyên
Cài đặt tài nguyên từ chart với các cấu hình tùy chỉnh từ file values.custom.yaml:
bashhelm install <release-name> <repo-name>/<tài nguyên cần cài> -f values.custom.yaml
Ví dụ:
bashhelm install jenkins devops/jenkins -f values.custom.yaml
Các lệnh hữu ích khác

Cập nhật repository:
bashhelm repo update

Kiểm tra trạng thái của release:
bashhelm status <release-name>

Gỡ cài đặt release:
bashhelm uninstall <release-name>

Xem các releases đã cài đặt:
bashhelm list

Nâng cấp release với cấu hình mới:
bashhelm upgrade <release-name> <repo-name>/<tài nguyên> -f values.custom.yaml


Khắc phục sự cố

Nếu gặp lỗi xác thực khi thêm repository, hãy đảm bảo Personal Access Token có đầy đủ quyền truy cập vào repository.
Đảm bảo cài đặt kubectl và cấu hình đúng context trước khi sử dụng Helm.
Kiểm tra logs nếu quá trình cài đặt thất bại:
bashkubectl logs -n <namespace> <pod-name>
