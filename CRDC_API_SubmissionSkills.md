---
name: crdc-api-data-submission
description: Submit data to the CRDC Submisison Portal using the API system instead of the web interface
---

# CRDC Data Submission Process using the API system

## Prerequisites
### GraphQL
The Data Submission Portal API uses [GraphQL](https://graphql.org/) and a good understanding of how to use GraphQL is required.  Since GraphQL can be complex, a tutorial is beyond the scope of this document, however the [GraphQL Documentation](https://graphql.org/learn/) can be very useful.

### Login.gov account
Use of the Data Submission Portal in general requires that a user have an account registered with [Login.gov](https://www.login.gov/) (NIH users can use their NIH account and PIV card).  Note that a Login.gov account is distinct from an eRA Commons identity that is frequently used at NIH.  They are not the same thing.

### Approved Submission
You must recieve approval to submit data to CRDC prior to using the Data Submission Portal APIs.  If you need approval, please read and follow the [Submissions Request Instructions](https://datacommons.cancer.gov/submit).  Instructions for using the graphical data submission process are on the same page.

### An API Token
If you are an approved submitter with a Login.gov or NIH account, you can generate an API token from the graphical interface.  Log into the system, then click on your user name and select the **API Token** menu option.  This will bring up a dialog box that allows you to create an API token and copy it to your clipboard.  There are two things to note about API tokens
- **Do not include the token in your scripts.  Store the token oustide of the script, such as in an envionment variable, and access it from there.  In this skills document, it is assumed the token is stored in an environment variable name PRODAPI**
- The token is tied to your user identity and can be used on any submission that you're approved to work on.
- You can have only one token at a time.  Generating a new token will revoke the previous token.

### API Documentation
Documentation (pdf format) of all API queries is maintained on the [Submission Portal](https://hub.datacommons.cancer.gov/CRDC_Data_Submission_API_Instructions.pdf)
Additional instructions and details (pdf format) on the submission process are [also available](https://datacommons.cancer.gov/data-submission-instructions)

## Submission Process
### Query submission code
The Submission Portal API uses GraphQL and eny language capable of sending GraphQL queries to the endpoint is suitable.  In this document we will use Python, and below is an example of how to use Python to communicate with the API endpoint using the Python *requests* library.

```python
import requests
import os

def apiQuery(tier, query, variables):
    if tier == 'prod':
        url = 'https://hub.datacommons.cancer.gov/api/graphql'
        token = os.environ['PRODAPI']
    elif tier == 'stage':
        #Note that use of Stage is for example purposes only, actual submissions should use the production URL.  If you wish to run tests on Stage, please contact the helpdesk.
        url = 'https://hub-stage.datacommons.cancer.gov/api/graphql'
        token = os.environ['STAGEAPI']
    else:
        return('Please provide either "stage" or "prod" as tier values')
    headers = {"Authorization": f"Bearer {token}"}
    try:
        if variables is None:
            result = requests.post(url = url, headers = headers, json={"query": query})
        else:
            result = requests.post(url = url, headers = headers, json = {"query":query, "variables":variables})
        if result.status_code == 200:
            return result.json()
        else:
            print(f"Error: {result.status_code}")
            return result.content
    except requests.exceptions.HTTPError as e:
        return(f"HTTP Error: {e}")
```

### Pagination and Sorting Note
Most of queries that return results can paginated, which is useful when trying to retrieve large numbers of restuls.  The number of available results from a query is found in the **total** field that can be returned if requested and pagination can be done using the **first** and **offest** fields in queries.  If the **first** field is not included in a query, the system defaults to returning the first 10 results.  In addition, results can be sorted by ascending or descending values in any returned field

| Field | Description |
|------------------------|-----------------------------------------------------------------------------------|
| **first** | The number of records to be returned.  If first is set to -1, the API will return all results |
| **offset** | The number of records to be skipped when returning results. |
| **sortDirection** | Can be either *asc* or *desc* to sort using the column indicated in *orderBy* |
| **orderBy** | The field to use in sorting |


### Step 1: Understanding what studies are available
Each user will have a different set of sudies they are allowed to submit to.  The API *getMyUser* query will return a list of studies that are available.  This query requires no variables to run.

```graphql
{
  getMyUser {
    userStatus
    studies {
      _id
      controlledAccess
      createdAt
      dbGaPID
      studyName
      studyAbbreviation
    }
  }
}
```

This query returns study names, however the *_id* field is the study identifier that should be used to indicate this specific study in other queries

### Step 2:  Creating a new submission or using an existing submission
The next step in the process is to either create a new submission or to use one of your existing submissions.  It is not necessary to create a new submission every time, if you have an existing submission that you need to continue working on, simply start using that submission.

#### Creating a new submission
New submissions are created using the *createSubmission* GraphQL mutation

```graphql
mutation CreateNewSubmission(
  $studyID: String!,
  $dataCommons: String!,
  $name: String!,
  $intention:String!,
  $dataType: String!,
){
  createSubmission(
    studyID: $studyID,
    dataCommons: $dataCommons,
    name: $name,
    intention: $intention,
    dataType: $dataType
  ){
    _id
    studyID
    dbGaPID
    dataCommons
    name
    intention
    dataType
    status
  }
}
```

There are multiple required variables that have to be provided in a GraphQL compatible way:

| Field | Description |
|------------------------|-----------------------------------------------------------------------------------|
| **studyID** |  This is the assigned Study ID that can be obtained from the **_id** field in the *getMyUser* query |
| **dataCommons** | This is the CRDC Data Commons where the submissions will be deposited |
| **name** | This can be anything that allows you to identify this specific submission.  This name is not used outside of the submission process |
| **intention** | Can be *New/Update* if you are adding information to the submission or  *Delete* if you are removing information from an earlier, completed submission |
| **dataType** | Can be either *Metadata and Data Files* or *Metadata Only*.  Which one is selected depends on whether or not data files will be included in the submission |

  This query will return the **_id** field which will be the newly created submission ID. It will also return a number of other fields that can be checked to make sure the submission was created properly

  Variables should be included in the query in a proper format:

  ```python
  {"studyID":< a valid study id>, "dataCommons": <GC, CTDC, ICDC, etx>, "name": {Any string, should be meaningful to the submitter}, "intention": <New/Update or Delete>,"dataType": <Metadata and Data File or Metadata Only>}
  ```


#### Working with existing submissions
If you already have submissions in the Data Submission Portal that you've been working with, you can continue to work with them instead of creating a new submission.  To continue work on a submission, you will first have to identify the submissions using the *listSubmissions* query.
The listSubmissions query requires that **status** be provided as a parameter.  The status can be any combination of:
- All
- New
- In Progress
- Submitted
- Released
- Completed
- Archived
- Canceled
- Rejected
- Withdrawn
- Deleted


Details about what each of these states means can be found in the Submission Documentation.  For most submitters, the important states are **New**, **In Progress**, and **Submitted** as those will be the states that allow work to be done on the submission.

The *listSubmissions* query also allows the list to be pagniated using the **first** and **offset** fields and sorted in ascending or descending order with the **sortDirection** field, and to request a sorting order by field with the **orderBy** field. \

```graphql
    query ListSubmissions(
    $status:[String],
    $first: Int,
    $offset: Int,
    $orderBy: String,
    $sortDirection: String){
          listSubmissions(
              status: $status,
              first: $first,
              offset: $offset,
              orderBy: $orderBy,
              sortDirection: $sortDirection){
            total
            submissions{
              _id
              name
              submitterName
              dataCommons
              studyAbbreviation
              dbGaPID
              modelVersion
              status
              conciergeName
              createdAt
              updatedAt
              intention
            }
          }
    }
```


Whether working with a new submission or an existing one, the *_id* field is the critical piece of information and will be used to identify the submission in all future steps.  

#### Step 3: Uploading Submission templates
Once the study is created or identified, the next step is to start uploading metadata submission templates.  Note that uploading data files is done using a specialzed tool, not the API interface.

##### Collecting information about the metadata files to upload
A group of files uploaded to the Submission Portal at the same time is termed a *batch*.  Through the API, first a batch must be created, then the files uploaded using signed URLs that are generated when the batch is created.  Creating a batch uses the *createBatch* mutation.

```graphql
mutation CreateBatch(
    $submissionID: ID!, 
    $type: String, 
    $files: [String!]!) {
  createBatch(submissionID: $submissionID, type: $type, files: $files) {
    _id
    submissionID
    type
    status
    createdAt
    updatedAt
    files {
      fileName
      signedURL
    }
  }
}
```

There are several variables that need to be set as part of this query

| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **submissionID** | This is the submission ID obtained in Step 2 |
| **type**| This is the type of upload and since data files cannot be uploaded via the API, this should be set to *metadata* |
| **fileName** | This is a list of the filenames that will be uploaded.  This should be just the file name, the path should not be included |

There are two critical fields in the returned data, *_id* and the *files* group.  In this case, *_id* is the **batch** id and for each file in the submitted *filename* list, there should be the filename and a signed URL.

With the batch created and the signed URLs in hand, the metadata files can be uploaded.  Since this is a repetitive task, it is useful to have a function that can do the upload.  A Python example is shown.

```python
def awsFileUpload(file, signedurl, datadir):
    #https://docs.aws.amazon.com/AmazonS3/latest/userguide/example_s3_Scenario_PresignedUrl_section.html
    headers = {'Content-Type': 'text/tab-separated-values'}
    try:
        fullFileName = datadir+file
        with open(fullFileName, 'rb') as f:
            filetext = f.read()
        res = requests.put(signedurl, data=filetext, headers=headers)
        if res.status_code == 200:
            return res
        else:
            print(f"Error: {res.status_code}")
            return res.content
    except requests.exceptions.HTTPError as e:
        return(f"HTTP error: {e}")
```

This function creates a full path plus filename (using *file* and *datadir*), then reads teh file into a variable (*filetext*).  The Python requests module is then used to send the information using the signed URL.

While this function will send the file using the signed URL and get the response, the response contains additional information that is required to finalize the batch.  Therefore a function to parse the returned information and create an *UploadResult* object is needed.

```python
def processFilesForUpload(metadatafilelist, datadir, batch_creation_results):
    file_upload_result = []
    for entry in metadatafileslist:
        for metadatafile in batch_creation_results['data']['createBatch']['files']:
            if entry == metadatafile['fileName']:
                metares = awsFileUpload(metadatafile['fileName'], metadatafile['signedURL'], datadir)
                if metares.status_code == 200:
                    succeeded = True
                else:
                    succeeded = False
                file_upload_result.append({'fileName':entry, 'succeeded': succeeded, 'errors':[], 'skipped':False})
    return file_upload_result
```

This function will process the result returned from the *createBatch* query that was run previously and parse out the file name and the signed URL.  These are then used in conjunction with the upload function immediately above to upload the metadata submission file to the API. The function then parses the information returned from the file upload to create an *UploadResult* objec which consists of:

| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **fileName** | The name of the file as returned by the upload process |
| **succeeded** | Boolean indicating if the upload succeeded (True) or failed (False) |
| **errors** | This should be set to an empty list |
| **skipped** | This should be set to False |

An entrey for each file should be created and aggregated into a list.

The last step in uploading metadata submission files is to finalize the batch with the *updateBatch* mutation.

```graphql
mutation UpdateBatch(
        $batchID: ID!
        $files: [UploadResult]
        ){
        updateBatch(batchID:$batchID, files:$files){
            _id
            displayID
        }
    }
```

This mutation requires two variables to be provided, the batch ID obttained from the *createBatch* query, and the list of *UploadResults* created in the upload step.

It should be noted that batches are treated as a single unit.  This means that if one file in a batch fails the upload check, all files in the batch are failed and must be re-uploaded after fixing any issues.  For this reason it can be useful, but optional, to run the *listBatches* query to check that any new batches are in the list. Any batches listed as **Failed** will be accompanied by a list of the errors that were encountered.  It is useful to use pagination with this query as an active submission can have an extensive number of batches.  If grouping files into a single batch is problematic, the files can be uploaded individually, there is no minimum or maximum number of files in a batch.

```graphql
query ListBatches($submissionID: ID!) {
  listBatches(submissionID: $submissionID) {
    total
    batches {
      _id
      submissionID
      displayID
      type
      fileCount
      files {
        fileName
      }
      status
      errors
    }
  }
}
```

###  Step 4: Running the Validations
Once either metadata templates or data files have been successfully uploaded to the Submission Portal, validations can be run to check the files against CRDC sbumission standards.  Validations can be run at any time, a complete upload of all files is not required.  However, running validations on incomplete submissions, may triggere errors relating to the missing information.

Validations are run using the *validateSubmissions* mutation.

```graphql
mutation ValidateSubmission(
  $id: ID!
  $types: [String]
  $scope: String
){
  validateSubmission(_id: $id, types: $types, scope: $scope){
    success
    message
  }
}
```

This mutation takes three variables:

| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **id** | This must be a valid **submission ID**, not a batch ID |
| **types** | This list should include *metadata* to validate metadata submission files, or *data* to validate uploaded data files.  Both can be included in the list to run both validations |
| **scope** | Set to *New* to validate only files that have not been previously validated, or *All* to validate all files regardless of previous validations |

The returned *success* field does *not* indicate the success or failure of the validation, it only indicates whether or not the validation process has successfully been started.

Validations can be time consuming depending on the amount of data being reviewed and therefore there are separate queries to review the results if they are available.  If results are not available, the return message will indicate validations are still running.

To check the validation results, there are two queries that can be run:

- **aggregatedSubmissionQCResults**: This query returns a summary of the errors that have been found.  Running this first is good practice as systemic issues can produce hundreds or thousands of lines of errors, and this report summarizes those into a more easily understood format.
- **submissionQCResults**: This query returns detailed results on each of the errors found during validation.  Note that the results from this query can be numerous and are be a good use case for pagination.  

It should be noted that results are classified into two different classes.  **Errors** are fundamental mistakes in the submission that must be fixed before the overall submission can be transferred to CRDC.  This includes issues such as non-CRDC vocabulary or missing required values.  **Warnings** are less severe in that they do not block submisison to CRDC, however they can be equally crticial.  Warnings include notification of changes to existing data and should be reviewed carefully to make sure that the changes are intended and not accidental.


#### Query for aggregated validation results
```graphql
query SummaryQueryQCResults(
        $submissionID: ID!,
        $severity: String,
        $first: Int,
        $offset: Int,
        $orderBy: String,
        $sortDirection: String
    ){
        aggregatedSubmissionQCResults(
            submissionID: $submissionID,
            severity: $severity,
            first: $first,
            offset: $offset,
            orderBy: $orderBy
            sortDirection: $sortDirection
        ){
            total
            results{
                title
                severity
                count
                code
            }
        }
    }
```

This query takes two main parameters plus pagination parameters:

| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **submissionID** | The submission ID for the submission being validated |
| **severity** | The type of error to return.  Can be set to *Error*, *Warnings*, or *All* which wil return both error and warnings |
| **first** | The number of the first record to be returned.  Setting first to *-1* will return all records |
| **offset** | The number of records to return |
| **orderBy** | The field to use in sorting |
| **sortDirection** | Can be either *asc* or *desc* to sort using the column indicated in *orderBy* |

An example variable could look like this:

```python
{"submissionID":submissionid, "severity":"All", "first":-1, "offset":0, "sortDirection": "desc", "orderBy": "count"}
```

- Setting **severity** to *All* returns both *Errors* and *Warnings*
- Setting **first** ot *-1* will return all the errors without any pagnination
- Setting **sortDirection** to *desc* will cause a descending order sort
- Setting **orderBy** to *count* will use the *count* field as the source for the descending sort


#### Query for teh detailed validation results

In cases where getting the detailed results rather than a summary, the *submissionQCResults* query will return the individual errors.  As there can easily be hundreds or thousands of errors in a submission, use of pagination is stronly encouraged for this query.

```graphql
query DetailedQueryQCResults(
        $id: ID!,
        $severities: String,
        $first: Int,
        $offset: Int,
        $orderBy: String,
        $sortDirection: String
        $issueCode: String
    ){
        submissionQCResults(
            _id:$id,
            severities: $severities,
            first: $first,
            offset: $offset,
            orderBy: $orderBy,
            sortDirection: $sortDirection,
            issueCode: $issueCode
        ){
        total
        results{
            submissionID
            type
            validationType
            batchID
            displayID
            submittedID
            severity
            uploadedDate
            validatedDate
            errors{
                title
                description
            }
            warnings{
                title
                description
            }
        }
        }
    }
```


This query takes two main parameters plus pagination parameters.  This also can take an issueCode as a parameter, which allows filtering to specifc types of errors:

| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **submissionID** | The submission ID for the submission being validated |
| **severity** | The type of error to return.  Can be set to *Error*, *Warnings*, or *All* which wil return both error and warnings |
| **issueCode** | This can be obtained from the summary query above and if provided will limit the information returned to just the errors in the provided issueCode |
| **first** | The number of the first record to be returned.  Setting first to *-1* will return all records |
| **offset** | The number of records to return |
| **orderBy** | The field to use in sorting |
| **sortDirection** | Can be either *asc* or *desc* to sort using the column indicated in *orderBy* |

An example of the variables is shown below:

```python
{"id": submissionid, "severities":"All", "first": -1, "offset": 0, "orderBy":"displayID", "sortDirection":"desc", "issueCode":"M003"}
```

This is similar to the summary query, however the use of **issueCode** will restrict the data returned to only those errors having an internal classification of *M003*.  Issue codes can be returned as part of the summary query.


###  Step 5:  Submitting, Canceling, or Withdrawing
The last step of this process techincally is the submission to CRDC, however the same query is used to cancel a submission, or to withdraw a submission.  All three actions are done uwing the submissionAction mutation.  Before sending this query, it is criticl to understand the three options:

- **Submit** : Once all of the validation errors have been corrected and the validation results are either completely clean or only have warnings, the study is ready to be submitted.  Sending a submit request will hand over control of the files and data to the CRDC Data Team for final checks.  Note that once you submit a submission, no further edits are allowed.
  
- **Cancel** : If you want to abandon a submission *that has not been submitted to CRDC yet*, sending a cancellation request will lock the submission and withdraw it from the system.  **Canceling a submission is a destructive process, all data will be deleted from the submission and further work is not allowed.**
  
- **Withdraw** : Withdraw is used after a *Submit* action and returns the submission to its previous, unsubmitted (In Progress), state.  This is a **non-destructive** process, no data are deleted as a result of a withdraw.  

```graphql
mutation Submit(
    $id: ID!
    $action: String!
    $comment: String
){
    submissionAction(submissionID: $id, action: $action, comment: $comment){
        name
        submitterID
        submitterName
        dataCommons
        modelVersion
        studyAbbreviation
        dbGaPID
        status
    }
}
```

The variable that are required for this mutation are:
| Field | Description |
|-------------------|-----------------------------------------------------------------------------------------------------------------------|
| **id** | The submission ID |
| **action** | Must be one of *Submit*, *Withdraw*, or *Cancel*.  Be sure to understand each option before sending the query |
| **comment**| This is a free-text comment field in case additional information needs to be provided to the CRDC |