# Operations/Production - Data & Backups

## Data that is stored by PASS

The data in the PASS application is the submissions, deposits, grants, journals, users, policies, and various PASS data
model objects. In addition, there are configuration files that detail how different services and data loaders are run.

## Where is the Data Stored?

The data for the PASS data model is stored in Amazon Relational Database Service (RDS), with the exact implementation
being a PostgreSQL database. The configuration files are stored in S3 buckets as well as temporary file storage for 
files uploaded through the Pass Core File Service. These files managed by the File Service are stored in an [Oxford Common File Layout (OCFL)](https://ocfl.io/) 
format that allows for an independent way of storing files in a structured, transparent and predictable fashion.

## How the Data is Protected and Backed-up

AWS RDS supports automated backups. These backups occur daily and include snapshots of the entire database instance, 
enabling point-in-time recovery to any second within the retention period. Moreover, manual snapshots of the RDS 
instance can be created at any time. These snapshots are user-initiated and are retained until explicitly deleted.

S3 buckets are used to store configuration files and metadata in the PASS application. AWS S3 buckets supports 
versioning. When versioning is enabled on an S3 bucket, AWS retains multiple versions of an object. This feature allows
recovery from accidental overwrites or deletions by restoring previous versions.

By using AWS S3 buckets it provides [data durability](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html) 
by ensuring that stored data is highly protected against loss or corruption. It achieves this by replicating data across
multiple devices in at least three distinct Availability Zones within an AWS Region. This redundancy helps S3 
preserve data even in the case of hardware failures or the loss of an entire Availability Zone. There are also routine 
procedures to verify the integrity of the data using checksums. Additionally, features that can be managed such as 
versioning, object lock, and cross-region replication further enhance data protection by safeguarding against accidental
or malicious deletion and enabling disaster recovery.