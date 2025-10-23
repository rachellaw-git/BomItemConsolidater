# Product Requirements Document (PRD)
## Excel Consolidation Tool

### Project Overview
A browser-based HTML/JavaScript application that consolidates data from multiple Excel files by identifying duplicate items and summing their quantities.

---

### Core Functionality

#### 1. File Input
- **Requirement**: UI component to select multiple Excel files from the user's file system
- **File Types**: `.xlsx`, `.xls`
- **User Experience**: Drag-and-drop support preferred, with traditional file picker as alternative
- **Quantity Multiplier**: After files are selected, allow user to input a positive integer (multiplier) for each file
  - Default value: 1
  - **Must be a positive integer only** (no decimals allowed)
  - Minimum value: 1
  - This multiplier will be applied to the `QTY` value of every item in that specific file before consolidation
  - Example: If a file has an item with QTY=5 and multiplier=3, the effective quantity for that item becomes 15

#### 2. Data Processing

**Column Detection:**
- Scan the first sheet of each Excel file to locate the header row
- **Target Columns** (configurable by user, in any order):
  - `QTY` - Quantity (numeric)
  - `BPP SKU` - SKU identifier
  - `MFR PART #` - Manufacturer Part Number (primary key for deduplication)
  - `MANUFACTURER` - Manufacturer name
  - `DESCRIPTION` - Item description
- **Important**: Only extract data from columns that match the user-configured column names. Ignore all other columns in the Excel files.

- **Header Row Location**: Headers are NOT guaranteed to be in row 1; scan until found
- **Duplicate Headers**: Header row may appear multiple times in the sheet; ignore subsequent occurrences after the first valid header row is found

**Data Extraction:**
- Once header row is identified, extract data **only from the columns specified in user configuration**
- For each row beneath the header, capture values only from the matched columns
- **Quantity Multiplier Application**: Multiply the `QTY` value by the file's multiplier (as specified by the user) before any consolidation
- **Data Row Criteria**: Only process rows that meet all of the following conditions:
  - Have data in both the `QTY` and `MFR PART #` columns
  - The value in the `QTY` column is a valid number
  - Rows that fail any of these criteria are not considered part of the dataset and should be skipped (not treated as errors)
- Ignore all other columns present in the Excel file
- Stop processing when encountering end of data
- Associate each extracted data value with its corresponding configured column header

**Deduplication Logic:**
- **Primary Key**: `MFR PART #`
- **Consolidation Rule**: Items with identical `MFR PART #` values are duplicates
- **Quantity Aggregation**: Sum the `QTY` values (after multiplier has been applied) for all duplicate items
- **Other Columns**: Retain the first occurrence's data for `BPP SKU`, `MANUFACTURER`, and `DESCRIPTION`

**Sorting:**
- After consolidation and deduplication is complete, sort the consolidated data:
  - **Primary Sort**: `MANUFACTURER` (alphabetical, A-Z)
  - **Secondary Sort**: `MFR PART #` (alphabetical, A-Z)
- Sorting should be case-insensitive

#### 3. User Configuration
- **Editable Column Names**: Provide UI controls to modify the expected column header names
- **Purpose**: Allow users to adapt the tool to Excel files with slightly different naming conventions
- **Note**: Only columns defined in this configuration will be extracted from the Excel files. Any columns not listed here will be ignored during processing, even if they contain data.
- **Default Values**: 
  - QTY
  - BPP SKU
  - MFR PART #
  - MANUFACTURER
  - DESCRIPTION

#### 4. Output Generation
- **Format**: Excel file (`.xlsx`)
- **Structure**: Single sheet with consolidated data
- **Title Row**: First row (row 1) should contain the text "BILL OF MATERIAL"
  - Font Size: 16
  - Bold: True
  - Horizontal Alignment: Center
- **Header Row**: Second row (row 2) contains column headers
  - Font Size: 11
  - Bold: True
- **Columns** (in this order):
  1. QTY (aggregated values)
  2. BPP SKU
  3. MFR PART #
  4. MANUFACTURER
  5. DESCRIPTION
- **Data Rows**: All rows below the header row
  - Font Size: 11
  - Bold: False
- **Column Widths**:
  - QTY, BPP SKU, MFR PART #, and MANUFACTURER: Auto-fit to content
  - DESCRIPTION: Do not auto-fit (use default or fixed width)
- **Output Filename**: Format as "BOM: {filename1} (xN), {filename2} (xM)..." where:
  - Filenames are the names of the input files (without extensions) separated by commas
  - If a file has a multiplier greater than 1, append " (xN)" where N is the multiplier value
  - If a file has a multiplier of 1, do not append any multiplier notation
  - Examples:
    - With multipliers of 1: "BOM: file1, file2, file3.xlsx"
    - With multipliers: "BOM: file1 (x3), file2, file3 (x2).xlsx"
  - **Filename Length Limit**: Maximum 250 characters
  - **Truncation Logic**: If the full filename would exceed 250 characters:
    - Include as many complete filenames (with their multipliers if applicable) as possible (separated by commas)
    - Append " and X more" where X is the count of remaining files not shown
    - Example: "BOM: file1 (x2), file2 and 3 more.xlsx"
    - Ensure the final filename (including ".xlsx" extension) does not exceed 250 characters
- **Download**: Trigger automatic download of the generated Excel file

---

### User Interface Requirements

#### Essential UI Elements
1. **File Selection Area**
   - Multiple file upload capability
   - Visual feedback showing selected file names and count
   - Input field for quantity multiplier next to each selected file (default: 1)
   - Validation to ensure multipliers are positive integers only (no decimals)
   
2. **Column Configuration Section**
   - Collapsible "Advanced Settings" section (collapsed by default)
   - When expanded, displays:
     - Text inputs for each column name
     - Clear labels indicating purpose
     - Reset to defaults button
   - Most users will not need to access this section

3. **Process Button**
   - Clearly labeled (e.g., "Consolidate Files")
   - Disabled until files are selected
   - Loading indicator during processing

4. **Results Display**
   - Success message with download link
   - Preview of consolidated data (optional but recommended)
   - Item count summary (e.g., "Consolidated 247 items from 5 files")

5. **Error Handling**
   - Clear error messages for:
     - Invalid file types
     - Missing required columns
     - Empty files
     - Processing failures

---

### Edge Cases to Handle

1. **Missing Columns**: If a required column is not found in a file, display error and skip that file
2. **Empty Files**: Skip files with no data rows
3. **Invalid Data**: Handle non-numeric values in `QTY` column (treat as 0 or skip row)
4. **Case Sensitivity**: Column header matching should be case-insensitive
5. **Whitespace**: Trim whitespace from column headers and `MFR PART #` values
6. **Long Filenames**: Truncate output filename if combined input filenames exceed 250 characters
7. **Invalid Multipliers**: Reset to 1 if user enters non-integer or negative values

---

### Success Criteria
- Successfully consolidates data from 2+ Excel files
- Correctly identifies and sums duplicate items
- Applies integer multipliers correctly to quantities
- Sorts consolidated data by MANUFACTURER, then by MFR PART #
- Generates a valid, downloadable Excel file with appropriate filename (max 250 chars)
- Handles various header row positions
- Provides clear feedback for errors
- User can customize column names without code changes