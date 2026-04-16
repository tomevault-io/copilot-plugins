## asikh-oms

> Always refer to the below database structure before writing backend code audit_logs


Always refer to the below database structure before writing backend code audit_logs
Column Type Comment PK Nullable Default
id uuid YES NO
table_name varchar(50) NO
record_id uuid NO
action varchar(10) NO
old_data jsonb YES
new_data jsonb YES
changed_by_id uuid YES
timestamp timestamp without time zone YES

batches
Column Type Comment PK Nullable Default
id uuid YES NO
batch_code varchar(100) NO
supervisor_id uuid NO
transport_mode varchar(50) YES
from_location uuid NO
to_location uuid YES
vehicle_number varchar(50) YES
driver_name varchar(100) YES
eta timestamp without time zone YES
departure_time timestamp without time zone YES
arrival_time timestamp without time zone YES
status varchar(50) YES
total_crates integer YES
total_weight double precision YES
notes varchar YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES
photo_url varchar YES
latitude double precision NO
longitude double precision NO

crate_reconciliations
Column Type Comment PK Nullable Default
id uuid YES NO
batch_id uuid NO
crate_id uuid NO
crate_harvest_date timestamp without time zone YES
qr_code varchar(100) YES
reconciled_by_id uuid NO
reconciled_at timestamp without time zone NO
weight double precision YES
original_weight double precision YES
weight_differential double precision YES
photo_url varchar YES
notes varchar YES
is_reconciled boolean NO

crates
Column Type Comment PK Nullable Default
id uuid YES NO
qr_code varchar(100) NO
harvest_date timestamp without time zone YES NO
gps_location jsonb YES
photo_url varchar(255) YES
supervisor_id uuid NO
weight double precision NO
notes text YES
variety_id uuid NO
batch_id uuid YES
quality_grade varchar(10) YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES
farm_id uuid YES

farms
Column Type Comment PK Nullable Default
id uuid YES NO
name varchar(100) NO
location varchar(255) YES
gps_coordinates jsonb YES
owner varchar(100) YES
contact_info jsonb YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES

packhouses
Column Type Comment PK Nullable Default
id uuid YES NO
name varchar(100) NO
location varchar(255) YES
gps_coordinates jsonb YES
manager varchar(100) YES
contact_info jsonb YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES

qr_codes
Column Type Comment PK Nullable Default
id uuid YES NO
code_value varchar(100) NO
status varchar(50) YES
entity_type varchar(50) YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES

reconciliation_logs
Column Type Comment PK Nullable Default
id uuid YES NO
batch_id uuid NO
scanned_qr varchar(100) YES
crate_id uuid YES
crate_harvest_date timestamp without time zone YES
status varchar(50) NO
timestamp timestamp without time zone YES NO
scanned_by_id uuid NO
location jsonb YES
device_info jsonb YES
notes varchar YES

users
Column Type Comment PK Nullable Default
id uuid YES NO
username varchar(100) NO
email varchar(100) NO
password varchar(255) NO
role varchar(50) NO
full_name varchar(100) YES
active boolean YES
phone_number varchar(20) YES
last_login timestamp without time zone YES
created_at timestamp without time zone YES
updated_at timestamp without time zone YES
pin varchar(255) YES
pin_set_at timestamp without time zone YES

varieties
Column Type Comment PK Nullable Default
id uuid YES NO
name varchar(100) NO
description text YES
created_at timestamp without time zone YES

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abhijeet-qsofte) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
