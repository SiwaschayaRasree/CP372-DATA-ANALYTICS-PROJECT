# CP372-DATA-ANALYTICS-PROJECT


prompt :
Act as a Data Engineer and Revenue Management Expert. Generate a synthetic hotel dataset in .xlsx format for the year 2025 (Jan 1 – Dec 31). The dataset must contain 5 relational tables, each placed in a separate worksheet in the Excel file. The dataset must simulate a Revenue Stagnation problem where the hotel shows high occupancy but low RevPAR compared to competitors.
1. Hotel Configuration Total Hotel Capacity: 90 rooms. Room types exist for pricing and booking purposes. You must use the room_type_id format:

SLG (Single): 50 rooms (Base Rate: $80, room_type_id: rt-01)
DLX (Deluxe): 30 rooms (Base Rate: $150, room_type_id: rt-02)
STE (Suite): 10 rooms (Base Rate: $350, room_type_id: rt-03) Important rule: While bookings in fact_bookings must specify the room_type_id, the daily room inventory and availability limits in dim_room_inventory must be combined and calculated at the total hotel level (90 rooms).
2. Seasons (Tropical / City Destination)

High Season: January – March, November – December
Shoulder Season: April – May, September – October
Low Season: June – August
3. Table Requirements The Excel file must contain the following worksheets.
Table 1: fact_bookings

Rows: 5,000 – 7,000 rows
Each row represents one reservation transaction.
Columns: booking_id (Primary Key), guest_id (Must follow the pattern g-xxxx, e.g., g-0001, g-0125), booking_date, check_in_date, check_out_date, room_type_id (rt-01, rt-02, rt-03), rate_code_id, channel_id, segment_id, status (Confirmed, Cancelled, No-Show), total_room_revenue, number_of_rooms, adults_count, children_count
Important constraints: Maximum Length of Stay: The length of stay (the difference between check_in_date and check_out_date) must be between 1 and 3 days maximum. number_of_rooms must represent how many rooms were booked in that reservation. The room nights between check_in_date and check_out_date must consume total room inventory. Check-out date must always be after check-in date.
Table 2: dim_room_inventory

Rows: 365 rows (one per date)
Columns: date, total_capacity (fixed = 90), rooms_out_of_order, rooms_available_for_sale
Important rules: Inventory is pooled at the total hotel level. Do not separate by room type. Calculation rule: rooms_available_for_sale = total_capacity − rooms_out_of_order. Logical Consistency: The rooms_out_of_order (maintenance rooms) must logically align with the sum of number_of_rooms booked per date in fact_bookings. Specifically, the equation rooms_out_of_order + total daily occupied rooms MUST NEVER exceed total_capacity (90).
Table 3: dim_rate_codes Include the following rows:

RACK - Rack Rate
AAA - AAA Discount
CORP - Corporate
NRF - Non-Refundable
PROMO - Seasonal Promo
Table 4: dim_channels Include the following:

DIRECT - Direct Website (0% commission)
OTA_EXP - Expedia (18% commission)
OTA_BKG - Booking.com (18% commission)
WALKIN - Walk-in (0% commission)
CORPC - Corporate (10% commission)
Table 5: dim_calendar

Rows: 365
Columns: date, day_name, is_weekend, is_holiday, season
4. Hidden Revenue Management Patterns The dataset must intentionally simulate Revenue Management inefficiencies.

4.1 The Volume Trap: Overall hotel occupancy should appear high (75–85%). However, bookings must be concentrated in rt-01 rooms and OTA channels (Expedia / Booking.com). Result: High occupancy, Low ADR, High commission costs.
4.2 Pricing Inefficiency: Weekend pricing must be poorly optimized. Rule: Weekend revenue should be only 5–10% higher than weekdays despite higher demand, simulating lost revenue opportunity.
4.3 The Suite Gap: rt-03 rooms must show very low occupancy (<20%). Reasons embedded in data: rt-03 rarely receives discounts, very few PROMO rate codes are applied to rt-03, and most demand flows into rt-01.
4.4 Lead Time Problem: Booking behavior should reflect inventory blocking by low-value demand. High-value demand: Direct bookings, short lead time (0–14 days before check-in). Low-value demand: OTA bookings, long lead time (30–120 days).
4.5 The Seasonality Illusion (High/Low Season Mismanagement): The data must reflect a critical yield management failure across different seasons.
High Season Failure (Displacement): During High Season, occupancy reaches 95-100%, but RevPAR remains suppressed. The hidden reason: The hotel forgets to close out PROMO rates and OTA channels. Over 70% of High Season capacity is booked 60-90 days in advance via OTA_EXP and OTA_BKG using PROMO or NRF rates, completely blocking out short-lead, full-price RACK demand from DIRECT channels.
Low Season Failure (Stubborn Pricing): During Low Season, the hotel refuses to discount or adapt. The dataset must show almost zero PROMO rate usage during June–August, causing occupancy to plummet to < 35%. They miss the opportunity to capture budget-conscious volume to offset fixed costs.
5. Returning Guest Pattern (Customer Loyalty Signal) The dataset must simulate a small proportion of returning guests.

