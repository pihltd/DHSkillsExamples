---
name: crdc-load-sheet-population
description: Use an existing dataset to populate CRDC submission loading sheets
---

# CRDC Data Submission Process

## When to use this skill
When transferring data from a local system or file to the CRDC submisison loading sheets.


## Workflow

### Step 1: Obtain the submission sheets for the appropriate data commons
Submissions sheets for all supported data commons can be obtained from the Data Model Navigator on the Submssion Portal:

#### Important Links
- [Submisison Portal](https://hub.datacommons.cancer.gov/)
- [Data Model Navigator for the General Commons](https://hub.datacommons.cancer.gov/model-navigator/GC/latest)
- [Data Model Navigator for the Clinical and Translational Data Commons](https://hub.datacommons.cancer.gov/model-navigator/CTDC/latest)
- [Data Model Navigator for the Integrated Caninde Data Commons](https://hub.datacommons.cancer.gov/model-navigator/ICDC/latest)


In the upper right of the DMN is a drop down menua labelled 'Available Downloads'.  From this menu download:

- The All Properties JSON formatted data dictionary
- The All Templates Submission templates

Once the data dictionary and submission templates are obtained, the files must be unzipped to use.

### Step 2: Prepare the original data
Data that will be submitted to the CRDC should be gathered into a tabluar format with column headers.  These can be text files or spreadsheet files.  If CDEs have been used to collect data, the CDE identifiers should be included in the original data file or in a data dictionary.

### Step 3: Map the properties
The columns headers in the original data should be mapped to the column headers in the Submission Template.  This can be done by string matching and comparison or using the CDE identifiers if they are present.  

### Step 4: Move the data
For each mapped column in the original data, the column values should be moved to the mapped location in the Submission Templates.  Any unmapped columns in the original data should be reviewed to understand where that data best fits on the Submission Templates.

### Step 5: Transform the data 
For each column in the Submission template that has a controlled vocabulary in the JSON formatted data dictionary, each value in the column must match a value in the data dictionary.  If the value in the column does not match any value in the data dictionary, a close match should be selected.

### Step 6: Save the Submission Template containng transformed data.
Once the data has been transformed, each Submission Template should be saved as tab-separated text files.


 


## Common Abbreviations
| Abbreviation | Description |
|-----------------|---------------------|
| GC | General Commons |
| CTDC | Clinical and Translational Data Commons |
| ICDC | Integrated Canine Data Commons |
| PSDC | Population Science Data Commons |
| DMN | Data Model Navigator |
| CDE | Common Data Element |
| CRDC | Cancer Research Data Commons |

## Definitions
| Term | Description |
|-----------------|---------------------|
| Submission Template | Submission templates are tab-separated text files (csv) where each file represents a node in the graph model and each column is a property associated with the node.  Most, but not all, properties are associated with a CDE |
| Common Data Element | A Common Data Element (CDE) is a standardized, precisely defined question, paired with a set of allowable responses, used systematically across different sites, studies, or clinical trials to ensure consistent data collection. |

