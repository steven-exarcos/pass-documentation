# Operations/Production - Data & Backups

## Data that is stored by PASS
The data in the PASS application is the submissions, deposits, grants, journals, users, policies, and various PASS data
model objects. In addition, there are configuration files that detail how different services and data loaders are run.

## Where is the Data Stored?
The data for the PASS data model is stored in Amazon Relational Database Service (RDS), with the exact implementation
being a PostgreSQL database. The configuration files are stored in S3 buckets.

## How the Data is Protected and Backed-up
AWS RDS supports automated backups. These backups occur daily and include snapshots of the entire database instance, 
enabling point-in-time recovery to any second within the retention period. Moreover, manual snapshots of the RDS 
instance can be created at any time. These snapshots are user-initiated and are retained until explicitly deleted.

S3 buckets are used to store configuration files and metadata in the PASS application. AWS S3 buckets supports 
versioning. When versioning is enabled on an S3 bucket, AWS retains multiple versions of an object. This feature allows
recovery from accidental overwrites or deletions by restoring previous versions.