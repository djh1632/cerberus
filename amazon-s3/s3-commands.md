# File Commands for Amazon S3

New file commands make it easy to manage your Amazon S3 objects. Using familiar syntax, you can view the contents of your S3 buckets in a directory-based listing.

```
$ aws s3 ls s3://mybucket
```

        LastWriteTime            Length Name

        ------------             ------ ----

                                PRE myfolder/

2013-09-03 10:00:00           1234 myfile.txt


You can perform recursive uploads and downloads of multiple files in a single folder-level command. The AWS CLI will run these transfers in parallel for increased performance.

```
$ aws s3 cp myfolder s3://mybucket/myfolder --recursive
```

upload: myfolder/file1.txt to s3://mybucket/myfolder/file1.txt

upload: myfolder/subfolder/file1.txt to s3://mybucket/myfolder/subfolder/file1.txt

A sync command makes it easy to synchronize the contents of a local folder with a copy in an S3 bucket.

```
$ aws s3 sync myfolder s3://mybucket/myfolder --exclude *.tmp
```

upload: myfolder/newfile.txt to s3://mybucket/myfolder/newfile.txt

Examples

Copying a local file to S3

The following cp command copies a single file to a specified bucket and key:

```
aws s3 cp test.txt s3://mybucket/test2.txt
```

Output:

upload: test.txt to s3://mybucket/test2.txt

Copying a file from S3 to S3

The following cp command copies a single s3 object to a specified bucket and key:

aws s3 cp s3://mybucket/test.txt s3://mybucket/test2.txt

Output:

copy: s3://mybucket/test.txt to s3://mybucket/test2.txt

Copying an S3 object to a local file

The following cp command copies a single object to a specified file locally:

aws s3 cp s3://mybucket/test.txt test2.txt

Output:

download: s3://mybucket/test.txt to test2.txt

Copying an S3 object from one bucket to another

The following cp command copies a single object to a specified bucket while retaining its original name:

aws s3 cp s3://mybucket/test.txt s3://mybucket2/

Output:

copy: s3://mybucket/test.txt to s3://mybucket2/test.txt

Recursively copying S3 objects to a local directory

When passed with the parameter --recursive, the following cp command recursively copies all objects under a specified prefix and bucket to a specified directory. In this example, the bucket mybucket has the objects test1.txt and test2.txt:

aws s3 cp s3://mybucket . --recursive

Output:

download: s3://mybucket/test1.txt to test1.txt
download: s3://mybucket/test2.txt to test2.txt

Recursively copying local files to S3

When passed with the parameter --recursive, the following cp command recursively copies all files under a specified directory to a specified bucket and prefix while excluding some files by using an --exclude parameter. In this example, the directory myDir has the files test1.txt and test2.jpg:

aws s3 cp myDir s3://mybucket/ --recursive --exclude "*.jpg"

Output:

upload: myDir/test1.txt to s3://mybucket/test1.txt

Recursively copying S3 objects to another bucket

When passed with the parameter --recursive, the following cp command recursively copies all objects under a specified bucket to another bucket while excluding some objects by using an --exclude parameter. In this example, the bucket mybucket has the objects test1.txt and another/test1.txt:

aws s3 cp s3://mybucket/ s3://mybucket2/ --recursive --exclude "another/*"

Output:

copy: s3://mybucket/test1.txt to s3://mybucket2/test1.txt

You can combine --exclude and --include options to copy only objects that match a pattern, excluding all others:

aws s3 cp s3://mybucket/logs/ s3://mybucket2/logs/ --recursive --exclude "*" --include "*.log"

Output:

copy: s3://mybucket/test/test.log to s3://mybucket2/test/test.log
copy: s3://mybucket/test3.log to s3://mybucket2/test3.log

Setting the Access Control List (ACL) while copying an S3 object

The following cp command copies a single object to a specified bucket and key while setting the ACL to public-read-write:

aws s3 cp s3://mybucket/test.txt s3://mybucket/test2.txt --acl public-read-write

Output:

copy: s3://mybucket/test.txt to s3://mybucket/test2.txt

Note that if you're using the --acl option, ensure that any associated IAM policies include the "s3:PutObjectAcl" action:

aws iam get-user-policy --user-name myuser --policy-name mypolicy

Output:

{
    "UserName": "myuser",
    "PolicyName": "mypolicy",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:PutObject",
                    "s3:PutObjectAcl"
                ],
                "Resource": [
                    "arn:aws:s3:::mybucket/*"
                ],
                "Effect": "Allow",
                "Sid": "Stmt1234567891234"
            }
        ]
    }
}

Granting permissions for an S3 object

The following cp command illustrates the use of the --grants option to grant read access to all users and full control to a specific user identified by their email address:

aws s3 cp file.txt s3://mybucket/ --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers full=emailaddress=user@example.com

Output:

upload: file.txt to s3://mybucket/file.txt

Uploading a local file stream to S3

WARNING:: PowerShell may alter the encoding of or add a CRLF to piped input.

The following cp command uploads a local file stream from standard input to a specified bucket and key:

aws s3 cp - s3://mybucket/stream.txt

Downloading an S3 object as a local file stream

WARNING:: PowerShell may alter the encoding of or add a CRLF to piped or redirected output.

The following cp command downloads an S3 object locally as a stream to standard output. Downloading as a stream is not currently compatible with the --recursive parameter:

aws s3 cp s3://mybucket/stream.txt -

