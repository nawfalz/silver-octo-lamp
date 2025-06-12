# Fleet Management Anomaly Detection Guide

# Introduction
 This document serves as a form of identifying key components to the types of data that the Agent will be handling. These includes Key Columns to track which depends on which kind of report the Agent is reviewing, Normal Behavior and Abnormal Behavior Patterns. Examples shown here are **not** to be referenced, and serves more as a form of understanding.
# Key Columns to Track
## Vehicle History Reports 
Vehicle History Reports from MTData have a total of 76 columns. The headers for these columns are at `Row 12`, you may ignore the first 11 rows (except `Row 8` and `Row 9` -> These rows gives us the *Fleet Name* and *Vehicle Number* respectively). The following are the key columns to note, you may ignore everything else apart from these columns:
1. Device Timestamp: str *deviceTimestamp* `Column B`
2. Reason: str *reportStatus* `Column D`
3. Driver: str *driverName* `Column F`
4. Latitude: str *gpsLatitude* `Column I`
5. Longitude: str *gpsLongitude* `Column J`
6. Speed: int *speedValue* `Column M`
7. Spd Acc: int *spdAcc* `Column X`
8. Suburb: str *placeName* `Column AA`
## MDP Data 
Reports from MDP have a total of 116 columns. The following are the key columns to note, you may ignore everything else apart from these columns:
1. Report: str *reportStatus* `Column A`
2. ReportId: int *reportNumber* `Column B`
3. FleetName: str *fleetName* `Column F`
4. VehicleName: int *vehicleNumber* `Column N`
5. DriverName: str *driverName* `Column Q`
6. GpsData_DeviceTime: str *deviceTimestamp* `Column AI`
7. GpsData_Latitude: str *gpsLatitude* `Column AK`
8. GpsData_Longitude: str *gpsLongitude* `Column AL`
9. GpsData_SpeedAccumulator: int *spdAcc* `Column AM`
10. GpsData_Speed: int *speedValue* `Column AO`
11. GpsData_PlaceName: str *placeName* `Column AQ`
## Normal Behavior Examples 
These examples were created from a python script that does data extraction of the data from Vehicle History Reports. The script looks for improper Ignition On/Off cycles and splits each cycle into the following 11 columns:.
1. Cycle Number:  int *cycleNumber*  `Column A`
2. Cycle Status: str *cycleStatus* `Column B`
3. Device (Timestamp): str *deviceTimestamp* `Column C`
4. Reason: str *reasonStatus* `Column D`
5. Driver: str *driverName*  `Column E`
6. Latitude: str *gpsLatitude*  `Column F`
7. Longitude: str *gpsLongitude*  `Column G`
8. Speed: int *speedValue*  `Column H`
9. Spd Acc: int *spdAcc*  `Column I`
10. Suburb: str *placeName*  `Column J`
11. Key Moment: str *keyMoment*  `Column K`
## Abnormal Behavior Examples 
These examples were created from a python script that does data extraction of the data from Vehicle History Reports. The script looks for improper Ignition On/Off cycles and splits each cycle into the following 11 columns:.
1. Cycle Number:  int *cycleNumber*  `Column A`
2. Cycle Status: str *cycleStatus* `Column B`
3. Device (Timestamp): str *deviceTimestamp* `Column C`
4. Reason: str *reasonStatus* `Column D`
5. Driver: str *driverName*  `Column E`
6. Latitude: str *gpsLatitude*  `Column F`
7. Longitude: str *gpsLongitude*  `Column G`
8. Speed: int *speedValue*  `Column H`
9. Spd Acc: int *spdAcc*  `Column I`
10. Suburb: str *placeName*  `Column J`
11. Key Moment: str *keyMoment*  `Column K`

