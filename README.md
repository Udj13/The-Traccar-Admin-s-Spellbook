# 🧙‍♂️ Traccar Admin's Spellbook  
*A collection of SQL commands for common Traccar admin tasks. My cheat sheet for a PostgreSQL-backed server.*  

## 🔌 PostgreSQL Database Connection
```sql 
psql -U traccar -d traccar -h localhost
```


## 🗑️ Data Cleanup (Older Than 40 Days)

Run these commands sequentially:

```sql 
-- Delete old positions (keeping linked device positions)
DELETE FROM tc_positions 
WHERE fixTime < (NOW() - INTERVAL '40 days') 
AND id NOT IN (SELECT positionId FROM tc_devices WHERE positionId IS NOT NULL);

-- Delete old events
DELETE FROM tc_events 
WHERE eventTime < (NOW() - INTERVAL '40 days');
```

## 📊 Table Size Inspection

Via phppgadmin or command line:

```sql 
-- Count positions
SELECT COUNT(*) FROM tc_positions;

-- Count events
SELECT COUNT(*) FROM tc_events;
```

## 🐞 Troubleshooting

### 🔍 Top 50 Devices by Record Count

```sql 
SELECT deviceid, COUNT(*) AS record_count
FROM tc_positions
GROUP BY deviceid
ORDER BY record_count DESC
LIMIT 50;
```

### 📟 Identify Device & Owner

```sql 
-- Device details
SELECT * FROM tc_devices 
WHERE id = your_device_id;

-- Linked user
SELECT * FROM tc_user_device 
WHERE deviceid = your_device_id;
```

## 🧹 Bulk Cleanup for High-Volume Trackers

### 1️⃣ Find Devices with >1M Records

```sql 
SELECT deviceid FROM tc_positions 
GROUP BY deviceid 
HAVING COUNT(*) > 1000000;
```

### 2️⃣ Manual Cleanup (e.g., Device ID 123)

```sql
DELETE FROM tc_positions
WHERE fixTime < (NOW() - INTERVAL '5 days')
AND id NOT IN (SELECT positionId FROM tc_devices WHERE positionId IS NOT NULL)
AND deviceid = 123;
```

### ⚡ Automated Cleanup (Combined Query)

```sql
-- Create temp table for high-volume devices
CREATE TEMP TABLE temp_deviceids AS 
SELECT deviceid FROM tc_positions 
GROUP BY deviceid 
HAVING COUNT(*) > 1000000;

-- Delete old data from these devices
DELETE FROM tc_positions 
WHERE fixTime < (NOW() - INTERVAL '5 days')
AND id NOT IN (SELECT positionId FROM tc_devices WHERE positionId IS NOT NULL)
AND deviceid IN (SELECT deviceid FROM temp_deviceids);

-- Cleanup temp table
DROP TABLE IF EXISTS temp_deviceids;
```

