Here are concrete ways to improve the instructions, followed by a **cleaned‑up rewritten version** that incorporates those improvements.

---

## Revised Version (Improved Instructions)

# CRDC General Commons File Load Sheet Population

## Purpose
Use this skill to populate a CRDC General Commons data submission load sheet with file metadata for files that will be submitted.

---

## Starting Instructions
Write a script of the type specified by the `script_type` parameter in the **Setup Parameters** section.

---

## Setup Parameters
- **script_type:** PowerShell  
- **starting_file:**  
  `C:\Users\pihltd\Downloads\TDP_Data_Loading_Template_file_v11.0.3.csv`
- **data_file_directory:**  
  `C:\Users\pihltd\Documents\COTC021\OneDrive_1_4-17-2025`
- **save_file:**  
  `C:\Users\pihltd\Downloads\Completed_file_loadsheet.tsv`
  
  All input and output files are tab-delimited text

---

## Step 1: Read the Starting File
- Read the CSV file specified by `starting_file`.
- Preserve all existing column headers and data.
- Do not modify column names or ordering.

---

## Step 2: Enumerate Data Files
- Enumerate all files in `data_file_directory`.
- Process only files (ignore directories).
- Do not recurse into subdirectories unless explicitly specified.

---

## Step 3: Populate the Submission Sheet
For each file found in `data_file_directory`:
- ignore anything that is not a file.
- ignore any duplicate files.
- ignore any hidden files.

- If the file name does **not** already exist in the `file_name` column:
  - Create a new row containing all existing columns.
  - Populate the `file_name` column with the file’s name (not the full path).
- Populate or update the following columns:
  - **file_size:** File size in bytes.
  - **md5sum:** MD5 checksum calculated using the MD5 algorithm.
- If the file name already exists, update `file_size` and `md5sum` only.
- Retain all existing values in other columns without modification.

---

## Step 4: Save the Output File
- Save the populated file as a **tab-delimited text file (TSV)**.
- Use the file path specified by the `save_file` parameter.
- Include the header row.
- Use UTF-8 encoding unless otherwise specified.