# Behavior Patterns
## Normal Behaviors
1. Data from sensors to be logged almost every minute
2. 1 *Ignition On* instance to 1 *Ignition Off* instance
3. As vehicle is **moving off**:
	- *Speed* value in `Speed` column gradually goes up
	- *Speed Accumulator* value in `Spd Acc` column cumulative total goes up
	- GPS *Latitude* value in `Latitude` column will change
	- GPS *Longitude* value in `Longitude` column will change
4. As vehicle is **moving**:
	- *Speed* value in `Speed` column is not 0
	- *Speed Accumulator* value in `Spd Acc` column cumulative total goes up
	- GPS *Latitude* value in `Latitude` column will change
	- GPS *Longitude* value in `Longitude` column will change
5. As vehicle is **slowing down**:
	- *Speed* value in `Speed` column gradually goes down
	- *Speed Accumulator* value in `Spd Acc` column cumulative total goes up
	- GPS *Latitude* value in `Latitude` column will change
	- GPS *Longitude* value in `Longitude` column will change
6. As vehicle is **parked** or **at rest**:
	- *Speed* value in `Speed` column is 0
	- *Speed Accumulator* value in `Spd Acc` column cumulative total is stagnant
	- GPS *Latitude* value in `Latitude` column will not change
	- GPS *Longitude* value in `Longitude` column will not change

| Description      | Speed               | Spd Acc                      | Latitude    | Longitude   |
| ---------------- | ------------------- | ---------------------------- | ----------- | ----------- |
| Moving off       | Gradually goes up   | Cumulative total goes up     | Will change | Will change |
| Moving           | Not 0               | Cumulative total goes up     | Will change | Will change |
| Slowing Down     | Gradually goes down | Cumulative total goes up     | Will change | Will change |
| Parked / At rest | Is 0                | Cumulative total is stagnant | No change   | No change   |

## Abnormal Behaviors

### 1. Data from sensors not logged for more than a minute

Example:

| Device (Timestamp)  | Reason                                | Row No. |
| ------------------- | ------------------------------------- | ------- |
| 2025-05-11 07:52:06 | Ignition On                           | 1       |
| 2025-05-11 07:52:47 | Status Report                         | 2       |
| 2025-05-11 07:52:55 | MDVR Status                           | 3       |
| 2025-05-11 07:54:21 | GForce Calibration - Start            | 4       |
| 2025-05-11 07:54:45 | Diagnostic                            | 5       |
| 2025-05-11 07:56:20 | GForce Calibration - Stage 1 Complete | 6       |
| 2025-05-11 08:00:26 | Unit Power Up                         | 7       |
| 2025-05-11 08:00:38 | Diagnostic Level                      | 8       |
| 2025-05-11 08:39:05 | Dep. Workshop                         | 9       |
| 2025-05-11 08:39:07 | GForce Calibration - Start            | 10      |
| 2025-05-11 08:39:40 | Arr. Workshop                         | 11      |
| 2025-05-11 08:41:08 | GForce Calibration - Stage 1 Complete | 12      |
In the example above, you can see gaps in the data. The Timestamp will need to be split into Year, Month, Day and Time. For this example, let's focus on the Time component. Data should be logged in at every minute however we can see small lapses such as between Row No. 3 and 4 (07:52:55 jumps to 07:54:21) and Row No. 5 and 6 (07:54:45 jumps to 07:56:20). Larger lapses can be seen in Row No. 8 and 9 (08:00:38 jumps to 08:39:05). 

### 2. Multiple 'Ignition On' instances before reaching next instance 'Ignition Off'

Example:

