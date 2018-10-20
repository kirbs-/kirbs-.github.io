---
layout: page
title: Low Cost, Encrypted S3 backups with Duplicity on Centos 7
order: 1
---

Duplicity is a flexible backup tool that compresses and encrypts backups and support multiple backend storage types. I want off site backup, so I'll use AWS S3 as my backend.

Duplicity has a lot of options, too many actually. Duply is a wrapper for duplicity that simplifies the backup/restore commands with profiles and configuration files. This enables simple one liners for cron without a lot of fuss.

## Step1: Install dependencies
Install duplicity, duply and keyring.

1. `yum install -y duplicity duply`
2. `pip install keyring`

## Step 2: Create a GPG key
1. Create a random passphrase `openssl rand -base64 20`. Save this for later.
2. Create a new GPG key pair `gpg --gen-key`. Use the passphrase from previous setup when prompted.
3. Export for safe keeping

  `gpg --armor --export duplicity`

  `gpg --armor --export-secret-key duplicity`

## Step 3: Setup S3 bucket

Create a new bucket and create a IAM policy restricting access to that bucket. You'll need to add this policy to the IAM user or role that duplicity is using.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket" ],
      "Resource": "arn:aws:s3:::my_bucket"
    },
    {
      "Effect": "Allow",
      "Action": [ "s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::my_bucket/*"
    }
  ]
}
```

## Step 4: (Optional) Create IAM user for duplicity
To limit duplicity access in AWS, create a user and attach the policy from the previous step. Save the AWS access key id and AWS secret access key for this user.

## Step 5: Store passwords in keyring
Don't hard code passwords. Use keyring instead. Keyring will prompt for the password after each command.

```bash
keyring set 'duplicity backup' AWS_ACCESS_KEY_ID
keyring set 'duplicity backup' AWS_SECRET_ACCESS_KEY
keyring set 'duplicity backup' GPG_PASSPHRASE
```


## Create duply profile
Create a new duply profile using `duply create profile_name`
`duply create s3-backup`.

## Update duply profile
SOURCE defines the folder location you want to backup. While TARGET is where you want to store the backups.

```shell
# ~/.duply/s3-backup/conf

SOURCE=/my/folder/to/backup
TARGET='s3://s3.amazonaws.com/my_bucket/_my_folder_to_backup_to'
```

Duplicity uses boto to interact with S3. Boto expects two environmental variables (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY). Add these as export statements to the config file.
```shell
# ~/.duply/s3-backup/conf

export AWS_ACCESS_KEY_ID=`keyring get 'duplicity backup' AWS_ACCESS_KEY_ID`
export AWS_SECRET_ACCESS_KEY=`keyring get 'duplicity backup' AWS_SECRET_ACCESS_KEY`
```

Set your GPG passphrase from above.
```shell
# ~/.duply/s3-backup/conf

GPG_PW=`keyring get 'duplicity backup' GPG_PASSPHRASE`
```
Lastly add any duplicity options. Below I want to use the cheaper S3 infrequent access storage class and do a full backup every 6 months.
```shell
# ~/.duply/s3-backup/conf

DUPL_PARAMS="$DUPL_PARAMS --s3-use-ia --full-if-older-than 6M "
```

## Start backup
Start duply and sitback and wait.

`duply s3-backup backup`

Once finished, duplicity prints out statistics
```
--------------[ Backup Statistics ]--------------
StartTime 1540001358.94 (Fri Oct 19 22:09:18 2018)
EndTime 1540044391.82 (Sat Oct 20 10:06:31 2018)
ElapsedTime 43032.88 (11 hours 57 minutes 12.88 seconds)
SourceFiles 28029
SourceFileSize 55294520907 (51.5 GB)
NewFiles 28029
NewFileSize 55294520907 (51.5 GB)
DeletedFiles 0
ChangedFiles 0
ChangedFileSize 0 (0 bytes)
ChangedDeltaSize 0 (0 bytes)
DeltaEntries 28029
RawDeltaSize 55292189166 (51.5 GB)
TotalDestinationSizeChange 51002426457 (47.5 GB)
Errors 1
-------------------------------------------------
```
