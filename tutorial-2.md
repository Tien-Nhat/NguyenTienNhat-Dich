# Triển khai Trạm Thời Tiết IoT và Kiến Trúc Dữ Liệu Kết Nối

## Giới Thiệu

Trong bài hướng dẫn này, bạn sẽ tạo một Trạm Thời Tiết IoT với một Raspberry Pi và Sense HAT. Ngoài ra, bạn sẽ thêm phân tích dữ liệu cho Nền Tảng Trạm Thời Tiết IoT này với giao tiếp Kiến Trúc Dữ Liệu Kết Nối giữa MiNiFi, HDF Sandbox và HDP Sandbox.

## Tiền Điều Kiện

- Đã kích hoạt Kiến Trúc Dữ Liệu Kết Nối

## Mục Lục

- [Bước 1: Thiết Lập Raspberry Pi Weather Station Node](#bước-1-thiết-lập-raspberry-pi-weather-station-node)
- [Bước 2: Cấu Hình Bridged Adapter Network cho VirtualBox](#bước-2-cấu-hình-bridged-adapter-network-cho-virtualbox)
- [Bước 3: Ánh Xạ Bridged IP đến Hostname Mong Muốn trong File hosts](#bước-3-ánh-xạ-bridged-ip-đến-hostname-mong-muốn-trong-file-hosts)
- [Bước 4: Xác Nhận Đã Đáp Ứng Các Tiền Điều Kiện](#bước-4-xác-nhận-đã-đáp-ứng-các-tiền-điều-kiện)
- [Bước 5: Khởi Động HDF Sandbox và Cấu Hình NiFi Site-To-Site](#bước-5-khởi-động-hdf-sandbox-và-cấu-hình-nifi-site-to-site)
- [Tóm Tắt](#tóm-tắt)
- [Đọc Thêm](#đọc-thêm)
- [Phụ Lục A: Cài Đặt Hệ Điều Hành Raspbian lên Raspberry Pi](#phụ-lục-a-cài-đặt-hệ-điều-hành-raspbian-lên-raspberry-pi)
- [Phụ Lục B: Đặt Lại Thời Gian và Ngày cho Raspberry Pi](#phụ-lục-b-đặt-lại-thời-gian-và-ngày-cho-raspberry-pi)

### Bước 1: Thiết Lập Raspberry Pi Weather Station Node

#### 1.1: Kết Nối Sense HAT với Raspberry Pi

1. Kết nối 40 chân cái của Sense HAT với 40 chân nam của Raspberry Pi.

![sense-hat-pins](assets/tutorial2/sense-hat-pins.jpg)

**Hình 1:** Chân cái 40 của Sense HAT

![raspberry-pi-pins](assets/tutorial2/raspberry-pi-pins.jpg)

**Hình 2:** Chân nam 40 của Raspberry Pi

![iot-device-pins-connected](assets/tutorial2/iot-device-pins-connected.jpg)

**Hình 3:** Chân cái của Sense HAT được kết nối với chân nam của Raspberry Pi

> Lưu ý: Nếu bạn chưa cài đặt Hệ Điều Hành Raspbian trên Raspberry Pi, tham khảo phần **Phụ Lục A**.

#### 1.2: Cung Cấp Nguồn Cho Raspberry Pi và Thiết Lập Kết Nối Internet

Kết nối Raspberry Pi với nguồn bằng cáp micro USB (có thể là sạc điện thoại) để cung cấp điện và sử dụng cáp Ethernet để kết nối với Internet.

![power-ethernet_rpi](assets/tutorial2/power-ethernet_rpi.png)

**Hình 4:** Cáp Ethernet của Raspberry Pi Được Kết Nối Để Truy Cập Internet

#### 1.3: SSH vào Raspberry Pi

Sử dụng **Adafruit's Pi Finder** để phát hiện địa chỉ IP của Raspberry Pi, giúp bạn truy cập từ xa. Phương pháp này hoạt động tốt nhất trong một mạng nội bộ.

1. Tải **Adafruit's Pi Finder** cho hệ điều hành tương ứng tại [Adafruit-Pi-Finder Latest Releases](https://github.com/adafruit/Adafruit-Pi-Finder/releases/tag/3.0.0).

2. Mở Raspberry **Pi Finder** và **Click Find My Pi!**:

![find_my_pi](assets/tutorial2/find_my_pi.png)

**Hình 5:** Giao diện của Pi Finder

**Pi Finder** - được sử dụng để phát hiện _địa chỉ IP của Raspberry Pi_ trong môi trường nhà.

3. Kết quả sẽ bao gồm **địa chỉ IP**, **Người Dùng SSH** và **Mật Khẩu SSH** của Raspberry Pi:

![pi_finder_found_my_pi](assets/tutorial2/pi_finder_found_my_pi.png)

**Hình 6:** Pi Finder Found My Pi

4. SSH vào Pi từ máy tính của bạn bằng cách nhấn nút **Terminal**.

![successful_ssh_to_rpi](assets/tutorial2/successful_ssh_to_rpi.png)

**Hình 7:** Giao diện dòng lệnh của Raspberry Pi

#### 1.4: Cài Đặt Phần Mềm Sense HAT Cho Raspberry Pi

Tải và cài đặt phần mềm Sense HAT bằng các lệnh sau:

```bash
sudo apt-get update
sudo apt-get install sense-hat
sudo pip3 install pillow
```

Bây giờ bạn đã có thư viện phần mềm Sense HAT, sẽ sử dụng nó trong bài hướng dẫn tiếp theo để lấy đọc các giá trị cảm biến.

#### 1.5: Cài Đặt MiNiFi Java Agent Cho Raspberry Pi

Trong phần này, bạn sẽ cài đặt Java 8 và JDK 1.8 trên Raspberry Pi vì chúng là yêu cầu để chạy MiNiFi.

Tải và cài đặt Java 8 và JDK1.8:

```bash
sudo apt-get update && sudo apt-get install oracle

-java8-jdk
```

> Lưu ý: Quá trình cài đặt sẽ mất khoảng 10 phút tùy thuộc vào tài nguyên của Hệ Điều Hành Raspbian.

1. Tải MiNiFi Java Agent từ [Apache nifi minifi Downloads](http://nifi.apache.org/minifi/download.html) ở phần **Releases -> MiNiFi (Java) -> Binaries**.

2. Nhấp vào **minifi-[latest-version]-bin.zip**, sau đó tải MiNiFi từ bất kỳ liên kết nào được cung cấp xuống máy tính của bạn.

![download_links_minifi](assets/tutorial2/download_links_minifi.jpg)

**Hình 8:** Tải MiNiFi

3. Sử dụng nút **Upload** của Pi Finder để chuyển ứng dụng MiNiFi lên Raspberry Pi. Chọn **minifi-[latest-version]-bin.zip** và nhấp **Open**.

![upload_minifi_to_rpi](assets/tutorial2/upload_minifi_to_rpi.jpg)

**Hình 9:** Tải MiNiFi lên Raspberry Pi

4. Sử dụng nút **Terminal** của Pi Finder để nhập Raspberry Pi và giải nén dự án MiNiFi.

```bash
unzip minifi-[latest-version]-bin.zip
```

Một MiNiFi Agent đã được cài đặt lên Raspberry Pi. Chúng ta sẽ giải thích thêm về MiNiFi Agent trong bài hướng dẫn tiếp theo.

#### 1.6: Tải MiNiFi Toolkit Lên Máy Tính Của Bạn

Trong phần này, bạn tải MiNiFi toolkit lên máy tính của mình vì nó cần để chuyển đổi luồng NiFi thành định dạng luồng MiNiFi. Trong bài hướng dẫn tiếp theo, bạn sẽ xây dựng luồng MiNiFi trong NiFi.

1. Tải MiNiFi Toolkit từ [Apache nifi minifi Downloads](http://nifi.apache.org/minifi/download.html) ở phần **Releases -> MiNiFi Toolkit Binaries -> [latest-version] - Compatible with MiNiFi Java [latest-version]**.

2. Nhấp vào **minifi-toolkit-[latest-version]-bin.zip** sau đó tải MiNiFi Toolkit từ bất kỳ liên kết nào được cung cấp xuống máy tính của bạn.

3. Di chuyển đến nơi đã tải MiNiFi và giải nén MiNiFi Toolkit bằng phần mềm giải nén yêu thích của bạn.

![decompress_minifi_toolkit](assets/tutorial2/decompress_minifi_toolkit.jpg)

**Hình 10:** Trích xuất MiNiFi Toolkit

Bây giờ MiNiFi Toolkit sẽ sẵn sàng sử dụng trong bài hướng dẫn tiếp theo.

#### 1.7: Hiệu Chỉnh Múi Giờ Của Raspberry Pi

Tại sao lại quan trọng để hiệu chỉnh múi giờ trên Raspberry Pi?

Để hệ thống của bạn có thời gian và ngày đúng, múi giờ cần được hiệu chỉnh. Một trong những lĩnh vực quan trọng của việc này là trong bài hướng dẫn tiếp theo khi bạn tạo kịch bản Python vẽ dấu thời gian cho mỗi đọc cảm biến.

SSH vào Raspberry Pi bằng nút **Terminal** của Pi Finder của Adafruit.

1. Gõ `sudo raspi-config`

- **raspi-config** được sử dụng để thay đổi cấu hình HĐH và sẽ được sử dụng để hiệu chỉnh ngày/thời gian hiện tại cho múi giờ của bạn

![raspi_config_menu](assets/tutorial2/raspi_config_menu.png)

**Hình 11:** Menu chính của raspi-config

2. Chọn **4. Internationalisation Options**. Nhấn "Enter" trên bàn phím.

3. Chọn **I2 Change Timezone**.

![change_timezone](assets/tutorial2/change_timezone.png)

**Hình 12:** Menu Tùy Chọn Quốc Tế

4. Chọn khu vực địa lý phù hợp của bạn.

- Ví dụ: US

![geographic_area](assets/tutorial2/geographic_area.png)

**Hình 13:** Các Mục Lựa Chọn Khu Vực Địa Lý

5. Chọn múi giờ phù hợp của bạn.

- Ví dụ: Thái Bình Dương

![time_zone](assets/tutorial2/time_zone.png)

**Hình 14:** Các Mục Lựa Chọn Múi Giờ

6. Bạn sẽ quay lại menu. Chọn **<Finish>**. Thời gian mới được hiệu chỉnh sẽ được hiển thị:

![output_calibrated_time](assets/tutorial2/output_calibrated_time.png)

**Hình 15:** Múi Giờ Đã Được Hiệu Chỉnh

### Bước 2: Cấu Hình Bridged Adapter Network cho VirtualBox

Trong bước này, bạn sẽ cấu hình Bridged Adapter Network cho máy ảo VirtualBox để cho phép máy ảo kết nối với cùng một mạng như Raspberry Pi.

1. Mở **Oracle VM VirtualBox** trên máy tính của bạn.

2. Chọn máy ảo HDF Sandbox trong danh sách máy ảo.

3. Nhấp chuột phải vào máy ảo và chọn **Settings**.

![virtualbox_settings](assets/tutorial2/virtualbox_settings.png)

**Hình 16:** Cài đặt máy ảo VirtualBox

4. Trong cửa sổ **Settings**, chọn **Network** từ thanh bên trái.

5. Chọn **Adapter 1**.

6. Trong mục **Attached to**, chọn **Bridged Adapter**.

![bridged_adapter_settings](assets/tutorial2/bridged_adapter_settings.png)

**Hình 17:** Cấu hình Bridged Adapter Network

7. Trong danh sách **Name**, chọn tên của thiết bị mạng của bạn.

> Lưu ý: Đảm bảo chọn thiết bị mạng có kết nối Internet.

8. Nhấn **OK** để đóng cửa sổ **Settings**.

### Bước 3: Ánh Xạ Bridged IP đến Hostname Mong Muốn trong File hosts

Để dễ quản lý và giao tiếp với các thiết bị trong mạng, bạn có thể ánh xạ địa chỉ IP Bridged của HDF Sandbox (máy ảo) đến một tên hostname mong muốn. Điều này sẽ giúp bạn gửi dữ liệu từ Raspberry Pi đến HDF Sandbox.

1. Mở **Notepad** hoặc **bất kỳ trình soạn thảo văn bản nào bạn muốn** trên máy tính của bạn với quyền quản trị.

2. Mở file **C:\Windows\System32\drivers\etc\hosts** (đối với Windows) hoặc **/etc/hosts** (đối với Linux/Mac).

3. Thêm dòng sau ở cuối file và lưu lại:

   ```
   [Bridged_IP_HDF_Sandbox] hdf-sandbox
   ```

   Thay thế `[Bridged_IP_HDF_Sandbox]` bằng địa chỉ IP bạn đã chọn cho Bridged Adapter Network của HDF Sandbox.

   ![hosts_file_example](assets/tutorial2/hosts_file_example.png)

   **Hình 18:** Một ví dụ về nội dung file hosts

### Bước 4: Xác Nhận Đã Đáp Ứng Các Tiền Điều Kiện

Để đảm bảo mọi thứ đều đã được thiết lập đúng, hãy xác nhận rằng:

1. Raspberry Pi có thể kết nối Internet.
2. Raspberry Pi có thể SSH vào HDF Sandbox qua địa chỉ hostname.

### Bước 5: Khởi Động HDF Sandbox và Cấu Hình NiFi Site-To-Site

1. Khởi động HDF Sandbox theo hướng dẫn đã được cung cấp.

2. Mở trình duyệt và truy cập **Ambari Dashboard** tại [http://hdf-sandbox:8080](http://hdf-sandbox:8080) (hoặc địa chỉ tương ứng của bạn).

3. Đăng nhập với tên người dùng là `admin` và mật khẩu là `admin`.

4. Trong trang **Ambari Dashboard**, chọn **NiFi** từ menu bên trái.

5. Trong tab **Summary**, nhấp vào liên kết **NiFi UI** để mở trang quản lý NiFi.

6. Trong giao diện NiFi, chọn **NiFi Flow Configuration** từ thanh điều hướng bên trái.

7. Thêm một Site-to-Site (S2S) hoặc một Output Port:

   - Nhấp chuột phải vào không gian làm việc và chọn **Configure**.
   - Chọn tab **Site-to-Site**.
   - Nhấp **+ (Add)** để thêm một Output Port mới.
   - Đặt **Port Name** và **Port Identifier** (ví dụ: `RaspberryPi_OutputPort`).

8. Lưu cấu hình và khởi động Output Port.

### Tóm Tắt

Trong bài hướng dẫn này, bạn đã thực hiện các bước sau:

- Thiết lập một Trạm Thời Tiết IoT với Raspberry Pi và Sense HAT.
- Cấu hình Bridged Adapter Network cho máy ảo VirtualBox.
- Ánh xạ địa chỉ IP Bridged đến một tên hostname mong muốn trong file hosts.
- Khởi động HDF Sandbox và cấu hình NiFi Site-To-Site.

Bài tiếp theo sẽ hướng dẫn bạn xây dựng luồng NiFi để nhận dữ liệu từ Raspberry Pi thông qua MiNiFi và thực hiện phân tích dữ liệu.