Rule: Exactly 2% of guests must be returning guests.
Implementation: Returning guests are identified strictly by exactly duplicate guest_id values (e.g., g-0125 appearing multiple times) in fact_bookings. A returning guest must have at least 2 separate reservations. The second booking's check_in_date must occur after the first stay's check_out_date. Guest distribution: ~98% one-time guests, ~2% repeat guests.
Returning Guest Behavior Patterns:
Channel Preference Shift: Returning guests are more likely to book via DIRECT or CORPC (At least 50–60% of returning guest bookings must use these channels).
Room Type Upgrade Tendency: Returning guests have a higher probability of booking rt-02 rooms compared to first-time guests.
Shorter Lead Time: Returning guests tend to book closer to arrival (7–21 days before check-in).
Higher Booking Value: Returning guests are less price-sensitive. They use RACK or CORP rate codes more often and use PROMO rate codes less frequently.
6. Output Format Write Python code using pandas and openpyxl to generate this dataset and save it as an Excel file (.xlsx) with five worksheets: fact_bookings, dim_room_inventory, dim_rate_codes, dim_channels, dim_calendar. All tables must be logically consistent, especially overall room capacity constraints, booking check-in/check-out dates (max 3 days stay), total inventory availability matching rooms_out_of_order constraint, occupancy calculations, and returning guest behavior patterns utilizing the g-xxxx guest ID and rt-xx room type formats. Please execute the python code to generate and provide the actual .xlsx file.

## 1. บทนำและความเป็นมา (Introduction & Background)
ในยุคปัจจุบันที่การจองห้องพักออนไลน์เข้ามามีบทบาทสำคัญ ทำให้โรงแรมมักมุ่งเน้นอัตราการเข้าพักสูง แต่กลับมองข้ามความสำคัญของรายได้สุทธิหลังหักค่าใช้จ่าย จากการวิเคราะห์ข้อมูลเบื้องต้นของโรงแรม พบว่าแม้จะมีตัวเลขการเข้าพักที่น่าพอใจ แต่โครงสร้างรายได้กลับมีความเปราะบางอย่างมีนัยสำคัญ
## 2. วัตถุประสงค์ของโครงการ (Research Objectives)
 - เพื่อวิเคราะห์โครงสร้างและพฤติกรรมข้อมูลธุรกิจโรงแรม
   และศึกษาความสัมพันธ์ของข้อมูลการจอง ช่องทางการขาย และโครงสร้างราคา
   เพื่อระบุจุดรั่วไหลของรายได้ ผ่านสมมติฐานหลักทั้ง 6 ข้อ
 - เพื่อเสนอแนะแนวทางการบริหารจัดการรายได้เชิงกลยุทธ์   
   สร้างข้อเสนอแนะในกลยุทธ์การตั้งราคาที่เป็นรูปธรรม
 - เพื่อทบทวนและบูรณาการความรู้ในวิชา CP372 Data analytics   
   การนำทฤษฎีและทักษะที่ได้เรียนรู้มาประยุกต์ใช้กับโจทย์ธุรกิจจริง

