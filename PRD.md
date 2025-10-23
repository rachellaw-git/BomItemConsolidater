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
- **Data Row Criteria**: Only process rows that have data in both the `QTY` and `MFR PART #` columns. Rows missing data in either of these columns are not considered part of the dataset and should be skipped (not treated as errors).
- Ignore all other columns present in the Excel file
- Stop processing when encountering end of data
- Associate each extracted data value with its corresponding configured column header

**Deduplication Logic:**
- **Primary Key**: `MFR PART #`
- **Consolidation Rule**: Items with identical `MFR PART #` values are duplicates
- **Quantity Aggregation**: Sum the `QTY` values for all duplicate items
- **Other Columns**: Retain the first occurrence's data for `BPP SKU`, `MANUFACTURER`, and `DESCRIPTION`

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
- **Columns** (in this order):
  1. QTY (aggregated values)
  2. BPP SKU
  3. MFR PART #
  4. MANUFACTURER
  5. DESCRIPTION
- **Header Row Formatting**: Preserve the formatting from the source header row (from the first file processed), including font size, bold, alignment, etc.
- **Data Cell Formatting**: The output cells should preserve the text formatting from the source Excel files, including:
  - Font size
  - Bold/italic/underline styling
  - Text alignment (left/center/right)
  - Any other text formatting present in the original data
  - Note: When consolidating duplicate items, use the formatting from the first occurrence (applies to all columns including QTY)
- **Column Widths**: Preserve the original column widths from the source files (from the first file processed)
- **Technical Note**: SheetJS (xlsx library) supports cell formatting preservation for this functionality
- **Download**: Trigger automatic download of the generated Excel file

---

### Technical Requirements

#### Libraries
- **Excel Processing**: SheetJS (xlsx library) for reading and writing Excel files
- **Browser Compatibility**: Modern browsers (Chrome, Firefox, Safari, Edge)

---

### User Interface Requirements

#### Essential UI Elements
1. **File Selection Area**
   - Multiple file upload capability
   - Visual feedback showing selected file names and count
   
2. **Column Configuration Section**
   - Text inputs for each column name
   - Clear labels indicating purpose
   - Reset to defaults button

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

---

### Success Criteria
- Successfully consolidates data from 2+ Excel files
- Correctly identifies and sums duplicate items
- Generates a valid, downloadable Excel file
- Handles various header row positions
- Provides clear feedback for errors
- User can customize column names without code changes