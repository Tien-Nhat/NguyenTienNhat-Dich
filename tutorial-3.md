---
title: Thu thập Dữ liệu thời tiết từ Sense HAT thông qua CDA
---

# Thu thập Dữ liệu thời tiết từ Sense HAT thông qua CDA

## Giới thiệu

Bạn sẽ học cách tạo một chương trình Python để thu thập đọc cảm biến từ Raspberry Pi Sense HAT cho nhiệt độ, độ ẩm và áp suất khí quyển. Bạn cũng sẽ chạy MiNiFi trên Raspberry Pi để tiếp nhận các đọc cảm biến thời tiết và chuyển tiếp chúng đến vị trí của NiFi trên HDF sandbox thông qua giao thức Site-to-Site. Cuối cùng, bạn sẽ xác minh rằng NiFi có thể liên lạc với HDP bằng cách lưu trữ dữ liệu vào Hệ thống Tệp phân tán Hadoop (HDFS).

## Tiền điều kiện

- Triển khai Trạm thời tiết IoT và Kiến trúc Dữ liệu Kết nối

## Kế hoạch

- [Bước 1: Tạo Kịch bản Python để Ghi Dữ liệu Thời tiết Sense HAT](#bước-1-tạo-kịch-bản-python-để-ghi-dữ-liệu-thời-tiết-sense-hat)
- [Bước 2: Xây dựng Luồng NiFi để Lưu trữ Dữ liệu MiNiFi vào HDFS](#bước-2-xây-dựng-luồng-nifi-để-lưu-trữ-dữ-liệu-minifi-vào-hdfs)
- [Bước 3: Xây dựng Luồng MiNiFi để Đẩy Dữ liệu đến NiFi](#bước-3-xây-dựng-luồng-minifi-để-đẩy-dữ-liệu-đến-nifi)
- [Tóm tắt](#tóm-tắt)
- [Đọc thêm](#đọc-thêm)
- [Phụ lục A: Khắc phục sự cố MiNiFi đến NiFi Site-to-Site](#phụ-lục-a-khắc-phục-sự-cố-minifi-đến-nifi-site-to-site)

### Bước 1: Tạo Kịch bản Python để Ghi Dữ liệu Thời tiết Sense HAT

Bạn sẽ học cách tạo một kịch bản Python trên Raspberry Pi để thu thập dữ liệu thời tiết từ Sense HAT. Có hai cách tiếp cận: **Cách tiếp cận 1** bạn sẽ triển khai kịch bản Python từng bước một trong khi **Cách tiếp cận 2** bạn sẽ tải kịch bản Python lên Raspberry Pi.

#### Cách tiếp cận 1: Triển khai Kịch bản Python lên Raspberry Pi

Chúng tôi sẽ giải thích từng phần của mã và ý nghĩa của chúng trong dự án từ 1.1 - 1.6. Ở 1.7, mã đầy đủ cho WeatherStation được cung cấp.

Mở HDF Sandbox Web Shell:

http://sandbox-hdf.hortonworks.com:4200

Tạo một tệp mới "WeatherStation.py"

```bash
touch WeatherStation.py
vi WeatherStation.py
```

Bây giờ bạn đã sẵn sàng bắt đầu thêm mã vào tệp văn bản của mình.

#### 1.1: Thu thập Số sê-ri của Raspberry Pi

Số sê-ri sẽ được sử dụng để phân biệt mỗi Trạm thời tiết Raspberry Pi.

```python
# Cố gắng lấy Số sê-ri của Raspberry Pi
serial = get_serial()
```

Mã gọi hàm **get_serial()** và lưu số sê-ri của Raspberry Pi vào biến **serial**.

```python
# Lấy Số sê-ri của Raspberry Pi
def get_serial():
  # Trích xuất số sê-ri từ tệp cpuinfo
  cpuserial = "0000000000000000"
  try:
    f = open('/proc/cpuinfo','r')
    for line in f:
      # Kiểm tra xem ký tự từ 0 đến 6 có khớp với Serial không
      if line[0:6]=='Serial':
        # sau đó gán cpuserial với các ký tự từ 10 đến 26 của dòng
        cpuserial = line[10:26]
      # lặp đến dòng tiếp theo trong đối tượng tệp cpuinfo f
    f.close()
  except:
    cpuserial = "ERROR000000000"
  return cpuserial
```

Hàm **get_serial()** sau đó tìm kiếm tệp `/proc/cpuinfo` để tìm từ khóa **Serial**. Khi từ khóa này được tìm thấy, số sê-ri của CPU được lưu vào biến **cpuserial**.

#### 1.2: Thu thập Dấu thời gian cho Đọc cảm biến

Dấu thời gian cho biết thời gian khi đọc cảm biến được thực hiện.

```python
# Lấy Thời gian hiện tại được ưa chuộng bởi OS
timestamp = get_time()
```

Mã gọi hàm **get_time()** và lưu thời điểm hiện tại vào biến **timestamp**.

```python
# Lấy Thời gian hiện tại được ưa chuộng bởi OS
def get_time():
  current_time = datetime.datetime.now().strftime("%Y-%m-%dT%H:%M:%SZ")
  return current_time
```

Bên trong hàm **get_time()**, **datetime.datetime.now()** truy xuất ngày và giờ hiện tại được ưa chuộng bởi hệ điều hành, sau đ

ó **strftime("%Y-%m-%dT%H:%M:%SZ")** định dạng ngày giờ theo năm, tháng, ngày, được phân tách bởi ký tự 'T' để chỉ thời gian của ngày trong giờ, phút và giây. Giá trị ngày giờ đã được định dạng này được trả về cho **current_time** dưới dạng chuỗi.

#### 1.3: Thu thập Đọc cảm biến thời tiết từ Sense HAT

Nhiệt độ, độ ẩm và áp suất khí quyển được trích xuất từ Sense HAT. **sense.get_temperature** lấy giá trị nhiệt độ và lưu vào biến **temp_c**.

```python
# Cố gắng lấy đọc cảm biến.
temp_c = sense.get_temperature()
humidity_prh = sense.get_humidity()
humidity_prh = round(humidity_prh, 2)
pressure_mb = sense.get_pressure()
pressure_mb = round(pressure_mb, 2)
```

#### 1.4: Thử hiệu chỉnh Đọc cảm biến nhiệt độ Sense HAT

Các giá trị Đọc cảm biến nhiệt độ Sense HAT bị chệch so với nhiệt độ thực tế do CPU của Raspberry Pi phát ra nhiệt ở xung quanh Sense HAT. Nhiệt độ nhân CPU của Raspberry Pi phát ra 55.8 Celsius (132.44 Fahrenheit). Do đó, để có thể thu thập dữ liệu hữu ích từ cảm biến nhiệt độ, chúng ta phải thử hiệu chỉnh cảm biến.

```python
# Lấy Nhiệt độ Lõi CPU của Raspberry Pi
cpu_temp_c = get_cpu_temp_c()

# Hiệu chỉnh Đọc cảm biến Nhiệt độ Sense HAT
temp_c = calibrate_temp_c(cpu_temp_c, temp_c)
temp_c = round(temp_c, 2)
```

Trong mã trên, hàm **get_cpu_temp_c()** được gọi để có được nhiệt độ CPU, sau đó kết quả đó được lưu vào biến **cpu_temp_c**.

Biến **temp_c** được ghi đè bằng giá trị nhiệt độ Sense HAT đã được hiệu chỉnh bằng cách gọi hàm **calibrate_temp_c(cpu_temp_c, temp_c)**.

```python
# Lấy Nhiệt độ Lõi CPU của Raspberry Pi qua lệnh shell "vcgencmd"
def get_cpu_temp_c():
  cpu_temp = subprocess.check_output("vcgencmd measure_temp", shell=True)
  # Phân tách một Chuỗi và thêm dữ liệu vào mảng chuỗi bằng bộ phân cách "="
  array = cpu_temp.split("=")
  array2 = array[1].split("'")
  # Lấy giá trị nhiệt độ từ phần tử 0 của mảng2
  cpu_tempc = float(array2[0])
  cpu_tempc = float("{0:.2f}".format(cpu_tempc))
  return cpu_tempc

# Đọc cảm biến Nhiệt độ Sense HAT bị chệch do nhiệt độ của CPU
# Hiệu chỉnh đọc cảm biến nhiệt độ bằng cách sử dụng hệ số tỷ lệ: 5.466
# Hệ số tỷ lệ là số độ mà Sense HAT bị chệch so với nhiệt độ thực tế
def calibrate_temp_c(cpu_tempc, temp_c):
  temp_c - ((cpu_tempc - temp_c)/5.466)
  return temp_c
```

Hàm **get_cpu_temp_c()** lưu đầu ra nhiệt độ CPU của Raspberry Pi từ lệnh shell "vcgencmd measure_temp" vào biến **cpu_temp**. Vì lệnh shell này xuất nhiệt độ CPU dưới dạng "temp=50.5'C", hai biến mảng (**array**, **array2**) được sử dụng để phân tách chuỗi và lấy giá trị nhiệt độ CPU, chẳng hạn như "50.5". Giá trị được trả về mỗi khi người dùng gọi hàm này.

**calibrate_temp_c(cpu_temp_c, temp_c)** nhận làm tham số: cpu_temp_c và temp_c, sau đó tính toán giá trị nhiệt độ Sense HAT đã hiệu chỉnh hơn trong khi tính đến hệ số tỷ lệ: 5.466. Hệ số tỷ lệ là số độ Sense HAT bị chệch so với nhiệt độ thực tế ở một vị trí cụ thể. Hãy nhớ rằng Nhiệt độ Sense HAT có thể vẫn chưa đúng so với nhiệt độ thực tế vì Sense HAT vẫn ở gần nhiệt độ CPU.

Nếu bạn quan tâm đến việc biết thêm về cách tính hệ số tỷ lệ, hãy đọc đoạn văn sau, nếu không, chuyển đến 1.5.

Giá trị hệ số tỷ lệ, chẳng hạn như 5.466, có thể được lấy bằng cách ghi lại đọc nhiệt độ thực tế (ví dụ: cảm biến DHT22) nhiều lần trong khoảng thời gian 24 giờ. Đồng thời, bạn cũng sẽ ghi lại đọc nhiệt độ từ Sense HAT. Sau đó, bạn sẽ lấy tất cả các giới thiệu về nhiệt độ thực tế, tính giá trị trung bình và trừ nó đi từ giới thiệu trung bình của Sense HAT. Kết quả của hệ số tỷ lệ sẽ hiển thị số độ Sense HAT chệch so với nhiệt độ thực tế.

#### 1.5: Thu thập Địa chỉ IP Công cộng của Raspberry Pi

Địa chỉ IP Công cộng của Raspberry Pi có thể được sử dụng để xác định thông tin địa lý, chẳng hạn như thành phố và tiểu bang nơi nút đó ghi lại dữ liệu thời tiết.

Mã trích xuất Địa chỉ IP Công cộng thông qua cuộc gọi REST đến IPIFY và sau đó phân tích cú pháp JSON để lấy giá trị **ip**.

```python
# Cố gắng lấy IP Công cộng
public_ip = get_public_ip()
```

Mã trên gọi hàm **get_public_ip()** và lưu Địa chỉ IP Công cộng của Raspberry Pi vào biến **ip**.

```python
# Lấy Địa chỉ IP Công cộng của Raspberry Pi qua Cuộc gọi REST IPIFY
def get_public_ip():
  ip = json.load(urllib2.urlopen('https://api.ipify.org/?format=json'))['ip']
  return ip
```

Bên trong hàm **get_public_ip()**, **urllib2.urlopen('https://api.ipify.org/?format=json')** mở URL của yêu cầu HTTP IPIFY, trả về Địa chỉ IP Công cộng của Raspberry Pi dưới dạng một đối tượng giống như tệp. **json.load()** đọc dữ liệu từ đối tượng giống như tệp, sau đó **['ip']** được đính kèm vào cuối **json.load()** là một Biểu thức JSONPath được sử dụng để trích xuất giá trị địa chỉ IP và lưu vào biến **ip**. Mỗi khi người dùng gọi **get_public_ip()**, họ sẽ nhận được Địa chỉ IP Công cộng của Raspberry Pi.

#### 1.6: In các giá trị Thuộc tính Thời tiết ra màn hình

Với các câu lệnh in, giá trị biến có thể được đầu ra ra đầu ra tiêu chuẩn và hiển thị trên màn hình:

```python
print "Serial = " + str(serial)
print "Time = \"" + str(timestamp) + "\""
print "Temperature_F = " + str(temp_f)
print "Humidity_PRH = " + str(humidity_prh)
print "Pressure_In = " + str(pressure_in)
print "Public_IP = " + str(public_ip)
```
