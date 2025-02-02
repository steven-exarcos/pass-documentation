# File

Files are associated with a [Submissions](Submission.md) to be used to form [Deposits](Deposit.md) into [Repositories](Repository.md)

| Attribute   | Type   | Description                                                       |
|-------------|--------|-------------------------------------------------------------------|
| id*         | String | Autogenerated identifier of object                                |
| name*       | String | File name, defaults to filesystem name                            |
| uri*        | String | Relative URI to the file servive which will return the bytestream |
| description | String | Description of file provided by [User](User.md)                   |
| fileRole    | String | Role of the file ([_see list below_](#file-role-options))         |
| mimeType    | String | Mime-type of file                                                 |


| Relationship | Type   | Target                      | Description                      |
|--------------|--------|-----------------------------|----------------------------------| 
| submission*  | To One | [Submission](Submission.md) | Submission the File is a part of |
 
*required 

## File role options

Status options for grant

| Value        | Description                                                 |
|--------------|-------------------------------------------------------------|
| manuscript   | Author accepted manuscript                                  |
| supplemental | Supplemental material for the [Publication](Publication.md) |
| figure       | An image, data plot, map, or schematic                      |
| table        | Tabular data                                                |
