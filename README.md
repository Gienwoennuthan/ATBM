
# Đề tài 11: Mô phỏng Upload/Download Video lên Cloud với Xử lý Lỗi Mạng

## Mô tả dự án

Ứng dụng này mô phỏng quá trình upload và download video lên hệ thống lưu trữ đám mây (giả lập Google Cloud Storage). Quá trình truyền dữ liệu được thực hiện thông qua giao tiếp socket TCP (giả lập qua HTTP) với các tính năng bảo mật như mã hóa bằng AES-CBC, ký số RSA với hàm băm SHA-512 và cơ chế xử lý lỗi mạng như mất gói tin, timeout. Hệ thống được thiết kế để đảm bảo dữ liệu luôn toàn vẹn và có thể tự động retry khi lỗi xảy ra.

## Công nghệ sử dụng

Ứng dụng sử dụng Python và Flask cho backend. Giao diện người dùng được xây dựng bằng HTML, CSS và JavaScript với Bootstrap 5. Mã hóa được thực hiện bằng thuật toán AES-CBC với khóa phiên, kết hợp ký số bằng RSA 2048-bit và kiểm tra toàn vẹn bằng SHA-512. Giao tiếp giữa client và cloud được mô phỏng dưới dạng truyền socket và các lỗi mạng được kiểm tra bằng xác suất.

## Cài đặt và chạy ứng dụng

Bước 1: Cài đặt các thư viện cần thiết bằng lệnh:

```
pip install -r requirements.txt
```

Bước 2: Chạy ứng dụng bằng lệnh:

```
python app.py
```

Bước 3: Truy cập ứng dụng qua trình duyệt tại các địa chỉ:

* Trang chủ: [http://localhost:5000](http://localhost:5000)
* Giao diện Client (người gửi): [http://localhost:5000/client](http://localhost:5000/client)
* Giao diện Cloud (người nhận): [http://localhost:5000/cloud](http://localhost:5000/cloud)

## Luồng xử lý chính

Bước 1: Handshake
Client gửi thông điệp "Hello!" tới cloud, cloud trả lời "Ready!". Trong quá trình này có thể xảy ra lỗi với xác suất 15% và hệ thống sẽ tự động thực hiện retry tối đa ba lần.

Bước 2: Xác thực và trao khóa
Client ký metadata bằng RSA/SHA-512 và mã hóa session key bằng RSA-2048 trước khi gửi lên cloud. Nếu xảy ra lỗi mạng, hệ thống sẽ retry tối đa ba lần.

Bước 3: Mã hóa và upload
File video được mã hóa bằng AES-CBC với một IV ngẫu nhiên. Hệ thống tính hash toàn bộ dữ liệu và ký bằng RSA để đảm bảo tính toàn vẹn. Gói tin bao gồm IV, dữ liệu mã hóa, hash và chữ ký sẽ được gửi qua mạng. Nếu có lỗi như mất gói (20%) hoặc timeout (10%), hệ thống sẽ retry tự động.

Bước 4: Download
Người dùng yêu cầu download file qua giao diện cloud. Cloud kiểm tra chữ ký xác thực, nếu hợp lệ sẽ cung cấp đường dẫn để tải về. Nếu không hợp lệ, cloud sẽ từ chối yêu cầu.

## Tính năng chính

Ứng dụng hỗ trợ mã hóa file bằng AES-CBC với khóa phiên ngẫu nhiên, ký số metadata bằng RSA 2048-bit với SHA-512, và kiểm tra toàn vẹn dữ liệu bằng hash SHA-512. Trong quá trình truyền tải dữ liệu, các lỗi mạng như mất gói và timeout được mô phỏng và xử lý bằng cơ chế retry. Hệ thống sử dụng giao diện web đơn giản và trực quan, chia rõ hai phần: client và cloud.

## Cấu trúc thư mục

```
.
├── app.py                # Backend chính sử dụng Flask
├── requirements.txt      # Danh sách thư viện cần cài
├── static/               # Tài nguyên tĩnh (CSS, JS, file upload)
│   └── cloud_storage/    # Thư mục chứa các file đã upload
├── templates/            # Các giao diện HTML
│   ├── index.html        # Trang chủ
│   ├── client.html       # Giao diện dành cho người gửi
│   └── cloud.html        # Giao diện dành cho người nhận
└── README.md             # Tài liệu hướng dẫn sử dụng
```

## API endpoints

Phía client:

* POST `/handshake`: Thiết lập kết nối ban đầu với cloud
* POST `/auth_key_exchange`: Xác thực metadata và trao đổi khóa AES
* POST `/upload_file`: Upload file đã mã hóa

Phía cloud:

* GET `/list_files`: Lấy danh sách các file đã upload
* GET `/download/<filename>`: Tải file từ cloud về
* POST `/download_file`: Yêu cầu tải file với xác thực

## Ghi chú khi sử dụng

Ứng dụng phù hợp với việc thử nghiệm các file có dung lượng vừa phải (dưới 100MB). Các lỗi mạng được mô phỏng có thể làm chậm quá trình upload, do đó log hiển thị thời gian thực sẽ giúp người dùng theo dõi trạng thái. Các file upload được lưu tại thư mục `static/cloud_storage/`.



