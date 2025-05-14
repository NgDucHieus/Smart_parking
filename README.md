# Nhận diện và Lưu Khuôn mặt từ Webcam trên Google Colab

Mã nguồn này cho phép nhận diện khuôn mặt và mắt người trong thời gian thực từ luồng video webcam trực tiếp trong môi trường Google Colaboratory. Nó sử dụng OpenCV Haar Cascades để phát hiện và có thể tự động lưu các vùng khuôn mặt được phát hiện vào bộ nhớ tạm thời của Colab hoặc Google Drive của bạn.

## Tính năng

* **Luồng Webcam Thời gian thực trong Colab**: Hiển thị video trực tiếp từ webcam của bạn trong output của cell Colab.
* **Nhận diện Khuôn mặt**: Sử dụng `haarcascade_frontalface_default.xml` của OpenCV để phát hiện khuôn mặt.
* **Nhận diện Mắt**: Sử dụng `haarcascade_eye.xml` của OpenCV để phát hiện mắt trong các vùng khuôn mặt đã nhận diện.
* **Hiển thị Trực quan**: Vẽ hình chữ nhật (bounding box) xung quanh khuôn mặt và mắt được phát hiện trực tiếp trên luồng video.
* **Lưu Khuôn mặt**: Tự động lưu ảnh crop của các khuôn mặt được phát hiện dưới dạng file JPG.
* **Cấu hình Lưu trữ Linh hoạt**:
    * Lưu vào thư mục tạm thời của Colab (mặc định, sẽ mất khi runtime reset).
    * Tùy chọn lưu vào thư mục tùy chỉnh trên Google Drive (yêu cầu mount Drive).
* **Điều chỉnh Khoảng thời gian Lưu**: Cấu hình tần suất lưu khuôn mặt (ví dụ: lưu mỗi N frame có khuôn mặt).
* **Giới hạn Xử lý**: Tự động dừng sau một số lượng khung hình tối đa được xử lý.
* **Tùy chỉnh Tham số Nhận diện**: Cho phép điều chỉnh các tham số `scaleFactor`, `minNeighbors`, `minSize` cho cả bộ nhận diện khuôn mặt và mắt để tinh chỉnh độ nhạy và hiệu suất.
* **Tải Tệp Cascade Tự động**: Tự động tải các tệp XML Haar Cascade cần thiết từ kho lưu trữ OpenCV nếu chúng chưa tồn tại.
* **Giao diện Người dùng Tương tác**:
    * Hiển thị trạng thái (đang khởi tạo, đang nhận diện, số khuôn mặt, lỗi camera).
    * Cho phép người dùng dừng luồng video bằng cách nhấp vào video hoặc một dòng chữ hướng dẫn.
* **Xử lý Lỗi và Dọn dẹp**: Bao gồm các khối `try-except` để xử lý lỗi và một hàm dọn dẹp để giải phóng các tài nguyên JavaScript khi kết thúc.

## Yêu cầu

* Môi trường Google Colaboratory.
* Trình duyệt web có quyền truy cập webcam.
* Các thư viện Python (thường đã được cài sẵn trên Colab):
    * `opencv-python` (cv2)
    * `requests`
    * `Pillow` (PIL.Image)
    * `numpy`
    * `IPython`
    * `google-colab`

## Cài đặt và Thiết lập

1.  **Mở trong Google Colab**: Tải file `.ipynb` này lên hoặc mở trực tiếp trong Google Colab.
2.  **Kết nối Google Drive (Tùy chọn)**:
    * Nếu bạn muốn lưu khuôn mặt vào Google Drive, hãy tìm đến phần `--- 3. Google Drive Mount (Tùy chọn) ---`.
    * Bỏ comment dòng `drive.mount('/content/drive')`.
    * Chạy cell đó và làm theo hướng dẫn để cấp quyền truy cập Google Drive cho Colab.

## Cách sử dụng và Cấu hình

Trước khi chạy toàn bộ script, bạn có thể tùy chỉnh các thông số trong phần `--- 4. Configuration / Cấu hình ---`:

