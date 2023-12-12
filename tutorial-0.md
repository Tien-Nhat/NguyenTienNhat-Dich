# Phân tích Dữ liệu Tại Chỗ từ Trạm Thời Tiết IoT thông qua Kiến Trúc Dữ Liệu Kết Nối

## Giới thiệu

Trong hai năm gần đây, San Jose đã trải qua một sự thay đổi trong điều kiện thời tiết từ việc có nhiệt độ cao nhất vào năm 2016 đến việc xuất hiện nhiều lũ lụt chỉ trong năm 2017. Bạn đã được thuê bởi Thành phố San Jose làm Data Scientist để xây dựng dự án Internet of Things (IoT) và Big Data, liên quan đến việc phân tích dữ liệu từ nhiều trạm thời tiết bằng cách sử dụng một khung dữ liệu di động và một khung dữ liệu ổn định để cải thiện việc theo dõi thời tiết. Bạn sẽ sử dụng Connected Data Architecture của Hortonworks (Hortonworks Data Flow (HDF), Hortonworks Data Platform (HDP)) và dự án con MiNiFi của Apache NiFi để xây dựng sản phẩm này.

Với tư cách là Data Scientist, bạn sẽ tạo một bằng chứng thực nghiệm trong đó sử dụng Raspberry Pi và Sense HAT để sao chép dữ liệu từ trạm thời tiết, HDF Sandbox và HDP Sandbox trên Docker để phân tích dữ liệu thời tiết. Cuối cùng, bạn sẽ có khả năng hiển thị những hiểu biết ý nghĩa về nhiệt độ, độ ẩm và áp suất.

Trong loạt bài hướng dẫn này, bạn sẽ xây dựng một Trạm Thời Tiết Internet of Things (IoT) bằng cách sử dụng Kiến Trúc Dữ Liệu Kết Nối của Hortonworks, kết hợp các khung dữ liệu nguồn mở như MiNiFi, Hortonworks DataFlow (HDF) và Hortonworks Data Platform (HDP). Ngoài ra, bạn sẽ làm việc với Raspberry Pi và Sense HAT. Bạn sẽ sử dụng một MiNiFi agent để định tuyến dữ liệu thời tiết từ Raspberry Pi đến HDF Docker Sandbox thông qua giao thức Site-to-Site, sau đó bạn sẽ kết nối dịch vụ NiFi đang chạy trên HDF Docker Sandbox với HBase đang chạy trên HDP Docker Sandbox. Từ bên trong HDP, bạn sẽ học cách theo dõi dữ liệu thời tiết một cách trực quan trong HBase bằng cách sử dụng Zeppelin's Phoenix Interpreter.

![cda-minifi-hdf-hdp-architecture](assets/tutorial1/cda-minifi-hdf-hdp-architecture.png)

**Hình 1:** Trạm thời tiết IoT và tích hợp Kiến trúc Dữ liệu Kết nối

### Các Công Nghệ Big Data được Sử Dụng để Phát Triển Ứng Dụng:

