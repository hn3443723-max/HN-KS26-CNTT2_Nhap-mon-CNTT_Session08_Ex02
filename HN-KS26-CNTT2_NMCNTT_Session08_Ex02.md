# Bài 2 : Chuẩn hóa tài liệu cài đặt hệ thống
## **Yêu cầu đầu ra**

### Báo cáo phân tích (I/O):
+ Input (văn bản thô): Hướng dẫn cài đặt cảm biến độ ẩm cho trang trại Lan;Phân biệt rõ văn bản giải thích và câu lệnh Terminal;Sử dụng inline code, code block Python màu sắc, bảng so sánh cảm biến cũ/mới; Tuân thủ quy tắc escape ký tự đặc biệt. 
+ Output (giao diện render chuyên nghiệp):Cấu trúc tài liệu rõ ràng với các phần: Chuẩn bị môi trường, Cài đặt hệ thống, Code điều khiển tự động, Bảng so sánh thiết bị, và Danh sách kiểm tra (Task List).

 ### Source code:
 # Tài liệu hướng dẫn cài đặt cảm biến độ ẩm cho trang trại Lan

Chào mừng kỹ thuật viên và chủ trang trại Lan đến với tài liệu cấu hình hệ thống giám sát và tưới tự động. Tài liệu này giúp bạn thiết lập phần cứng và chạy mã điều khiển máy bơm thông minh.

---

## Bước 1: Chuẩn bị hệ thống và cài đặt thư viện
```bash
sudo apt-get update && sudo apt-get upgrade -y
pip install Adafruit-DHT RPi.GPIO
```
---

## Bước 2: So sánh loại cảm biến (Cũ vs Mới)


| Tiêu chí | Cảm biến độ ẩm cũ (YL-69 điện trở) | Cảm biến độ ẩm mới (Điện dung chống ăn mòn) |
| :--- | :--- | :--- |
| **Nguyên lý đo** | Dẫn điện trực tiếp qua 2 bản cực kim loại | Đo điện dung (không tiếp xúc trực tiếp ion nước) |
| **Độ bền thực tế** | Dễ bị oxi hóa, gỉ sét sau 3 - 6 tháng trong môi trường ẩm | Chống ăn mòn tốt, tuổi thọ trên 2 năm |
| **Độ chính xác** | Thấp, dễ sai số do khoáng chất trong phân bón Lan | Cao và ổn định hơn |
| **Giá thành** | Rẻ ($\sim 15.000$ đến $25.000$ VNĐ) | Trung bình ($\sim 50.000$ đến $80.000$ VNĐ) |

---

## Bước 3: Mã nguồn Python điều khiển máy bơm tự động

```python
import time
import RPi.GPIO as GPIO

# Khai báo cấu hình chân kết nối
RELAY_PIN = 17 
# Chân điều khiển máy bơm qua Relay
MOISTURE_THRESHOLD = 60  # Ngưỡng kích hoạt tưới (%)

GPIO.setmode(GPIO.BCM)
GPIO.setup(RELAY_PIN, GPIO.OUT)
GPIO.output(RELAY_PIN, GPIO.HIGH)  # Mặc định tắt bơm (Active Low tùy loại relay)

def doc_do_am_gia_the():
    # Hàm giả lập đọc giá trị từ cảm biến ADC / I2C
    # Trong thực tế gọi thư viện đọc từ chân Analog/I2C
    return 45  # Ví dụ độ ẩm hiện tại là 45%

try:
    print("Bắt đầu giám sát độ ẩm trang trại Lan...")
    while True:
        do_am_hien_tai = doc_do_am_gia_the()
        print(f"Độ ẩm hiện tại: {do_am_hien_tai}%")

        if do_am_hien_tai < MOISTURE_THRESHOLD:
            print("Cảnh báo: Giá thể khô! Đang bật máy bơm...")
            GPIO.output(RELAY_PIN, GPIO.LOW)  # Bật máy bơm
            time.sleep(10)  # Tưới trong 10 giây
            GPIO.output(RELAY_PIN, GPIO.HIGH) # Tắt máy bơm
            print("Đã tưới xong, chờ chu kỳ kiểm tra tiếp theo...")
        
        time.sleep(30)

except KeyboardInterrupt:
    print("Đã dừng chương trình thủ công.")
finally:
    GPIO.cleanup()
```

---

## Bước 4: Task List kiểm tra thiết bị trước khi vận hành

- [ ] Kiểm tra nguồn điện cấp cho Relay và Raspberry Pi (đảm bảo ổn định 5V/2A).
- [ ] Xác nhận chân tín hiệu `GPIO 17` đã kết nối đúng với cực IN của Relay.
- [ ] Cắm đầu dò cảm biến điện dung ngập đúng vạch kỹ thuật vào chậu Lan mẫu.
- [ ] Chạy thử lệnh kiểm tra file log trên Terminal:
  ```bash
  tail -f /var/log/lan_sensor.log
  ```
- [ ] Xác nhận máy bơm hoạt động đúng chu kỳ bật/tắt khi độ ẩm thay đổi.


