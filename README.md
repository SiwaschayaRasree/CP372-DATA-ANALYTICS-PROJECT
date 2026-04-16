# CP372-DATA-ANALYTICS-PROJECT


prompt :
Act as a Data Engineer and Revenue Management Expert.
Generate a synthetic hotel dataset in .xlsx format for the year 2025 (Jan 1 – Dec 31). The dataset must contain 5 relational tables, each placed in a separate worksheet in the Excel file. The dataset must simulate a Revenue Stagnation problem where the hotel shows high occupancy but low RevPAR compared to competitors.
1. Hotel Configuration Total Hotel Capacity: 90 rooms. Room types exist for pricing and booking purposes. You must use the room_type_id format:

SLG (Single): 50 rooms (Base Rate: $80)
DLX (Deluxe): 30 rooms (Base Rate: $150)
STE (Suite): 10 rooms (Base Rate: $350)
Important rule: While bookings in fact_bookings must specify the room_type_id, the daily room inventory and availability limits in dim_room_inventory must be combined and calculated at the total hotel level (90 rooms).
2. Seasons (Tropical / City Destination)

High Season: January – March, November – December
Shoulder Season: April – May, September – October
Low Season: June – August
3. Table Requirements The Excel file must contain the following worksheets.
Table 1: fact_bookings

Rows: 5,000 – 7,000 rows
Each row represents one reservation transaction.
Columns:
booking_id (Primary Key)
guest_id (Must follow the pattern g-xxxx, e.g., g-0001, g-0125)
booking_date
check_in_date
check_out_date
room_type_id (Must use the ID pattern: rt-01, rt-02, rt-03)
rate_code_id
channel_id
segment_id
status (Confirmed, Cancelled, No-Show)
total_room_revenue
number_of_rooms
adults_count
children_count
Important constraints:
Maximum Length of Stay: The length of stay (the difference between check_in_date and check_out_date) must be between 1 and 3 days maximum.
number_of_rooms must represent how many rooms were booked in that reservation.
The room nights between check_in_date and check_out_date must consume total room inventory.
Check-out date must always be after check-in date.
Table 2: dim_room_inventory

Rows: 365 rows (one per date)
Columns:
date
total_capacity (fixed = 90)
rooms_out_of_order
rooms_available_for_sale
Important rules:
Inventory is pooled at the total hotel level. Do not separate by room type.
Calculation rule: rooms_available_for_sale = total_capacity − rooms_out_of_order
Logical Consistency: The rooms_out_of_order (maintenance rooms) must logically align with the sum of number_of_rooms booked per date in fact_bookings. Specifically, the equation rooms_out_of_order + total daily occupied rooms MUST NEVER exceed total_capacity (90).
Table 3: dim_rate_codes Include the following rows:

RACK - Rack Rate
AAA - AAA Discount
CORP - Corporate
NRF - Non-Refundable
PROMO - Seasonal Promo
Table 4: dim_channels Include the following:

DIRECT - Direct Website (0% commission)
OTA_EXP - Expedia (18% commission)
OTA_BKG - Booking.com (18% commission)
WALKIN - Walk-in (0% commission)
CORPC - Corporate (10% commission)
Table 5: dim_calendar

Rows: 365
Columns:
date
day_name
is_weekend
is_holiday
season
4. Hidden Revenue Management Patterns The dataset must intentionally simulate Revenue Management inefficiencies.

4.1 The Volume Trap: Overall hotel occupancy should appear high (75–85%). However, bookings must be concentrated in rt-01 rooms and OTA channels (Expedia / Booking.com). Result: High occupancy, Low ADR, High commission costs.
4.2 Pricing Inefficiency: Weekend pricing must be poorly optimized. Rule: Weekend revenue should be only 5–10% higher than weekdays despite higher demand, simulating lost revenue opportunity.
4.3 The Suite Gap: rt-03 rooms must show very low occupancy (<20%). Reasons embedded in data: rt-03 rarely receives discounts, very few PROMO rate codes are applied to rt-03, and most demand flows into rt-01.
4.4 Lead Time Problem: Booking behavior should reflect inventory blocking by low-value demand. High-value demand: Direct bookings, short lead time (0–14 days before check-in). Low-value demand: OTA bookings, long lead time (30–120 days).
5. Returning Guest Pattern (Customer Loyalty Signal) The dataset must simulate a small proportion of returning guests.