## 3. คำถามการวิจัยและสมมติฐาน (Research Questions & Hypothesis)
**hypothesis 1 : The Volume trap** : แม้โรงแรมจะมีอัตราการเข้าพัก (Occupancy Rate) สูงถึง 71.4% แต่รายได้เฉลี่ยต่อห้อง (Average Daily Rate) กลับต่ำกว่าที่ควรจะเป็น เนื่องจากยอดการจองห้องยอดใหญ่เป็นการจองห้องประเภท single (rt-01) เป็นหลัก และมาจากช่องทาง OTA (Expedia, [Booking.com](http://booking.com)) ทำให้โรงแรมต้องแบกรับต้นทุนค่าคอมมิชชันที่สูงจนกินกำไรสุทธิ 

**hypothesis 2 : The Suite Gap** : อัตราการเข้าพักของห้องสวีท (rt-03) ที่อยู่ในระดับต่ำมาก เป็นผลมาจากการวางโครงสร้างเรตราคาที่แข็งเกินไป (Rigid Pricing) และการไม่นำ PROMO code มาปรับใช้กับห้องประเภทนี้ ทำให้ลูกค้าเลือกที่จะจองห้องประเภทเริ่มต้น (rt-01) หรือไปพักโรงแรมคู่แข่งแทน

**hypothesis 3 : Pricing Inefficiency** : โรงแรมสูญเสียโอกาสในการทำกำไร (Revenue Opportunity Loss) ในช่วงวันหยุดสุดสัปดาห์ เนื่องจากโครงสร้างราคาไม่ได้ถูกปรับให้สอดคล้องกับความต้องการ (Demand) ที่สูงขึ้นอย่างแท้จริง ส่งผลให้รายได้เฉลี่ยในช่วงสุดสัปดาห์เติบโตสูงกว่าวันธรรมดาเพียงแค่ 5-10% เท่านั้น

**hypothesis 4 : Lead Time Problem** : รายได้โดยรวมของโรงแรมถูกกดทับจากการที่กลุ่มลูกค้าที่ได้ราคาถูก (Low-value demand จาก OTA) ซึ่งมีพฤติกรรมจองล่วงหน้านาน (30-120 วัน) เข้ามาจองจนเต็มความจุ ส่งผลให้เป็นการบล็อก (Block) พื้นที่ของกลุ่มลูกค้าที่ยอมจ่ายราคาเต็ม (High-value demand จาก Direct/Walk-in) ที่มักมีพฤติกรรมการจองในระยะกระชั้นชิด (0-14 วัน)

**hypothesis 5 : The Seasonality Illusion** : โรงแรมขาดกลยุทธ์การตั้งราคาที่มีประสิทธิภาพทั้ง 2 ฤดู โดยในช่วง High Season มีการปล่อยห้องด้วยราคา PROMO ผ่าน OTA ล่วงหน้านานเกินไปจนสูญเสียลูกค้าที่พร้อมจ่ายราคา RACK (Displacement Effect) และในทางกลับกัน ช่วง Low Season โรงแรมกลับดื้อรั้นที่จะคงราคาเดิมไว้ (Stubborn Pricing) โดยไม่ใช้กลยุทธ์ลดราคาเพื่อดึงวอลลุ่ม ส่งผลให้ Occupancy ร่วงลง

hypothesis 6 : The Loyalty Leak : แม้โรงแรมจะมีกลุ่มลูกค้าที่ประทับใจและกลับมาเข้าพักซ้ำ (Returning Customers) แต่โรงแรมกลับล้มเหลวโดยสิ้นเชิงในการดึงลูกค้ากลุ่มนี้ให้เปลี่ยนพฤติกรรมมา 'จองตรง' (Direct Booking) ส่งผลให้ลูกค้าเก่าส่วนใหญ่ยังคงกลับไปจองผ่านช่องทาง OTA ซ้ำแล้วซ้ำเล่า ทำให้โรงแรมต้องเสียค่าคอมมิชชัน (Commission Cost) ซ้ำซ้อนให้กับลูกค้าคนเดิมแบบไม่รู้จบ


## 4. ชุดข้อมูลและตัวแปรที่ใช้ (Dataset & Features)
* จำนวนแถวข้อมูล: 7806 แถว
* จำนวนตัวแปรทั้งหมด: 14 ตัวแปร

### Data Dictionary
1. Sheet: fact_bookings

| Attribute | คำอธิบาย | Data Type | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
|---|---|---|---|
| booking_id | หมายเลขการจอง | Nominal (Text) | b-002266, b-003216 |
| guest_id | หมายเลขผู้เข้าพัก | Nominal (Text) | g-2157, g-3107 |
| booking_date | วันที่ทำการจอง | Interval (Date) | 26/09/2024 |
| check_in_date | วันที่เช็คอิน | Interval (Date) | 01/01/2025 |
| check_out_date | วันที่เช็คเอาท์ | Interval (Date) | 03/01/2025 |
| room_type_id | ประเภทห้องพัก | Nominal | rt-01, rt-02, rt-03 |
| rate_code_id | รหัสอัตราค่าห้อง | Nominal | NRF, PROMO, AAA, RACK, CORP |
| channel_id | ช่องทางการจอง | Nominal | OTA_BKG, OTA_EXP, DIRECT, WALKIN, CORPC |
| segment_id | กลุ่มลูกค้า | Nominal | LEISURE, BUSINESS |
| status | สถานะการจอง | Nominal | Confirmed, Cancelled, No-Show |
| total_room_revenue | รายได้รวมจากห้องพัก (USD) | Ratio (Continuous) | 0 – 4,200 |
| number_of_rooms | จำนวนห้องที่จอง | Ratio (Discrete) | 1 – 4 |
| adults_count | จำนวนผู้ใหญ่ | Ratio (Discrete) | 1 – 8 |
| children_count | จำนวนเด็ก | Ratio (Discrete) | 0 – 4 |

2. Sheet: dim_room_inventory

| Attribute | คำอธิบาย | Data Type | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
|---|---|---|---|
| date | วันที่ | Interval (Date) | 01/01/2025 |
| total_capacity | จำนวนห้องทั้งหมดของโรงแรม | Ratio (Discrete) | 100 – 500 |
| rooms_out_of_order | ห้องที่ไม่สามารถขายได้ (ซ่อม/ปิดปรับปรุง) | Ratio (Discrete) | 0 – 50 |
| rooms_available_for_sale | ห้องที่พร้อมขาย | Ratio (Discrete) | 50 – 500 |

3. Sheet: dim_rate_codes

| Attribute | คำอธิบาย | Data Type | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
|---|---|---|---|
| rate_code_id | รหัสอัตราค่าห้อง | Nominal | NRF, PROMO, AAA, RACK, CORP |
| rate_name | ชื่อเรท | Nominal (Text) | Non-refundable, Promotion, Corporate Rate |
| description | รายละเอียดเรท / สิ่งที่รวมอยู่ | Nominal (Text) | Includes Breakfast & Wifi |
| is_commissionable | ระบุว่ามีการจ่ายค่าคอมมิชชั่นหรือไม่ | Boolean | True, False |

4. Sheet: dim_channels

| Attribute | คำอธิบาย | Data Type | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
|---|---|---|---|
| channel_id | รหัสช่องทางการจอง | Nominal | OTA_BKG, OTA_EXP, DIRECT |
| channel_name | ชื่อช่องทาง | Nominal (Text) | Booking.com, Expedia, Direct |
| channel_type | ประเภทช่องทางการจอง | Nominal | OTA, Direct, Wholesaler |
| commission_pct | เปอร์เซ็นต์ค่าคอมมิชชั่น | Ratio (Continuous) | 0 – 0.30 (เช่น 0.15 = 15%) |

5. Sheet: dim_calendar

| Attribute | คำอธิบาย | Data Type | ช่วงค่าที่ถูกต้อง / ตัวอย่าง |
|---|---|---|---|
| date | วันที่ | Interval (Date) | 01/01/2025 |
| day_name | ชื่อวันในสัปดาห์ | Nominal | Monday, Tuesday |
| is_weekend | เป็นวันหยุดสุดสัปดาห์หรือไม่ | Boolean (0 = False, 1 = True) | 0, 1 |
| is_holiday | เป็นวันหยุดนักขัตฤกษ์หรือไม่ | Boolean (0 = False, 1 = True) | 0, 1 |
| season | ฤดูกาล | Nominal | High, Low, Shoulder |


### ตัวแปรเป้าหมาย (Target Variable)
**สมมติฐานข้อที่ 1 ปริมาณการจองลวงตา (The Volume Trap)**
**ตัวแปรเป้าหมาย :**
Net Revenue : รายได้สุทธิหลังหักค่าคอมมิชชัน (Total Room Revenue - Commission Cost)
Commission Cost : ต้นทุนค่าคอมมิชชันที่จ่ายให้ OTA
**ตัวแปรประกอบ :** 
ADR (Average Daily Rate) และ สัดส่วนการจองแยกตามประเภทห้อง (Room Type)

**สมมติฐานข้อที่ 2 ช่องว่างของห้องพักระดับบน (The Suite Gap)**
**ตัวแปรเป้าหมาย :** 
Occupancy Rate (rt-03) : อัตราการเข้าพักเฉพาะของห้องประเภทสวีท
Rate Code Distribution: สัดส่วนการใช้ราคา RACK เทียบกับ PROMO สำหรับห้องสวีท
**ตัวแปรประกอบ :** 
ราคาขายเฉลี่ยของห้อง rt-03 เทียบกับ rt-01

**สมมติฐานข้อที่ 3 ความไร้ประสิทธิภาพในการตั้งราคาช่วงสุดสัปดาห์ (Pricing Inefficiency)** 
**ตัวแปรเป้าหมาย :**
 ADR (Weekend vs Weekday) : เปรียบเทียบราคาเฉลี่ยต่อห้องระหว่างวันธรรมดาและวันหยุด
Revenue Opportunity Loss: มูลค่ารายได้ที่หายไปจากการไม่ปรับราคา
**ตัวแปรประกอบ :** 
สัดส่วน Rate Code (PROMO vs RACK) ในช่วงวันหยุด


**สมมติฐานข้อที่ 4 ปัญหาการแย่งชิงโควตาจากระยะเวลาการจอง (Lead Time Problem)**
**ตัวแปรเป้าหมาย :** 
Revenue per Booking: รายได้ต่อการจองหนึ่งครั้ง
ADR by Lead Time Group: ราคาเฉลี่ยที่ได้ตามระยะเวลาการจอง (0-14 วัน vs 60+ วัน)
**ตัวแปรประกอบ :** 
Lead Time (ระยะเวลาจองล่วงหน้า) และ Channel ID

**สมมติฐานข้อที่ 5 ความล้มเหลวในการบริหารจัดการฤดูกาล (The Seasonality Illusion)**
**ตัวแปรเป้าหมาย :** 
Occupancy Rate by Season: อัตราการเข้าพักแยกตามหน้า High และ Low Season
Yield Percentage: ประสิทธิภาพการทำกำไรในแต่ละเดือน
**ตัวแปรประกอบ :** 
การกระจายตัวของ Rate Code ในแต่ละเดือน

**สมมติฐานข้อที่ 6: รอยรั่วของฐานลูกค้าประจำ (The Loyalty Leak)**
**ตัวแปรเป้าหมาย :** 
Retention Rate: อัตราการกลับมาพักซ้ำของลูกค้า (Returning vs New Guests)
Channel Conversion Rate: สัดส่วนลูกค้าเก่าที่เปลี่ยนจากจองผ่าน OTA มาเป็นจองตรง (Direct)
**ตัวแปรประกอบ :** 
Guest ID และ Channel ID (เปรียบเทียบครั้งแรกกับครั้งที่สอง)


## 5. ระเบียบวิธีวิจัย (Methodology)
**5.1 Data Cleaning**

 **Table fact bookings**
<img width="1920" height="1020" alt="Screenshot 2026-04-17 231355" src="https://github.com/user-attachments/assets/19a413cb-acd1-4401-bbf2-fb49ca06bfbd" />

 - ไม่พบค่า Missing Values และ มีบางส่วนที่เป็นค่า Duplicate Records
   เนื่องจากการที่ลูกค้ากลับมาใช้ซ้ำ (guest_id)
 - ตรวจสอบและปรับชนิดข้อมูลของตัวแปร (Data Types)   
   ให้เหมาะสมกับการวิเคราะห์

<img width="1920" height="1020" alt="Screenshot 2026-04-17 231404" src="https://github.com/user-attachments/assets/078137f0-0ebd-462e-b229-19646c260b3a" />

<img width="1920" height="1020" alt="Screenshot 2026-04-17 231410" src="https://github.com/user-attachments/assets/c6aca2b6-af43-451c-9003-7a229254b444" />

<img width="1920" height="1020" alt="Screenshot 2026-04-17 231440" src="https://github.com/user-attachments/assets/185ccecf-b112-4ed5-b62d-9fc39ab14141" />

<img width="1920" height="1020" alt="Screenshot 2026-04-17 231445" src="https://github.com/user-attachments/assets/3c3ecb21-8a42-4d88-b3a3-16778f223b30" />

ใน **Table room inventory , Table calendar , Table rate codes และ Table channels**

 - ไม่พบค่า Missing Values และ Duplicate Records
 - ตรวจสอบและปรับชนิดข้อมูลของตัวแปร (Data Types)   
   ให้เหมาะสมกับการวิเคราะห์
   
**5.2 Exploratory Data Analysis (EDA)**

**วิเคราะห์สมมติฐานข้อที่ 1 ปริมาณการจองลวงตา (The Volume Trap)**
แม้โรงแรมจะมีอัตราการเข้าพัก (Occupancy Rate) สูงถึง 71.4% แต่รายได้เฉลี่ยต่อห้อง (Average Daily Rate) กลับต่ำกว่าที่ควรจะเป็น เนื่องจากยอดการจองห้องยอดใหญ่เป็นการจองห้องประเภท single (rt-01) เป็นหลัก และมาจากช่องทาง OTA (Expedia, [Booking.com](http://booking.com)) ทำให้โรงแรมต้องแบกรับต้นทุนค่าคอมมิชชันที่สูงจนกินกำไรสุทธิ

pic 01

จากกราฟ พบว่าโรงแรมพึ่งพาช่องทาง OTA_BKG, OTA_EXP และ CORPC สูงถึง 73.88% (โดยเป็น OTA 68.5%) ขณะที่ช่องทางที่โรงแรมได้รับรายได้เต็มจำนวน ได้แก่ Direct และ Walk-in มีสัดส่วนรวมเพียง 26.12% เท่านั้น

แม้ OTA จะเป็นช่องทางหลักที่สร้างรายได้สูงตามกราฟ Total Room Revenue แต่ก็มาพร้อมต้นทุนค่า Commission ที่สูง ส่งผลให้โรงแรมสูญเสียรายได้รวมถึง 293,137 ดอลลาร์

นอกจากนี้ ลูกค้าส่วนใหญ่เลือกห้อง rt-01 มากกว่า rt-03 อย่างชัดเจน 

หลังจากการวิเคราะห์ทางผู้จัดทำได้เห็นว่าเมื่อนำ 2 ภาพด้านบนมาเปรียบเทียบเพื่อวิเคราะห์ต่อทำให้เราได้ทราบว่า หากต้องการที่จะเพิ่มยอดขาย ทางโรงแรมควรที่จะกันห้องไว้สำหรับ direct และ walk in ทำส่วนลดเฉพาะเว็บไซต์โรงแรม (เช่น -5%) ให้สิทธิพิเศษ: Late check-out, Free breakfast และลดโควตาห้องที่ขายบน 3rd party ลงเพื่อลดราคาสำหรับการจ่ายค่า commission สำหรับปัญหาด้านการที่ลูกค้าเข้าพักเฉพาะห้อง single ทางโรงแรมควรจัดโปรโมชันสำหรับการอัพเกรดห้องเพื่อปรับเป็นdeluxe (rt-02)

จาก hypothesis ที่ 1 พบว่าช่องว่างของ roomtype มีความห่างกันมาก ผู้จัดทำจึงยกมาวิเคราะห์ต่อใน hypothesis ที่ 2
**วิเคราะห์สมมติฐานข้อที่ 2 ช่องว่างของห้องพักระดับบน (The Suite Gap)**
อัตราการเข้าพักของห้องสวีท (rt-03) ที่อยู่ในระดับต่ำมาก เป็นผลมาจากการวางโครงสร้างเรตราคาที่แข็งเกินไป (Rigid Pricing) และการไม่นำ PROMO code มาปรับใช้กับห้องประเภทนี้ ทำให้ลูกค้าเลือกที่จะจองห้องประเภทเริ่มต้น (rt-01) หรือไปพักโรงแรมคู่แข่งแทน

pic02

กราฟแสดงจำนวนห้องทั้งหมดที่ขายออก (Room nights) แบ่งตามประเภทห้อง

 - rt-01 (ห้องประเภทเริ่มต้น): มีอัตราการเข้าพักสูงสุดที่ 64.14%
 - rt-02 (ห้องระดับกลาง): อัตราการเข้าพักลดลงมาอยู่ที่ 30.21%
 - rt-03 (ห้องสวีท): อัตราการเข้าพักต่ำที่สุด ตกหลุมลงไปเหลือเพียง 5.65%

กราฟแสดงอัตราการเข้าพัก (Occupancy Rate) แบ่งตามประเภทห้อง

 - rt-01 (ห้องประเภทเริ่มต้น): มีอัตราการเข้าพักสูงสุดที่ 45.79%
 - rt-02 (ห้องระดับกลาง): อัตราการเข้าพักลดลงมาอยู่ที่ 22.82%
 - rt-03 (ห้องสวีท): อัตราการเข้าพักต่ำที่สุด ตกหลุมลงไปเหลือเพียง 7.09%

จากการตรวจสอบสมมติฐานนั้นทำให้เราได้ทราบว่าอัตราการเข้าพักของห้องสวีท (rt-03) อยู่ในระดับต่ำมาก (น้อยกว่า 10%) 

เมื่อเปรียบเทียบสัดส่วนของ Rate Code ที่ขายได้ในแต่ละประเภทห้อง (Room Type) 

 - ห้อง single และห้อง deluxe (rt-01 & rt-02)
   มีการกระจายตัวของเรตราคาที่สมดุล โดยมีสัดส่วนของ PROMO (สีส้ม) และ
   RACK (สีชมพู) อยู่ที่ประมาณ 29-32%
 - ห้อง suite (rt-03) มีความผิดปกติอย่างชัดเจน สัดส่วนกว่า 54.64%
   ถูกขายในราคา RACK (ราคาเต็ม) รองลงมาคือ NRF (35.33%) และไม่มีการใช้
   PROMO
   
กราฟนี้พิสูจน์ว่าการวางโครงสร้างเรทราคาที่แข็งเกินไป และการไม่ใช้ Promotion ทำให้ลูกค้าถูกบังคับให้ต้องจ่ายในราคาเต็ม (RACK) หรือแบบซื้อขาด (NRF) เท่านั้น

ทั้งนี้ผู้จัดทำจึงได้เสนอแนวทางแก้ไขปัญหานี้ หากทางโรงแรมเพิ่มโปรโมชันสำหรับห้องพักประเภท rt-03 เพื่อเพิ่มความน่าสนใจ และเป็นการกระตุ้นให้ลูกค้าสนใจหากว่าถ้าจ่ายห้องพักประเภท rt-02 เพิ่มเงินขึ้นอีก 15-20% เพื่ออัพเกรดห้อง พร้อมบริการเสริม อาทิเช่น ฟรีอาหารเช้า, บริการสปา, บริการรถรับ-ส่งสนามบิน หรือบริการซักรีด เป็นต้น


**วิเคราะห์สมมติฐานข้อที่ 3 ความไร้ประสิทธิภาพในการตั้งราคาช่วงสุดสัปดาห์ (Pricing Inefficiency)** 
โรงแรมสูญเสียโอกาสในการทำกำไร (Revenue Opportunity Loss) ในช่วงวันหยุดสุดสัปดาห์ เนื่องจากโครงสร้างราคาไม่ได้ถูกปรับให้สอดคล้องกับความต้องการ (Demand) ที่สูงขึ้นอย่างแท้จริง ส่งผลให้รายได้เฉลี่ยในช่วงสุดสัปดาห์เติบโตสูงกว่าวันธรรมดาเพียงแค่ 5-10% เท่านั้น

pic03

จากกราฟนี้เป็นการพูดถึงรายได้เฉลี่ยต่อวัน (Average Daily Rate) ในช่วงวันธรรมดาอยู่ที่ 100 ดอลลาร์ และวันหยุดสุดสัปดาห์อยู่ที่ 108 ดอลลาร์ ซึ่งหมายความว่าทางโรงแรมสามารถทำยอดขายได้เพิ่มจากวันธรรมดาเพียงแค่ 8.16% เท่านั้น ซึ่งตรงกับสมมติฐาน และจากยอดความต้องการ ของวันธรรมดาและวันหยุดสุดสัปดาห์ 71.42 และ 71.34 ตามลำดับ พบว่ามีความต้องการในอัตราที่ใกล้เคียงกันดังนั้นจึงไม่สามารถรองรับสมมติฐานเกี่ยวกับความต้องการที่พุ่งขึ้นสูงในช่วงวันหยุดสุดสัปดาห์ได้ จึงดูการใช้ code ID และ channel ID แทน และพบว่าในช่วงวันหยุดสุดสัปดาห์ มีลูกค้าเลือกจ่ายเต็มเพียงแค่ 31.55% และมีลูกค้าที่จองผ่าน OTA รวม 70.25 ทำให้เสียรายได้ในช่วงวันหยุดสุดสัปดาห์ไป

จากกราฟทั้งสามกราฟ สรุปได้ว่า demand ของวันธรรรมดาและวันหยุดสูงพอ ๆ กัน ทางผู้จัดทำจึง visualize กราฟอีกหนึ่งกราฟขึ้นมาดู และพบว่า เพราะส่วนใหญ่ weekend มีการปล่อย promotion ที่สูงเกินไป ทำให้รายได้ต่ำ ดังนั้น นอกจากปรับราคาห้องในช่วง weekend ให้สูงขึ้น เราควรปิดการใช้ promotion นอกจากนี้ควรลดจำนวนปริมาณห้องที่ปล่อยประเภท OTA เพื่อลดการเพิ่มต้นทุนจากค่า commission เพื่อรับรายได้ที่สูงขึ้น

**วิเคราะห์สมมติฐานข้อที่ 4 พฤติกรรมการเข้าพักของลูกค้า (Lead Time Problem)** 
รายได้โดยรวมของโรงแรมถูกกดทับจากการที่กลุ่มลูกค้าที่ได้ราคาถูก (Low-value demand จาก OTA) ซึ่งมีพฤติกรรมจองล่วงหน้านาน (30-120 วัน) เข้ามาจองจนเต็มความจุ ส่งผลให้เป็นการบล็อก (Block) พื้นที่ของกลุ่มลูกค้าที่ยอมจ่ายราคาเต็ม (High-value demand จาก Direct/Walk-in) ที่มักมีพฤติกรรมการจองในระยะกระชั้นชิด (0-14 วัน)

pic04

กราฟนี้สนับสนุนพฤติกรรมลูกค้า (Customer Behavior) ตามที่สมมติฐานที่เราตั้งไว้

 - OTA คือ Low-value demand ลูกค้าชอบที่จะจองล่วงหน้า 
 - Direct/Walk-in คือกลุ่มที่จองกระชั้นชิด และมีแนวโน้มที่จะสร้างรายได้ให้โรงแรมมากกว่า

เมื่อเราดูสัดส่วนการจองห้องพัก (Room Nights) ที่แบ่งตามระยะเวลาการจองล่วงหน้า (Lead Time Groups) กราฟแสดงให้เห็นถึงการแบ่งแยกพฤติกรรม

 - กลุ่มจองล่วงหน้านาน (60+ Days) : ส่วนใหญ่จะจองผ่านทาง OTA ทั้งหมด
   แบ่งเป็น OTA_BKG (สีเหลือง) 50.55% และ OTA_EXP (สีชมพู) 49.45%
 - กลุ่มจองระยะกลาง (15-60 Days) : ฝั่ง OTA
   ก็ยังคงเป็นสัดส่วนใหญ่ที่สุด(เกิน 50-70%)
 - กลุ่มจองระยะกระชั้นชิด (0-14 Days) : ในช่วง 8-14 Days ลูกค้า Direct
   (สีส้ม) มีการจองอยู่ที่ 72.32% และในช่วง 0-7 Days จะเป็นกลุ่ม Direct
   58.74% คู่กับ Walk-in (สีเขียวมิ้นต์) 37.12%

กราฟเส้นแสดงให้เห็นถึงการเปลี่ยนแปลงของราคาเฉลี่ย (ADR)ตามระยะเวลาการจองล่วงหน้า 

 - กลุ่มจองล่วงหน้านาน (15 ถึง 60+ Days) : กราฟจะกองอยู่ในระดับต่ำ โดย
   ADR อยู่แค่ช่วง 94.73 - 100.58 ดอลลาร์
 - กลุ่มจองระยะกระชั้นชิด (0 ถึง 14 Days) : เมื่อขยับมาที่ช่วง 8-14 วัน
   กราฟพุ่งชันขึ้นอย่างเห็นได้ชัด 116.21 ดอลลาร์และไปแตะจุดสูงสุดที่ช่วง
   0-7 วัน ด้วย ADR ถึง 121.98 ดอลลาร์

เมื่อนำกราฟเส้นนี้ ไปประกอบกับกราฟแท่งสีๆ (Volume Blockage) ก่อนหน้านี้จะได้ข้อสรุปว่า กราฟก่อนหน้าบอกเราว่า กลุ่ม 15-60+ Days คือ ลูกค้า OTA ลูกค้ากลุ่มนั้นจ่ายเงินให้เรา ~94-100 ดอลลาร์/คืน (Low-value demand) และกราฟก่อนหน้าบอกเราว่า กลุ่ม 0-14 Days คือ ลูกค้า Direct & Walk-in บ่งบอกว่า ลูกค้ากลุ่มนี้ยินดีจ่ายเงินสูงถึง ~116-122 ดอลลาร์/คืน (High-value demand) ซึ่งหมายถึงความเต็มใจจ่าย และ High-value Demand

หลังจากการวิเคราะห์ทางผู้จัดทำคาดว่าหากเรานำระบบจัดการโควตา (Allotment) มาใช้โดยตั้งกฎว่าสำหรับช่วงเวลาที่ล่วงหน้ามากกว่า 30-60 วัน (Lead Time > 30 Days) โรงแรมจะเปิดโควตาให้ช่องทาง OTA (Booking/Expedia) ไม่เกิน 40-50% ของจำนวนห้องทั้งหมด ส่วนห้องที่เหลืออีก 50% ให้สำรองไว้ เพื่อนำมาเปิดขายเฉพาะช่องทาง Direct และ Walk-in ในช่วงระยะเวลา 0-14 วันก่อนวันเข้าพัก และสำหรับ ลูกค้า Direct มักจะรอจองกระชั้นชิด (0-14 วัน) เราอาจจัดการด้วยการออกแคมเปญ "Direct Early-Bird " มอบส่วนลดพิเศษ 10% หรือแถมอาหารเช้าฟรี หากจองตรงผ่านเว็บไซต์ล่วงหน้า 30 วันขึ้นไป


**วิเคราะห์สมมติฐานข้อที่ 5 กำไรที่สวนทางกับช่วงฤดูกาล (The Seasonality Illusion)** 
โรงแรมขาดกลยุทธ์การตั้งราคาที่มีประสิทธิภาพทั้ง 2 ฤดู โดยในช่วง High Season มีการปล่อยห้องด้วยราคา PROMO ผ่าน OTA ล่วงหน้านานเกินไปจนสูญเสียลูกค้าที่พร้อมจ่ายราคา RACK (Displacement Effect) และในทางกลับกัน ช่วง Low Season โรงแรมกลับดื้อรั้นที่จะคงราคาเดิมไว้ (Stubborn Pricing) โดยไม่ใช้กลยุทธ์ลดราคาเพื่อดึงวอลลุ่ม ส่งผลให้ Occupancy ร่วงลง

pic05

จากแดชบอร์ดจะพบว่าช่วง high season รวมถึงช่วงคาบเกี่ยวอย่าง shoulder จะมีการเข้าพักที่สูงลดหลั่นตามกันมา และทิ้ง gap ระยะห่างของ low season ไว้เยอะ และยอดขายของ high season รวมถึงได้ 58.13 ซึ่งถือว่าอยู่ในเกณฑ์ที่ดี แต่เมื่อดูตัวแปร channel ID และ rate code ID จะพบว่าช่วง high season จะมีอัตราการปล่อยห้องให้จองผ่าน OTA สูงมาก และมีอัตราการใช้ Promotion ที่สูง แต่ในทางกลับกันช่วง low season ที่มียอดขายรวมกันเพียงแค่ 9.99% ที่แทบไม่มีการปล่อยให้จองห้องพักผ่าน OTA หรือการกระตุ้นยอดขายจากการใช้ Promotion

 - การรับมือสำหรับในช่วง High Season จำกัดโควตา ของช่องทาง OTA และ Rate
   Code ประเภท PROMO/NRF ไม่ให้เกิน 30-40% ของจำนวนห้องทั้งหมดในช่วง
   High Season ใช้ระบบจัดการโควตา (Yield Management) เพื่อสงวนห้องพัก
   50-60% ไว้ขายเฉพาะช่องทาง DIRECT และในเรทราคา RACK ในช่วง 1-3
   สัปดาห์ก่อนวันเข้าพัก (Lead Time 0-21 วัน) ซึ่งเป็นช่วงที่ Demand
   ของลูกค้าพร้อมจ่ายสูงสุด และควรระงับการขายผ่านช่องทาง OTA
   ในช่วงวันหยุดยาว
   
 - สำหรับในช่วง Low Season โรงแรมควรปรับกลยุทธ์มาอัดโปรโมชัน Flash Sale
   หรือ Buy 1 Get 1 Free (เรท PROMO) เพื่อดึง Volume เข้ามาหล่อเลี้ยง
   Fix Cost และครอบคลุมค่าใช้จ่ายดำเนินงาน ไม่ลดราคาจนเสียสมดุล
   แต่ใช้การแถม (เช่น ฟรีอาหารเช้า, สปา, รถรับส่ง)
   เพื่อกระตุ้นการตัดสินใจ

วิเคราะห์สมมติฐานข้อที่ 6 The Loyalty Leak 
แม้โรงแรมจะมีกลุ่มลูกค้าที่ประทับใจและกลับมาเข้าพักซ้ำ (Returning Customers) แต่โรงแรมกลับล้มเหลวโดยสิ้นเชิงในการดึงลูกค้ากลุ่มนี้ให้เปลี่ยนพฤติกรรมมา 'จองตรง' (Direct Booking) ส่งผลให้ลูกค้าเก่าส่วนใหญ่ยังคงกลับไปจองผ่านช่องทาง OTA ซ้ำแล้วซ้ำเล่า ทำให้โรงแรมต้องเสียค่าคอมมิชชัน (Commission Cost) ซ้ำซ้อนให้กับลูกค้าคนเดิมแบบไม่รู้จบ

**เมทริกซ์สหสัมพันธ์ของการทำกำไรของโรงแรมและตัวแปรจากทั้ง 6 สมมติฐาน**
pic correlation


จาก correlation matrix สรุปได้ดังนี้
lead_time กับ commission_rate (0.66) — สูงที่สุด ยืนยัน สมมติฐานข้อที่ 4 ชัดเจน OTA ที่คิด commission สูงมีพฤติกรรมจองล่วงหน้านาน ซึ่งเป็นต้นเหตุของการที่โรงแรมเสียโอกาสในการทำรายได้จากกลุ่มลูกค้า high demand หรือ displacement effect
lead_time กับ net_revenue (-0.11) ยิ่งจองล่วงหน้านาน ยิ่งทำให้ได้รายได้สุทธิน้อย สอดคล้องกับสมมติฐานข้อที่ 1 ที่ว่าเราเสียไปกับค่า commission จำนวนมาก
commission_rate กับ net_revenue (-0.16) ยิ่ง commission สูง ยิ่งได้ net revenue น้อย แต่มีนัยสำคัญเพราะ total_revenue กับ net_revenue สูงถึง 0.99 นั่นหมายความว่าที่ค่า net_revenue มาจากค่า commission 
total_revenue กับ commission_cost (0.71) ห้องที่ทำรายได้สูง มีต้นทุน commission สูงตามด้ซึ่งเป็นกับดักของสมมติฐานข้อที่ 1 ที่ขายได้มากแต่กำไรสุทธิไม่ได้มากตาม
number_of_rooms กับ net_revenue (0.43) จำนวนห้องที่จองมีความสัมพันธ์ปานกลางกับรายได้สุทธิ บอกว่า volume เพียงอย่างเดียวไม่ใช่คำตอบ ต้องดู channel และ rate code ด้วย
โดยรวมเมทริกซ์สหสัมพันธ์นี้รองรับทุกข้อสมมติฐานที่เราตั้งโดยเฉพาะสมมติฐานข้อที่ 1 และ 4

## 6. Business Insights

**วิเคราะห์สมมติฐานข้อที่ 1 ปริมาณการจองลวงตา (The Volume Trap)**
เนื่องจากรายได้ของโรงแรมส่วนใหญ่นั้นพึ่งพา OTA สูงถึง 73.88% ทำให้เกิดค่าคอมมิชชันสะสมกว่า 293,137 ดอลลาร์ และยอดจองกระจุกตัวอยู่ที่ห้อง Single (rt-01) เป็นหลัก ทำให้รายได้รวมถูกกดทับแม้จะมีอัตราการเข้าพักสูงก็ตาม

**วิเคราะห์สมมติฐานข้อที่ 2 ช่องว่างของห้องพักระดับบน (The Suite Gap)**
ห้องสวีท (rt-03) มี Occupancy ต่ำกว่า 10% เพราะโครงสร้างราคาที่แข็งทื่อ และทางโรงแรมไม่ยอมปล่อยโปรโมชัน (PROMO) ทำให้ลูกค้าต้องจ่ายราคาเต็ม (RACK) 

**วิเคราะห์สมมติฐานข้อที่ 3 ความไร้ประสิทธิภาพในการตั้งราคาช่วงสุดสัปดาห์ (Pricing Inefficiency)** 
ในช่วงวันหยุด เพราะโรงแรมไม่ยอมปิดกั้นโปรโมชันทำให้ ADR โตขึ้นเพียง 8% แม้จะมี Demand สูง ทำให้เสียโอกาสทำกำไร (Revenue Opportunity Loss) 

**วิเคราะห์สมมติฐานข้อที่ 4 พฤติกรรมการเข้าพักของลูกค้า (Lead Time Problem)** 
เกิด Displacement Effect โดยลูกค้าราคาถูก (OTA) เข้ามาจองล่วงหน้านานๆ จนเต็มความจุ ทำให้ไม่มีห้องเหลือขายให้ลูกค้า Direct/Walk-in ที่พร้อมจ่ายแพงกว่าถึง 30 ดอลลาร์ต่อคืน 

**วิเคราะห์สมมติฐานข้อที่ 5 กำไรที่สวนทางกับช่วงฤดูกาล (The Seasonality Illusion)** 
ในช่วงฤดูกาลท่องเที่ยว (High Season) โรงแรมถูก OTA แย่งจองโควตาห้องราคาถูกล่วงหน้านานเกินไปจนเต็มความจุ (Displacement Effect) ทำให้เสียโอกาสขายห้องให้ลูกค้าที่พร้อมจ่ายราคาเต็ม (Walk-in/Direct) ที่มักจองกระชั้นชิด

### Summary
ผลการวิเคราะห์ชี้ให้เห็นว่าโรงแรมมีปัญหาหลักด้าน Yield Management ที่ไม่ยืดหยุ่น การมุ่งเน้นเพียงตัวเลข Occupancy โดยไม่มีการควบคุมโควตาช่องทางขาย  และการทำ Pricing Fencing ที่มีประสิทธิภาพ ทำให้เกิดค่าเสียโอกาสจำนวนมาก
