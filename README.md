# Smart Parking IoT System

Hệ thống bãi đỗ xe thông minh sử dụng STM32 làm bộ điều khiển chính,
kết hợp cảm biến, LCD 16x2, EEPROM, giao tiếp RS232,
nhận diện biển số xe ANPR và Web Dashboard.

## Features

- Quản lý 4 vị trí đỗ xe
- STM32 làm bộ điều khiển trung tâm
- LCD 16x2 hiển thị trạng thái
- LED và Buzzer cảnh báo
- EEPROM 24C16 lưu trạng thái và bộ đếm
- Chế độ AUTO / TEST / DEMO
- Chống nhiễu tín hiệu RAW / STAB
- Giao tiếp RS232 với máy tính
- Python Backend
- Nhận diện biển số xe ANPR
- Lưu lịch sử biển số bằng SQLite
- Web Dashboard theo dõi trạng thái bãi xe

## Hardware

- ARM KIT 1 / STM32
- LCD 16x2
- EEPROM 24C16
- LED
- Buzzer
- Push Buttons / Sensors
- USB-RS232

## Software

- STM32CubeIDE
- STM32CubeMX
- Python
- OpenCV
- YOLO
- PaddleOCR
- SQLite
- Web Dashboard

## System Architecture

```text
Sensors / Buttons
       |
       v
     STM32
       |
  +----+----+
  |         |
 LCD     EEPROM
  |
 RS232
  |
  v
Python Backend
  |
  +----> ANPR
  |       |
  |    YOLO + OCR
  |       |
  |    SQLite
  |
  v
Web Dashboard