1.  **Chọn Thư mục Lưu trữ (`SAVE_FOLDER_PATH`)**:
    * **Mặc định (Colab)**:
        ```python
        SAVE_FOLDER_PATH = "/content/khuon_mat_da_luu_colab"
        ```
        Khuôn mặt sẽ được lưu vào thư mục này trong runtime hiện tại của Colab. Dữ liệu sẽ bị xóa khi runtime được reset.
    * **Google Drive**:
        * Đảm bảo bạn đã thực hiện bước "Kết nối Google Drive" ở trên.
        * Đặt tên thư mục mong muốn trên Drive:
            ```python
            GDRIVE_FOLDER_NAME = "KhuonMatDaLuuTrenDrive" # Hoặc tên bạn chọn
            ```
        * Bỏ comment và sử dụng đường dẫn sau cho `SAVE_FOLDER_PATH`:
            ```python
            # SAVE_FOLDER_PATH = os.path.join(GDRIVE_MOUNT_PATH, "MyDrive", GDRIVE_FOLDER_NAME)
            ```
            (Nhớ comment dòng `SAVE_FOLDER_PATH` của Colab).

2.  **Khoảng cách giữa các lần lưu (`SAVE_INTERVAL`)**:
    * Số khung hình có phát hiện khuôn mặt cần chờ trước khi lưu lại.
    * Ví dụ: `SAVE_INTERVAL = 10` nghĩa là nếu có khuôn mặt được phát hiện, nó sẽ được lưu, và sau đó script sẽ đợi ít nhất 10 frame tiếp theo *có khuôn mặt* trước khi lưu lại.
    * Đặt là `1` để lưu mọi frame có khuôn mặt.

3.  **Số khung hình tối đa (`MAX_FRAMES_TO_PROCESS`)**:
    * Giới hạn số lượng khung hình được xử lý để tránh chạy vô hạn. Ví dụ: `500`.

4.  **Tham số Nhận diện Haar Cascade**:
    * `FACE_SCALE_FACTOR`, `FACE_MIN_NEIGHBORS`, `FACE_MIN_SIZE`: Điều chỉnh cho bộ nhận diện khuôn mặt.
        * `scaleFactor`: Mức độ thu nhỏ ảnh ở mỗi bước quét. Giá trị nhỏ hơn (ví dụ: `1.05`) chậm hơn nhưng có thể phát hiện tốt hơn.
        * `minNeighbors`: Số lượng "hàng xóm" mà mỗi hình chữ nhật ứng viên phải có để được coi là một phát hiện. Giá trị cao hơn làm giảm số phát hiện giả nhưng có thể bỏ sót khuôn mặt.
        * `minSize`: Kích thước đối tượng tối thiểu (pixel) để được phát hiện.
    * `EYE_SCALE_FACTOR`, `EYE_MIN_NEIGHBORS`, `EYE_MIN_SIZE`: Tương tự cho bộ nhận diện mắt.

### Chạy Script

1.  Sau khi đã cấu hình xong, chạy tất cả các cell trong notebook theo thứ tự từ trên xuống dưới.
2.  Script sẽ:
    * Tạo thư mục lưu trữ (nếu chưa có).
    * Tải các tệp Haar Cascade nếu cần.
    * Khởi tạo luồng video từ webcam. Trình duyệt của bạn sẽ yêu cầu quyền truy cập camera. **Hãy cho phép (Allow)**.
3.  Một cửa sổ video sẽ xuất hiện trong output của cell, hiển thị luồng webcam với các hình chữ nhật đánh dấu khuôn mặt (màu xanh dương) và mắt (màu xanh lá). Trạng thái nhận diện cũng sẽ được hiển thị.
4.  Các ảnh khuôn mặt được crop sẽ được lưu vào thư mục đã cấu hình.

### Dừng Script

* **Cách 1**: Nhấp chuột vào vùng video đang hiển thị hoặc vào dòng chữ "Nhấp vào video hoặc dòng chữ này để dừng."
* **Cách 2**: Script sẽ tự động dừng khi đạt `MAX_FRAMES_TO_PROCESS`.
* **Cách 3**: Ngắt kernel của Colab (ví dụ: bằng cách nhấn Ctrl+C trong output của cell đang chạy hoặc chọn "Interrupt execution" từ menu Runtime).

