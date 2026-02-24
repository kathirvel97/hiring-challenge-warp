# Space Missions Log Analysis

## Problem Statement

Analyze the Interplanetary Space Agency's mission database (`space_missions.log`) to identify the security code of the longest successful Mars mission conducted between 2030-2070.

## Dataset Characteristics

**File Structure:**
- Format: Pipe-delimited (|) text file
- Size: ~9.7 MB with thousands of mission records
- Fields: Date | Mission ID | Destination | Status | Crew Size | Duration (days) | Success Rate | Security Code

**Data Challenges:**
- Contains comment lines (prefixed with #)
- System metadata lines (SYSTEM, CONFIG, CHECKSUM)
- Inconsistent whitespace around delimiters
- Multiple mission statuses (Completed, Failed, In Progress, Aborted, etc.)
- Various destinations (Mars, Moon, Jupiter, Proxima Centauri, etc.)

## Solution Approach

### Step 1: Initial Data Exploration

First, examined the file structure to understand the data format:

```bash
head -50 space_missions.log
```

**Key Findings:**
- Field 3: Destination
- Field 4: Status
- Field 6: Duration in days
- Field 8: Security Code
- Whitespace inconsistency requires trimming

### Step 2: Filter Criteria Implementation

Applied the following filters using AWK:
1. Skip comment lines and system metadata
2. Parse pipe-delimited fields
3. Trim whitespace from all fields
4. Filter for `Destination == "Mars"` AND `Status == "Completed"`
5. Track maximum duration and corresponding security code

### Step 3: Data Processing Script

```bash
awk -F'|' '
# Skip comments, system lines, and empty lines
/^#/ || /^SYSTEM/ || /^CONFIG/ || /^CHECKSUM/ || /^[[:space:]]*$/ { next }

{
    # Trim whitespace from all fields
    for (i=1; i<=NF; i++) {
        gsub(/^[[:space:]]+|[[:space:]]+$/, "", $i)
    }

    # Check if we have all 8 fields
    if (NF == 8) {
        destination = $3
        status = $4
        duration = $6 + 0  # Convert to number

        # Check for Mars missions with Completed status
        if (destination == "Mars" && status == "Completed") {
            mars_missions++
            if (duration > max_duration) {
                max_duration = duration
                max_date = $1
                max_mission = $2
                max_code = $8
            }
        }
    }
}

END {
    print "Total Mars Completed missions: " mars_missions
    print "Longest Duration: " max_duration " days"
    print "Security Code: " max_code
}
' space_missions.log
```

### Step 4: Verification and Validation

To ensure accuracy, created a verification script that extracts the top 10 longest Mars missions:

```bash
awk -F'|' '
/^#/ || /^SYSTEM/ || /^CONFIG/ || /^CHECKSUM/ || /^[[:space:]]*$/ { next }

{
    for (i=1; i<=NF; i++) {
        gsub(/^[[:space:]]+|[[:space:]]+$/, "", $i)
    }

    if (NF == 8 && $3 == "Mars" && $4 == "Completed") {
        duration = $6 + 0
        missions[duration] = $1 "|" $2 "|" $8
        all_durations[count++] = duration
    }
}

END {
    # Find top 10 by iterating through all durations
    for (i = 0; i < 10; i++) {
        max = 0
        for (j = 0; j < count; j++) {
            d = all_durations[j]
            if (d > max && !(d in printed)) {
                max = d
            }
        }
        if (max > 0) {
            printed[max] = 1
            split(missions[max], parts, "|")
            printf "%4d days | %s | %s | %s\n", max, parts[1], parts[2], parts[3]
        }
    }
}
' space_missions.log
```

## Technical Implementation Details

### AWK Script Breakdown

**1. Field Separator Setup**
```bash
-F'|'
```
Sets the pipe character as the field delimiter.

**2. Data Cleaning**
```bash
for (i=1; i<=NF; i++) {
    gsub(/^[[:space:]]+|[[:space:]]+$/, "", $i)
}
```
Removes leading and trailing whitespace from each field using regex substitution.

**3. Filtering Logic**
```bash
if (destination == "Mars" && status == "Completed")
```
Ensures only successful Mars missions are considered.

**4. Maximum Tracking**
```bash
if (duration > max_duration) {
    max_duration = duration
    max_code = $8
}
```
Maintains running maximum and associated security code.

## Why AWK works here?

**Advantages:**
- Efficient for large text file processing (handles multi-MB files easily)
- Built-in field parsing with custom delimiters
- Minimal memory footprint
- Single-pass processing capability
- No external dependencies

**Performance Characteristics:**
- Time Complexity: O(n) where n = number of records
- Space Complexity: O(1) for main script, O(k) for verification script where k = matching records
- Processing time: < 1 second on standard hardware for ~10MB file

## Validation Strategy

1. **Count verification:** Ensure total number of matching missions is reasonable
2. **Top-N analysis:** Compare top 10 longest missions to verify clear winner
3. **Spot checks:** Manually verify a sample of records for correct parsing
4. **Field integrity:** Confirm all 8 fields are present before processing