| Device (Timestamp)  | Reason           | Row No. |
| ------------------- | ---------------- | ------- |
| 2025-05-11 07:50:06 | Ignition On      | 1       |
| 2025-05-11 07:51:47 | Unit Power Up    | 2       |
| 2025-05-11 07:52:55 | Diagnostic Level | 3       |
| 2025-05-11 07:53:21 | Ignition On      | 4       |
| 2025-05-11 07:54:45 | Status Report    | 5       |
| 2025-05-11 07:55:20 | Diagnostic       | 6       |
| 2025-05-11 08:00:26 | Status Report    | 7       |
| 2025-05-11 08:00:38 | Ignition On      | 8       |
| 2025-05-11 08:39:05 | Ignition On      | 9       |
| 2025-05-11 08:39:07 | Diagnostic       | 10      |
| 2025-05-11 08:39:40 | Status Report    | 11      |
| 2025-05-11 08:41:08 | Ignition Off     | 12      |
In the example above, you can see there are multiple 'Ignition On' instances before reaching the next instance of 'Ignition Off' in the **Reason** Column. This should not be the case and is an obvious anomaly if found in the report. The report should have a 1-to-1 pairing of 'Ignition On' to 'Ignition Off'.
### 3. Multiple 'Ignition Off' instances before reaching next instance 'Ignition On'

Example:

| Device (Timestamp)  | Reason           | Row No. |
| ------------------- | ---------------- | ------- |
| 2025-05-11 07:50:06 | Ignition On      | 1       |
| 2025-05-11 07:51:47 | Unit Power Up    | 2       |
| 2025-05-11 07:52:55 | Diagnostic Level | 3       |
| 2025-05-11 07:53:21 | Ignition Off     | 4       |
| 2025-05-11 07:54:45 | Status Report    | 5       |
| 2025-05-11 07:55:20 | Diagnostic       | 6       |
| 2025-05-11 08:00:26 | Status Report    | 7       |
| 2025-05-11 08:00:38 | Ignition Off     | 8       |
| 2025-05-11 08:39:05 | Ignition Off     | 9       |
| 2025-05-11 08:39:07 | Diagnostic       | 10      |
| 2025-05-11 08:39:40 | Status Report    | 11      |
| 2025-05-11 08:41:08 | Ignition On      | 12      |
In the example above, you can see there are multiple 'Ignition Off' instances before reaching the next instance of 'Ignition On' in the **Reason** Column. This should not be the case and is an obvious anomaly if found in the report. The report should have a 1-to-1 pairing of 'Ignition On' to 'Ignition Off'.

# Data Extraction Python Script

This script handles the data extraction portion for Normal and Abnormal Behavior Examples.