## Chi tiết Hoạt động (Tóm tắt)

1.  **JavaScript cho Webcam**:
    * `video_stream()`: Thiết lập giao diện HTML (thẻ `video`, `canvas`, `img` cho overlay, `span` cho label) và JavaScript để lấy luồng từ webcam.
    * Hàm `stream_frame(label, imgData)` (trong JS): Được gọi từ Python. Nó cập nhật label trạng thái, hiển thị ảnh `imgData` (chứa bounding box) lên trên video, và chụp một frame mới từ webcam. Frame này được trả về Python dưới dạng chuỗi base64 JPEG.
    * `removeDom()` (trong JS): Dọn dẹp các phần tử DOM khi dừng.
2.  **Python Backend**:
    * `video_frame(label, bbox_data_url)`: Gọi hàm `stream_frame` của JavaScript thông qua `eval_js` và nhận về dữ liệu ảnh.
    * `js_to_image(js_reply_img)`: Chuyển đổi ảnh base64 từ JS thành ảnh OpenCV (NumPy array).
    * `bbox_to_bytes(bbox_array)`: Chuyển đổi ảnh overlay (NumPy array RGBA chứa bounding box) thành chuỗi data URL base64 PNG (PNG để giữ độ trong suốt) để gửi cho JavaScript.
    * `run_face_detection_and_save()`:
        * Tải và khởi tạo các bộ phân loại Haar Cascade.
        * Vòng lặp chính:
            * Lấy frame từ webcam thông qua `video_frame`.
            * Chuyển đổi sang ảnh xám và cân bằng histogram để cải thiện nhận diện.
            * Sử dụng `face_cascade.detectMultiScale()` để tìm khuôn mặt.
            * Với mỗi khuôn mặt:
                * Vẽ hình chữ nhật lên một ảnh overlay trong suốt (`bbox_overlay_rgba`).
                * Lưu ảnh khuôn mặt nếu đủ điều kiện `SAVE_INTERVAL`.
                * Chạy `eye_cascade.detectMultiScale()` trên vùng khuôn mặt để tìm mắt.
                * Vẽ hình chữ nhật cho mắt lên ảnh overlay.
            * Chuyển đổi ảnh overlay thành data URL và gửi lại cho `video_frame` để hiển thị.
        * Xử lý dừng và dọn dẹp.

## Lưu ý

* Hiệu suất có thể bị ảnh hưởng bởi chất lượng webcam, điều kiện ánh sáng, và tài nguyên CPU/GPU được cấp phát cho Colab.
* Các bộ nhận diện Haar Cascade nhạy cảm với góc nghiêng của khuôn mặt và điều kiện ánh sáng.
* Dữ liệu lưu trong `/content/` (bộ nhớ tạm thời của Colab) sẽ bị mất khi runtime kết thúc hoặc reset. Sử dụng Google Drive để lưu trữ lâu dài.
* Nếu gặp lỗi "Lỗi camera! Kiểm tra quyền hoặc kết nối.", hãy đảm bảo trình duyệt của bạn có quyền truy cập webcam và không có ứng dụng nào khác đang sử dụng webcam.

## Khắc phục sự cố

* **Không thấy video / Lỗi camera**:
    * Kiểm tra xem bạn đã cấp quyền truy cập camera cho trình duyệt chưa.
    * Đảm bảo không có ứng dụng nào khác đang sử dụng webcam.
    * Thử reset runtime của Colab và chạy lại.
* **Nhận diện kém**:
    * Thử điều chỉnh các tham số Haar Cascade (ví dụ: giảm `minNeighbors`, thay đổi `scaleFactor`).
    * Đảm bảo đủ ánh sáng và khuôn mặt nhìn thẳng vào camera.
* **Lỗi khi lưu vào Google Drive**:
    * Đảm bảo bạn đã mount Google Drive thành công và đường dẫn `SAVE_FOLDER_PATH` là chính xác.
    * Kiểm tra dung lượng trống trên Google Drive.

Chúc bạn thành công với dự án nhận diện khuôn mặt!
