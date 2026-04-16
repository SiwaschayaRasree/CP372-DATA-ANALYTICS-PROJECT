# CP372-DATA-ANALYTICS-PROJECT


prompt :
Act as a Data Engineer and Hotel Revenue Management Expert.
Generate a synthetic hotel dataset in Excel (.xlsx) format for 1 year (Jan 1 – Dec 31, 2025).
The dataset must contain 5 relational tables that simulate a Revenue Stagnation scenario (high occupancy but low RevPAR compared to competitors).
1. Hotel Configuration
Total hotel capacity: 120 rooms
Room Types:
Single: 70 rooms (Base Rate: $80)
Deluxe: 30 rooms (Base Rate: $150)
Suite: 20 rooms (Base Rate: $350)
Seasons (Tropical / City destination):
High Season
January – March
November – December
Shoulder Season
April – May
September – October
Low Season
June – August
2. Tables to Generate
Table 1 — fact_bookings
Target size: 5,000 – 7,000 rows
Columns:
booking_id
guest_id
booking_date
check_in_date
check_out_date
room_type_id
rate_code_id
channel_id
segment_id
status (Confirmed, Cancelled, No-Show)
total_room_revenue
number_of_rooms
adults_count
children_count
Rules:
Most bookings are 1–3 nights
number_of_rooms usually 1
Some cancellations (~5-10%)
Table 2 — dim_room_inventory
Exactly 365 rows (1 row per date)
Columns:
date
total_capacity
rooms_out_of_order
rooms_available_for_sale
Rules:
total_capacity must always equal 120
rooms_out_of_order randomly between 0–3
rooms_available_for_sale must be calculated as
rooms_available_for_sale = total_capacity - rooms_out_of_order - occupied_rooms
where occupied_rooms is derived from fact_bookings for that date
IMPORTANT DATA CONSISTENCY RULE (CRITICAL)
The dataset must ensure logical consistency between fact_bookings and dim_room_inventory.
For each date:
Calculate the number of rooms occupied from fact_bookings
where
check_in_date ≤ date < check_out_date
AND status = 'Confirmed'
Then compute
rooms_available_for_sale
= total_capacity
- rooms_out_of_order
- occupied_rooms
Example:
If on 2025-05-01
total_capacity = 120
rooms_out_of_order = 2
occupied rooms from fact_bookings = 5
Then
rooms_available_for_sale = 120 - 2 - 5 = 113
This rule must hold for every date.
The booking data and inventory table must perfectly match.
Table 3 — dim_rate_codes
Include at least:
Rack Rate
AAA
Corporate
Non-Refundable
Seasonal Promo
Columns:
rate_code_id
rate_name
description
is_commissionable
Table 4 — dim_channels
Include:
Direct Website (0% commission)
OTA (15-20% commission)
Walk-in (0%)
Corporate (10%)
Columns:
channel_id
channel_name
channel_type
commission_rate
Table 5 — dim_calendar
365 rows.
Columns:
date
day_name
is_weekend
is_holiday
season
3. Hidden Business Patterns to Inject
Embed the following realistic revenue management problems in the data.
1. The Volume Trap
Overall occupancy 75-85%, but:
Most bookings in Single rooms
Heavy dependence on OTA channels
Direct bookings remain relatively low
2. Pricing Inefficiency
Weekend prices should be only 5-10% higher than weekdays, even though demand is higher.
This simulates poor revenue management pricing strategy.
3. The Suite Gap
Suites should have very low occupancy (<20%) because:
Suites are mostly sold at Rack Rate
Rarely discounted or promoted
4. Lead Time Problem
Booking Lead Time (BLT):
OTA / low-value channels book far in advance (60-120 days)
Direct high-value guests book very late (0-7 days)
This creates inventory blockage and revenue loss.
4. Output Format
Generate one Excel file (.xlsx).
Each table must be a separate worksheet named:
fact_bookings
dim_room_inventory
dim_rate_codes
dim_channels
dim_calendar
The data should be clean, realistic, and relationally consistent so it can be used for SQL, BI dashboards, and revenue analysis.