Rule: Exactly 2% of guests must be returning guests.
Implementation:
Returning guests are identified strictly by exactly duplicate guest_id values (e.g., g-0125 appearing multiple times) in fact_bookings.
A returning guest must have at least 2 separate reservations.
The second booking's check_in_date must occur after the first stay's check_out_date.
Guest distribution: ~98% one-time guests, ~2% repeat guests.
Returning Guest Behavior Patterns:
Channel Preference Shift: Returning guests are more likely to book via DIRECT or CORPC (At least 50–60% of returning guest bookings must use these channels).
Room Type Upgrade Tendency: Returning guests have a higher probability of booking rt-02 rooms compared to first-time guests.
Shorter Lead Time: Returning guests tend to book closer to arrival (7–21 days before check-in).
Higher Booking Value: Returning guests are less price-sensitive. They use RACK or CORP rate codes more often and use PROMO rate codes less frequently.
6. Output Format Generate one Excel file (.xlsx) with five worksheets:

fact_bookings
dim_room_inventory
dim_rate_codes
dim_channels
dim_calendar
All tables must be logically consistent, especially overall room capacity constraints, booking check-in/check-out dates (max 3 days stay), total inventory availability matching rooms_out_of_order constraint, occupancy calculations, and returning guest behavior patterns utilizing the g-xxxx guest ID and rt-xx room type formats.

## 1. บทนำและความเป็นมา (Introduction & Background)

## 2. วัตถุประสงค์ของโครงการ (Research Objectives)

## 3. คำถามการวิจัยและสมมติฐาน (Research Questions & Hypothesis)
*research question*

## 4. ชุดข้อมูลและตัวแปรที่ใช้ (Dataset & Features)
* จำนวนแถวข้อมูล: 5,409 แถว
* จำนวนตัวแปรทั้งหมด: 14 ตัวแปร

### Data Dictionary
1. Sheet: fact_bookings
| Attribute          | คำอธิบาย                  | Data Type          | ช่วงค่าที่ถูกต้อง / ตัวอย่าง            |
| ------------------ | ------------------------- | ------------------ | --------------------------------------- |
| booking_id         | หมายเลขการจอง             | Nominal (Text)     | b-002266, b-003216                      |
| guest_id           | หมายเลขผู้เข้าพัก         | Nominal (Text)     | g-2157, g-3107                          |
| booking_date       | วันที่ทำการจอง            | Interval (Date)    | 26/09/2024                              |
| check_in_date      | วันที่เช็คอิน             | Interval (Date)    | 01/01/2025                              |
| check_out_date     | วันที่เช็คเอาท์           | Interval (Date)    | 03/01/2025                              |
| room_type_id       | ประเภทห้องพัก             | Nominal            | rt-01, rt-02, rt-03                     |
| rate_code_id       | รหัสอัตราค่าห้อง          | Nominal            | NRF, PROMO, AAA, RACK, CORP             |
| channel_id         | ช่องทางการจอง             | Nominal            | OTA_BKG, OTA_EXP, DIRECT, WALKIN, CORPC |
| segment_id         | กลุ่มลูกค้า               | Nominal            | LEISURE, BUSINESS                       |
| status             | สถานะการจอง               | Nominal            | Confirmed, Cancelled, No-Show           |
| total_room_revenue | รายได้รวมจากห้องพัก (USD) | Ratio (Continuous) | 0 – 4,200                               |
| number_of_rooms    | จำนวนห้องที่จอง           | Ratio (Discrete)   | 1 – 4                                   |
| adults_count       | จำนวนผู้ใหญ่              | Ratio (Discrete)   | 1 – 8                                   |
| children_count     | จำนวนเด็ก                 | Ratio (Discrete)   | 0 – 4                                   |