```python
import pandas as pd
import openpyxl
import os
import re
from datetime import datetime
from openpyxl import Workbook
from openpyxl.styles import numbers

def sanitize_name(text):
    """Remove special characters for safe filenames"""
    return re.sub(r'[\\/*?:"<>|]+', '', str(text)).strip()

def detect_key_moments(df):
    """Identifies key vehicle moments based on Speed/Spd Acc patterns"""
    moments = ['']  # First row has no previous data
    for i in range(1, len(df)):
        speed_diff = df.iloc[i]['Speed'] - df.iloc[i-1]['Speed']
        spd_acc_diff = df.iloc[i]['Spd Acc'] - df.iloc[i-1]['Spd Acc']
        if speed_diff > 0 and spd_acc_diff > 0:
            moments.append('Moving off')
        elif speed_diff < 0 and spd_acc_diff > 0:
            moments.append('Slowing down')
        elif df.iloc[i]['Speed'] == 0 and spd_acc_diff == 0:
            moments.append('At rest')
        else:
            moments.append('')
    return moments
# =============================================================================
# Modified Key Moment Detection with Traffic Stop Filtering
# =============================================================================
def filter_rest_periods(key_moments):
    """Filters out 'At rest' between movement events"""
    filtered = []
    in_brief_rest = False
    for i, moment in enumerate(key_moments):
        if moment == 'At rest':
            # Check surrounding moments
            prev = key_moments[i-1] if i > 0 else ''
            next_m = key_moments[i+1] if i < len(key_moments)-1 else ''
            # Pattern: Moving off → At rest → Slowing down
            if ('Moving off' in prev) and ('Slowing down' in next_m):
                filtered.append('Brief stop')
            # Pattern: Between two movement events
            elif any(m in prev for m in ['Moving off', 'Slowing down']) and \
                any(m in next_m for m in ['Moving off', 'Slowing down']):
                filtered.append('Brief stop')
            else:
                filtered.append(moment)
        else:
            filtered.append(moment)
    return filtered
    
# Load original report
report_filename = 'Report_51995_Vehicle History Report (1).xlsx'

# Define columns to keep
target_columns = ["Device", "Reason", "Driver", "Latitude", "Longitude", "Speed", "Spd Acc", "Suburb"]
# =============================================================================
# 1. Extract Fleet & Vehicle Info
# =============================================================================
try:
    wb = openpyxl.load_workbook(report_filename, data_only=True)
    sheet = wb.active
    # Extract from merged cells (C8 and C9 contain the values)
    fleet_name = sheet['C8'].value or 'UnknownFleet'
    vehicle_number = sheet['C9'].value or 'UnknownVehicle'
    wb.close()
except Exception as e:
    print(f"Error reading metadata: {e}")
    fleet_name = 'UnknownFleet'
    vehicle_number = 'UnknownVehicle'
# =============================================================================
# 2. Create Output Folder
# =============================================================================
safe_fleet = sanitize_name(fleet_name)
safe_vehicle = sanitize_name(vehicle_number)
timestamp = datetime.now().strftime("%Y%m%d_%H%M")
folder_name = f"{safe_fleet}_{safe_vehicle}_{timestamp}"
os.makedirs(folder_name, exist_ok=True)
# =============================================================================
# 3. Load Data with Automatic Skipping
# =============================================================================
try:
    # Skip first 11 rows (0-10) and use row 11 as header
    data = pd.read_excel(
        report_filename,
        skiprows=11,
        header=0
    )
except Exception as e:
    raise ValueError(f"Data loading failed: {e}")
# =============================================================================
# 4. Add Metadata to All Exports
# =============================================================================
metadata = {
    'Fleet': fleet_name,
    'Vehicle': vehicle_number,
    'Report Date': datetime.now().strftime("%Y-%m-%d %H:%M:%S")
}
# =============================================================================
# 5. Enhanced Cycle Processing with Off Pairing
# =============================================================================
cycles = []
current_cycle = []
last_off_idx = -1  # Track position of last Off in current cycle
in_cycle = False

# Initialize cycle tracking
for idx, row in data.iterrows():
    row_data = row[target_columns].to_dict()
    if row['Reason'] == 'Ignition On':
        if in_cycle:
            # Close previous cycle using last available Off
            if last_off_idx != -1:
                cycles.append({
                    'status': 'complete',
                    'data': current_cycle[:last_off_idx+1]  # Slice up to last Off
                })
            else:
                cycles.append({'status': 'incomplete', 'data': current_cycle})
            # Reset tracking
            current_cycle = []
            last_off_idx = -1
            
        # Start new cycle
        current_cycle = [row_data]
        in_cycle = True
    elif row['Reason'] == 'Ignition Off' and in_cycle:
        current_cycle.append(row_data)
        last_off_idx = len(current_cycle) - 1  # Update last Off position
    elif in_cycle:
        current_cycle.append(row_data)

# Handle final cycle
if in_cycle:
    if last_off_idx != -1:
        cycles.append({
            'status': 'complete',
            'data': current_cycle[:last_off_idx+1]
        })
    else:
        cycles.append({'status': 'incomplete', 'data': current_cycle})
# =============================================================================
# 6. Generate Key Moments and Save Cycles (.csv)
# =============================================================================
# Convert to DataFrame and add key moments
for i, cycle in enumerate(cycles, 1):
    cycle_df = pd.DataFrame(cycle['data'], columns=target_columns)
    # cycle_df['Key Moment'] = detect_key_moments(cycle_df)
    cycle_df['Key Moment'] = filter_rest_periods(detect_key_moments(cycle_df))
    cycle_df.insert(0, 'Cycle Number', i)
    cycle_df.insert(1, 'Cycle Status', cycle['status'])
    # Save to output folder
    # out_csv = os.path.join(folder_name, f'{safe_vehicle}_ignition_cycle_{i}_{cycle["status"]}.csv')
    if cycle['status'] == 'incomplete':
        out_csv = os.path.join(folder_name, f'{safe_vehicle}_abnormal_example_{i}.csv')
    else:
        out_csv = os.path.join(folder_name, f'{safe_vehicle}_normal_behavior_example_{i}.csv')
    cycle_df.to_csv(out_csv, index=False)
# =============================================================================
# 7. Generate Summary Report
# =============================================================================
## Create summary report
summary = []
for i, cycle in enumerate(cycles, 1):
    try:
        start_time = cycle['data'][0].get('Device')
        end_time = cycle['data'][-1].get('Device')
        summary.append({
            'Cycle': i,
            'Status': cycle['status'],
            'Start Time': start_time or 'N/A',
            'End Time': end_time or 'N/A',
            'Rows': len(cycle['data'])
        })
    except KeyError:
        print(f"Missing timestamp data in cycle {i}")
  
# Add totals to summary
total_cycles = len(cycles)
complete_cycles = len([c for c in cycles if c['status'] == 'complete'])
incomplete_cycles = total_cycles - complete_cycles
ignition_on_count = data['Reason'].value_counts().get('Ignition On', 0)
ignition_off_count = data['Reason'].value_counts().get('Ignition Off', 0)

# Add columns to summary report
summary_df = pd.DataFrame(summary)
summary_df['Ignition On Count'] = ignition_on_count
summary_df['Ignition Off Count'] = ignition_off_count
summary_df['Total Cycles'] = total_cycles
summary_df['Complete Cycles'] = complete_cycles
summary_df['Incomplete Cycles'] = incomplete_cycles
summary_df['Fleet'] = fleet_name
summary_df['Vehicle'] = vehicle_number
summary_df.to_csv(os.path.join(folder_name, 'summary.csv'), index=False)
# summary_df.to_excel(os.path.join(folder_name, 'summary.xlsx'), index=False)
print(f"All outputs saved to: {os.path.abspath(folder_name)}")
```