- [Raspberry Pi Sense Hat](https://projects.raspberrypi.org/en/projects/getting-started-with-the-sense-hat)
- [HDF Sandbox](https://hortonworks.com/products/data-platforms/hdf/)
    - [Apache Ambari](https://ambari.apache.org/)
    - [Apache NiFi](https://nifi.apache.org/)
    - [Apache Kafka](http://kafka.apache.org/)
- [HDP Sandbox](https://hortonworks.com/products/data-platforms/hdp/)
    - [Apache Ambari](https://ambari.apache.org/)
    - [Apache Hadoop - HDFS](https://hadoop.apache.org/docs/r3.1.1/)
    - [Apache HBase](https://hbase.apache.org/)
    - [Apache Spark](https://spark.apache.org/)
    - [Apache Kafka](http://kafka.apache.org/)
    - [Apache Zeppelin](https://zeppelin.apache.org/)

## Mục Tiêu và Đối Tượng

Cuối cùng của loạt bài hướng dẫn này, bạn sẽ đạt được kiến thức cơ bản để xây dựng các ứng dụng liên quan đến IoT của riêng bạn. Bạn sẽ có khả năng kết nối MiNiFi, HDF Sandbox và HDP Sandbox. Bạn sẽ học cách vận chuyển dữ liệu qua các hệ thống từ xa và hiển thị dữ liệu để mang lại hiểu biết ý nghĩa cho khách hàng của bạn. Bạn sẽ cần có một nền tảng vững về các khái niệm cơ bản của lập trình (bất kỳ ngôn ngữ nào là đủ) để làm giàu kinh nghiệm của mình trong bài hướng dẫn này.

**Các mục tiêu học tập của loạt bài hướng dẫn này bao gồm:**

- Triển khai Trạm Thời Tiết IoT và Kiến Trúc Dữ Liệu Kết Nối
- Hiểu về Dự án IoT Raspberry Pi
- Hiểu chức năng của Cảm Biến Áp Suất/ Nhiệt Độ/ Độ Cao của Sense HAT
- Thực hiện một Đoạn mã Python để Kiểm soát Sense HAT để Tạo Dữ liệu Thời Tiết
- Tạo Bảng HBase để Lưu trữ Các Đọc Cảm Biến
- Xây dựng một luồng MiNiFi để Vận chuyển Dữ liệu Cảm Biến từ Raspberry Pi đến NiFi từ xa đang chạy trên HDF trên máy tính của bạn
- Xây dựng một luồng NiFi trên HDF Sandbox để tiền xử lý dữ liệu và làm phong phú dữ liệu cảm biến với thông tin vị tr

í địa lý, và lưu trữ dữ liệu vào HBase trên HDP Sandbox
- Hiển thị Dữ liệu Cảm Biến với Trình thông dịch Phoenix của Apache Zeppelin

## Danh Sách Vật Liệu:

- Raspberry Pi 3 Essentials Kit - Kết nối WiFi và Bluetooth tích hợp
- Raspberry Pi Sense Hat

## Yêu cầu Tiên Quyết

- Đã tải và triển khai [Hortonworks Data Platform (HDP)](https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html?utm_source=mktg-tutorial) Sandbox
- Đọc qua [Learning the Ropes of the HDP Sandbox](https://hortonworks.com/tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) để thiết lập ánh xạ tên máy chủ thành địa chỉ IP
- Nếu bạn không có 32GB RAM dành riêng cho HDP Sandbox, hãy tham khảo [Deploying Hortonworks Sandbox on Microsoft Azure](https://hortonworks.com/tutorial/sandbox-deployment-and-install-guide/section/4/)
- Kích hoạt Kiến Trúc Dữ Liệu Kết Nối:
  - [Bật CDA cho VirtualBox](https://hortonworks.com/tutorial/sandbox-deployment-and-install-guide/section/1/#enable-connected-data-architecture-cda---advanced-topic)
  - [Bật CDA cho VMware](https://hortonworks.com/tutorial/sandbox-deployment-and-install-guide/section/2/#enable-connected-data-architecture-cda---advanced-topic)
  - [Bật CDA cho Docker](https://hortonworks.com/tutorial/sandbox-deployment-and-install-guide/section/3/#enable-connected-data-architecture-cda---advanced-topic)

## Tổng Quan Về Loạt Bài Hướng Dẫn

Trong bài hướng dẫn này, chúng ta sẽ làm việc với dữ liệu cảm biến áp suất không khí, nhiệt độ và độ ẩm được thu thập từ Raspberry Pi bằng cách sử dụng Apache MiNiFi. Chúng ta sẽ vận chuyển dữ liệu MiNiFi đến NiFi bằng cách sử dụng Site-To-Site, sau đó tải dữ liệu với NiFi vào HBase để thực hiện phân tích dữ liệu.

1. **[Khái Niệm về IoT và Kiến Trúc Dữ Liệu Kết Nối](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/1/)** - Làm quen với Raspberry Pi, Chức năng Cảm biến Sense HAT, Giao tiếp Bộ lọc HDF và HDP Docker Sandbox, NiFi, MiNiFi, Zookeeper, HBase, Phoenix và Zeppelin.

2. **[Triển Khai Trạm Thời Tiết IoT và Kiến Trúc Dữ Liệu Kết Nối](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/2/)** - Cài đặt Trạm Thời Tiết IoT để xử lý dữ liệu cảm biến. Bạn sẽ cài đặt hệ điều hành Raspbian và MiNiFi trên Raspberry Pi, HDF Sandbox và HDP Sandbox trên máy tính cục bộ của bạn.

3. **[Thu Thập Dữ Liệu Thời Tiết Sense HAT trên CDA](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/3/)** - Lập trình Raspberry Pi để truy xuất dữ liệu cảm biến từ Cảm biến Sense HAT. Ghép một MiNiFi Agent vào Raspberry Pi để thu thập dữ liệu cảm biến và vận chuyển nó đến NiFi trên HDF qua Site-to-Site. Lưu trữ các đọc cảm biến thô vào HDFS trên HDP bằng cách sử dụng NiFi.

4. **[Đổ Dữ Liệu HDP HBase bằng Luồng HDF NiFi](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/4/)** - Tăng cường luồng NiFi bằng cách thêm vào các thuộc tính vị trí địa lý cho dữ liệu cảm biến và chuyển đổi nó thành định dạng JSON để lưu trữ dễ dàng vào HBase.

5. **[Hiển Thị Dữ Liệu Thời Tiết với Trình thông dịch Phoenix của Zeppelin](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/5/)** - Theo dõi dữ liệu thời tiết với Phoenix và tạo các hình ảnh minh họa cho những đọc cảm biến đó bằng cách sử dụng Trình thông dịch Phoenix của Zeppelin.

<!--

### Track 2: Hướng dẫn sử dụng Dữ liệu Mô phỏng

**[Hiển Thị Dữ Liệu Thời Tiết từ Nhiều Trạm]**(https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture/section/1/) - Triển khai nhiều container MiNiFi Docker trong Mạng Docker của VM khách của bạn, chúng kéo các dữ liệu mô phỏng của riêng mình, mô phỏng dữ liệu cảm biến mà sẽ được kéo từ Sense HAT và định tuyến dữ liệu từ các container nút biên đến container HDF từ xa nơi NiFi đang chạy. Bạn sẽ nhập một luồng NiFi, mẫu này có nhiều cổng vào nó lắng nghe dữ liệu đến từ các đại lý Trạm Thời Tiết MiNiFi và sau đó tiền xử lý dữ liệu, thêm thông tin vị trí địa lý, chuyển đổi dữ liệu thành định dạng JSON và lưu trữ nó vào HBase. Bạn sẽ tạo một bảng Phoenix trong Zeppelin và hiển thị dữ liệu.

-->

Loạt bài hướng dẫn được chia thành nhiều bài hướng dẫn cung cấp hướng dẫn từng bước, để bạn có thể hoàn thành các mục tiêu học tập và các nhiệm vụ liên quan đến nó. Bạn cũng được cung cấp một mẫu luồng dữ liệu cho mỗi bài hướng dẫn mà bạn có thể sử dụng để xác minh. Mỗi bài hướng dẫn xây dựng trên bài hướng dẫn trước đó.
