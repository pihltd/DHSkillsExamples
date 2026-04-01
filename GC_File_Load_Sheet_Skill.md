---
name: general-commons-file-load-sheet-population
description: Use an existing dataset to populate CRDC submission loading sheets
---

# CRDC General Commons File Load Sheet population

## When to use this skill
When a General Commons data submission sheet needs to be filed out for files that will be submitted

### Starting instructions
Write a script of the type indicated by script_type in the Setup parameters section.

### Setup parameters
script_type: Powershell
starting_file: "C:\Users\pihltd\Downloads\TDP_Data_Loading_Template_file_v11.0.3.csv"
data_file_directory: "C:\Users\pihltd\Documents\COTC021\OneDrive_1_4-17-2025"
save_file: "C:\Users\pihltd\Downloads\ Completed_file_loadsheet.tsv"

### Step 1: Read the starting file

Read the file indicated by the starting_file statement in the Setup parameters section.  All column headers should be preserved.

# Step 3: Write a script to populat the uploaded submission sheet

For each file in the data_file_directory:

- If the file name is not in the file_name column, create a new row using all existing columns and put the file name in the file_name column
- The file size in bytes for the file in the file_name column should be obtained and placed in the file_size column
- the md5sum for the file in the file_name column shold be calculated using the md5 algorithm and placed in the md5sum column
- Any other columns containing information should be retained.

### Step 4: Save the file
The populated file should be saved as a tab-delimited text file to the file name indicated by the save_file parameter in Setup parameters