# How to Use This Document

This section outlines how to integrate and operationalize the anomaly definitions and examples in your Agent or BI tool.

1. **Ingestion**    
    - Load this Markdown file into your Agent’s knowledge ingestion pipeline or your BI tool’s documentation loader.        
    - Parse the YAML front-matter for metadata (version, date, author).        
    - Extract sections and tables to map anomaly definitions to internal data models.        
2. **Expected Output Schema**  
    When parsed, each anomaly definition should conform to the following JSON schema:    
    ```json
    {
      "anomaly_id": "string",
      "anomaly_name": "string",
      "description": "string",
      "detection_rule": "string (Boolean expression or pseudocode)",
      "fields": {
        "device_timestamp": "ISO8601 timestamp",
        "reason_code": "string",
        "speed_kmh": "number",
        "...": "..."
      }
    }
    ```    
3. **Example Data**  
    Sample CSVs demonstrating normal and abnormal behaviors are stored in the `/examples/` directory next to this document:    
    - `normal_behavior_example_2.csv`  
        Snippet of typical, anomaly-free telemetry.        
    - `abnormal_example_3.csv`  
        Snippet containing labeled anomalies with `anomaly_type` values.            
    **Access these files** via knowledge base.    
4. **Agent Integration**    
    - Use the `anomaly_id` labels in the `anomaly_type` column of your CSVs to train and validate detection models.        
    - At runtime, the Agent cross-references live telemetry against the `detection_rule` expressions defined here to flag real-time events.        
5. **Updating the Document**    
    - When adding new anomaly types, append entries under **Anomaly Categories** with a unique `anomaly_id`.        
    - Upload corresponding CSV snippets to the `/examples/` folder.        
    - Increment the `version` and `last-updated` fields in the YAML front-matter.