2. Sheet: dim_room_inventory
| Attribute                  | คำอธิบาย                                  | Data Type        | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
| -------------------------- | ----------------------------------------- | ---------------- | ---------------------------- |
| `date`                     | วันที่                                    | Interval (Date)  | 01/01/2025                   |
| `total_capacity`           | จำนวนห้องทั้งหมดของโรงแรม                 | Ratio (Discrete) | 100 – 500                    |
| `rooms_out_of_order`       | ห้องที่ไม่สามารถขายได้ (ซ่อม/ปิดปรับปรุง) | Ratio (Discrete) | 0 – 50                       |
| `rooms_available_for_sale` | ห้องที่พร้อมขาย                           | Ratio (Discrete) | 50 – 500                     |

3. Sheet: dim_rate_codes
| Attribute      | คำอธิบาย         | Data Type      | ช่วงค่าที่ถูกต้อง / ตัวอย่าง              |
| -------------- | ---------------- | -------------- | ----------------------------------------- |
| `rate_code_id` | รหัสอัตราค่าห้อง | Nominal        | NRF, PROMO, AAA, RACK, CORP               |
| `description`  | คำอธิบายเรท      | Nominal (Text) | Non-refundable, Promotion, Corporate Rate |

4. Sheet: dim_channels
| Attribute        | คำอธิบาย                 | Data Type          | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
| ---------------- | ------------------------ | ------------------ | ---------------------------- |
| `channel_id`     | รหัสช่องทางการจอง        | Nominal            | OTA_BKG, OTA_EXP, DIRECT     |
| `channel_name`   | ชื่อช่องทาง              | Nominal (Text)     | Booking.com, Expedia, Direct |
| `commission_pct` | เปอร์เซ็นต์ค่าคอมมิชชั่น | Ratio (Continuous) | 0 – 0.30 (เช่น 0.15 = 15%)   |

5. Sheet: dim_calendar
| Attribute    | คำอธิบาย                     | Data Type        | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
| ------------ | ---------------------------- | ---------------- | ---------------------------- |
| `date`       | วันที่                       | Interval (Date)  | 01/01/2025                   |
| `day_name`   | ชื่อวันในสัปดาห์             | Nominal          | Monday, Tuesday              |
| `is_weekend` | เป็นวันหยุดสุดสัปดาห์หรือไม่ | Boolean (Encoded as Integer) | 0, 1              |
| `is_holiday` | เป็นวันหยุดนักขัตฤกษ์หรือไม่ | Boolean (Encoded as Integer) | 0, 1             |
| `season`     | ฤดูกาล                       | Nominal          | High, Low, Shoulder          |


ตัวแปรเป้าหมาย (Target Variable)
ตัวแปรสำคัญที่ใช้วิเคราะห์ (Key Features)

## 5. ระเบียบวิธีวิจัย (Methodology)
5.1 Data Cleaning
<img width="1920" height="1020" alt="Screenshot 2026-04-16 205034" src="https://github.com/user-attachments/assets/1bea2f51-b62b-4fb1-912d-755e4837e5f1" />
<img width="1920" height="1020" alt="Screenshot 2026-04-16 210316" src="https://github.com/user-attachments/assets/8d11ac6e-7d59-4011-9092-1e1eeebf869d" />
<img width="1920" height="1020" alt="Screenshot 2026-04-16 205628" src="https://github.com/user-attachments/assets/b18e9707-a3ae-4106-b98d-bfa489c302dd" />
<img width="1920" height="1020" alt="Screenshot 2026-04-16 205532" src="https://github.com/user-attachments/assets/e14b1764-ed8e-4cad-9b24-0671e05900d2" />
Data Cleaning Summary
ไม่พบค่า Missing Values และ Duplicate Records
ตรวจสอบและปรับชนิดข้อมูลของตัวแปร (Data Types) ให้เหมาะสมกับการวิเคราะห์



(แปะรูป)
