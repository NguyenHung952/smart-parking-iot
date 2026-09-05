# 🚗 Smart Parking IoT System

> **Hệ thống bãi đỗ xe thông minh sử dụng STM32, giao tiếp RS232, nhận diện biển số ANPR và Web Dashboard.**

[![Platform](https://img.shields.io/badge/Platform-STM32-blue)](https://www.st.com/)
[![IDE](https://img.shields.io/badge/IDE-STM32CubeIDE-orange)](https://www.st.com/en/development-tools/stm32cubeide.html)
[![Python](https://img.shields.io/badge/Python-3.x-yellow)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-Computer%20Vision-green)](https://opencv.org/)
[![YOLO](https://img.shields.io/badge/YOLO-Object%20Detection-red)](https://github.com/ultralytics/ultralytics)
[![PaddleOCR](https://img.shields.io/badge/PaddleOCR-OCR-lightgrey)](https://github.com/PaddlePaddle/PaddleOCR)

---

## 📌 Giới thiệu

**Smart Parking IoT** là đồ án xây dựng một mô hình bãi đỗ xe thông minh với **4 vị trí đỗ xe**.

Hệ thống sử dụng **STM32 làm bộ điều khiển trung tâm**, thực hiện đọc trạng thái các vị trí đỗ, điều khiển hiển thị và quản lý trạng thái bãi xe.

Dữ liệu được truyền từ STM32 đến máy tính thông qua **RS232**, sau đó được xử lý bởi chương trình Python.

Hệ thống được phát triển theo hướng kết hợp:

```text
Embedded System
      +
Serial Communication
      +
Computer Vision / ANPR
      +
Database
      +
Web Dashboard
```

---

## ✨ Tính năng chính

### 🔧 Embedded System

* Quản lý **4 vị trí đỗ xe**
* STM32 làm bộ điều khiển trung tâm
* LCD 16x2 hiển thị trạng thái
* LED và Buzzer cảnh báo
* EEPROM 24C16 lưu trạng thái và bộ đếm
* Hỗ trợ các chế độ:

  * `AUTO`
  * `TEST`
  * `DEMO`
* Xử lý chống nhiễu tín hiệu với:

  * `RAW`
  * `STAB`

### 📡 Communication

Giao tiếp giữa STM32 và máy tính thông qua:

```text
RS232
115200 baud
8 data bits
No parity
1 stop bit
```

Ví dụ dữ liệu trạng thái:

```text
STATUS,S1=0,S2=0,S3=0,S4=0,FREE=4,OCC=0,IN=0,OUT=0,GATE=IDLE
```

Trong đó:

```text
S1 → S4     : trạng thái 4 vị trí
FREE        : số vị trí còn trống
OCC         : số vị trí đang sử dụng
IN          : số lượt xe vào
OUT         : số lượt xe ra
GATE        : trạng thái cổng
```

---

## 📷 ANPR – Automatic Number Plate Recognition

Hệ thống có module nhận diện biển số xe (**ANPR**).

Pipeline xử lý:

```text
Camera / Image
      ↓
YOLO Detection
      ↓
License Plate
      ↓
PaddleOCR
      ↓
Plate Normalization
      ↓
SQLite Database
```

Module ANPR được tổ chức riêng:

```text
Core/Python/ANPR/
│
├── camera.py
├── detector.py
├── ocr.py
├── plate_history.py
│
├── models/
│   └── best.pt
│
└── data/
    └── plate_history.db
```

Ví dụ biển số sau khi chuẩn hóa:

```text
12-B1-168-88
```

---

## 🗄️ Database

ANPR sử dụng **SQLite** để lưu lịch sử nhận diện biển số.

Thông tin lưu trữ bao gồm:

```text
ID
Biển số
Thời gian
Sự kiện
Vị trí
```

Ví dụ:

```text
ID=1
Plate=12-B1-168-88
Event=IN
Slot=SLOT 1
```

và:

```text
ID=2
Plate=12-B1-168-88
Event=OUT
Slot=SLOT 1
```

Database ANPR hiện được tách riêng để tránh ảnh hưởng đến phần Web Dashboard đang phát triển.

---

## 🌐 Web Dashboard

Web Dashboard được xây dựng để theo dõi trạng thái bãi đỗ từ máy tính.

Dự kiến hiển thị:

```text
┌─────────────────────────────────────┐
│          SMART PARKING              │
├─────────┬─────────┬─────────┬───────┤
│ SLOT 1  │ SLOT 2  │ SLOT 3  │ SLOT4 │
│  FREE   │  OCCUPY │  FREE   │ FREE  │
├─────────┴─────────┴─────────┴───────┤
│ Vehicles IN       : 10               │
│ Vehicles OUT      : 7                │
│ Available Slots   : 3                │
│ Occupied Slots    : 1                │
└─────────────────────────────────────┘
```

> **Trạng thái:** Web Dashboard đang được tiếp tục hoàn thiện và tích hợp với hệ thống ANPR.

---

## 🧩 Kiến trúc hệ thống

```text
                 ┌──────────────────┐
                 │  Parking Sensors │
                 │    / Buttons     │
                 └────────┬─────────┘
                          │
                          ▼
                ┌──────────────────┐
                │      STM32       │
                │ Main Controller  │
                └───────┬───┬──────┘
                        │   │
                ┌───────┘   └────────┐
                ▼                    ▼
          ┌──────────┐         ┌──────────┐
          │ LCD 16x2 │         │ EEPROM   │
          │ LED/Buzzer│        │ 24C16    │
          └──────────┘         └──────────┘
                │
                │ RS232
                ▼
        ┌──────────────────┐
        │  Python Backend  │
        └───────┬──────────┘
                │
        ┌───────┴───────────────┐
        ▼                       ▼
 ┌──────────────┐       ┌──────────────┐
 │     ANPR     │       │ Web Dashboard│
 │ YOLO + OCR   │       │              │
 └───────┬──────┘       └──────────────┘
         │
         ▼
 ┌──────────────┐
 │    SQLite    │
 │ Plate History│
 └──────────────┘
```

---

## 🛠️ Hardware

| Thành phần            | Chức năng                                 |
| --------------------- | ----------------------------------------- |
| **ARM KIT 1 / STM32** | Bộ điều khiển trung tâm                   |
| **LCD 16x2**          | Hiển thị trạng thái                       |
| **EEPROM 24C16**      | Lưu trạng thái và bộ đếm                  |
| **LED**               | Hiển thị / cảnh báo                       |
| **Buzzer**            | Cảnh báo                                  |
| **Button / Sensor**   | Mô phỏng hoặc phát hiện trạng thái vị trí |
| **USB-RS232**         | Giao tiếp STM32 ↔ PC                      |

---

## 💻 Software & Technologies

### Firmware

* STM32CubeIDE
* STM32CubeMX
* HAL Library
* UART
* I2C
* EEPROM
* State Machine

### Python

* Python 3.x
* PySerial
* OpenCV
* Ultralytics YOLO
* PaddleOCR
* SQLite

### Web

* HTML
* CSS
* JavaScript
* Python Backend

---

## 📂 Project Structure

```text
smart-parking-iot/
│
├── Core/
│   ├── Inc/
│   ├── Src/
│   │
│   └── Python/
│       │
│       └── ANPR/
│           ├── camera.py
│           ├── detector.py
│           ├── ocr.py
│           ├── plate_history.py
│           │
│           ├── models/
│           │   └── best.pt
│           │
│           └── data/
│               └── plate_history.db
│
├── Web/
│   ├── index.html
│   ├── style.css
│   └── app.js
│
├── Documentation/
│
├── .gitignore
└── README.md
```

---

## 🔄 System Workflow

### Xe vào

```text
Xe đến cổng
     ↓
Phát hiện xe
     ↓
Nhận diện biển số
     ↓
Kiểm tra vị trí trống
     ↓
Mở cổng
     ↓
Xe vào
     ↓
Cập nhật trạng thái STM32
     ↓
Lưu lịch sử
     ↓
Web Dashboard cập nhật
```

### Xe ra

```text
Xe đến cổng OUT
     ↓
Nhận diện biển số
     ↓
Tìm lịch sử xe
     ↓
Xác định vị trí
     ↓
Xe rời bãi
     ↓
Cập nhật STM32
     ↓
Giải phóng vị trí
     ↓
Lưu sự kiện OUT
     ↓
Web Dashboard cập nhật
```

---

## 🧪 Development Status

| Module                      |     Status    |
| --------------------------- | :-----------: |
| STM32 Firmware              |   ✅ Working   |
| 4 Parking Slots             |   ✅ Working   |
| LCD 16x2                    |   ✅ Working   |
| LED / Buzzer                |   ✅ Working   |
| EEPROM                      |   ✅ Working   |
| AUTO / TEST / DEMO          |   ✅ Working   |
| RAW / STAB                  |   ✅ Working   |
| RS232                       |    ✅ Tested   |
| Python Serial Communication |    ✅ Tested   |
| ANPR Database               |   ✅ Working   |
| YOLO Detection              | 🔄 Developing |
| PaddleOCR                   | 🔄 Developing |
| Web Dashboard               | 🔄 Developing |
| Full ANPR Integration       | 🔄 Developing |

---

## 🎯 Mục tiêu phát triển

Các bước tiếp theo của dự án:

* [ ] Hoàn thiện Web Dashboard
* [ ] Hoàn thiện tích hợp ANPR
* [ ] Đồng bộ dữ liệu ANPR với hệ thống parking
* [ ] Hoàn thiện luồng xe `IN / OUT`
* [ ] Hiển thị lịch sử xe trên Web Dashboard
* [ ] Hoàn thiện giao diện người dùng
* [ ] Kiểm thử toàn bộ hệ thống

---

## 📸 Demo

> Hình ảnh và video demo của hệ thống sẽ được bổ sung trong quá trình hoàn thiện dự án.

Có thể thêm:

```text
docs/
├── hardware.jpg
├── lcd-demo.jpg
├── serial-monitor.jpg
├── anpr-demo.jpg
└── dashboard.jpg
```

---

## 📚 Documentation

Tài liệu dự án có thể bao gồm:

* Sơ đồ khối hệ thống
* Sơ đồ nguyên lý
* Sơ đồ kết nối phần cứng
* Flowchart thuật toán
* Tài liệu giao tiếp RS232
* Tài liệu ANPR
* Database design
* Hướng dẫn cài đặt
* Hướng dẫn chạy chương trình
* Báo cáo đồ án

---

## 👨‍💻 Project

**Project:** Smart Parking IoT System
**Type:** Advanced Engineering Project
**Platform:** STM32 + Python + Web
**Parking Slots:** 4

---

## 📜 License

This project is developed for educational and academic purposes.
