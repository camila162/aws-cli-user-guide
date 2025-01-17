--------

--------

# Loading AWS CLI parameters from a file<a name="cli-usage-parameters-file"></a>

Some parameters expect file names as arguments, from which the AWS CLI loads the data\. Other parameters enable you to specify the parameter value as either text typed on the command line or read from a file\. Whether a file is required or optional, you must encode the file correctly so that the AWS CLI can understand it\. The file's encoding must match the reading system's default locale\. You can determine this by using the Python `locale.getpreferredencoding()` method\.

**Note**  
By default, Windows PowerShell outputs text as UTF\-16, which conflicts with the UTF\-8 encoding used by many Linux systems\. We recommend that you use `-Encoding ascii` with your PowerShell `Out-File` commands to ensure the AWS CLI can read the resulting file\. 

**Topics**
+ [How to load parameters from a file](#cli-usage-parameters-file-how)
+ [Binary files](#cli-usage-parameters-file-binary)

## How to load parameters from a file<a name="cli-usage-parameters-file-how"></a>

Sometimes it's convenient to load a parameter value from a file instead of trying to type it all as a command line parameter value, such as when the parameter is a complex JSON string\. To specify a file that contains the value, specify a file URL in the following format\.

```
file://complete/path/to/file
```
+ The first two slash '/' characters are part of the specification\. If the required path begins with a '/', the result is three slash characters: `file:///folder/file`\.
+ The URL provides the path to the file that contains the actual parameter content\. 
+ When using files with spaces or special characters, following the [quoting and escaping rules](cli-usage-parameters-quoting-strings.md) for your terminal\. 

The file paths in the following examples are interpreted to be relative to the current working directory\.

------
#### [ Linux or macOS ]

```
// Read from a file in the current directory
$ aws ec2 describe-instances --filters file://filter.json

// Read from a file in /tmp
$ aws ec2 describe-instances --filters file:///tmp/filter.json

// Read from a file with a filename with whitespaces
$ aws ec2 describe-instances --filters 'file://filter content.json'
```

------
#### [ Windows command prompt ]

```
// Read from a file in C:\temp
C:\> aws ec2 describe-instances --filters file://C:\temp\filter.json

// Read from a file with a filename with whitespaces
C:\> aws ec2 describe-instances --filters "file://C:\temp\filter content.json"
```

------

The `file://` prefix option supports Unix\-style expansions, including "`~/`", "`./`", and "`../`"\. On Windows, the "`~/`" expression expands to your user directory, stored in the `%USERPROFILE%` environment variable\. For example, on Windows 10 you would typically have a user directory under `C:\Users\UserName\`\.

You must still escape JSON documents that are embedded as the value of another JSON document\.

```
$ aws sqs create-queue --queue-name my-queue --attributes file://attributes.json
```

**attributes\.json**

```
{
  "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:us-west-2:0123456789012:deadletter\", \"maxReceiveCount\":\"5\"}"
}
```

## Binary files<a name="cli-usage-parameters-file-binary"></a>

For commands that take binary data as a parameter, specify that the data is binary content by using the `fileb://` prefix\. Commands that accept binary data include: 
+  **`aws ec2 run-instances:`** `--user-data` parameter\. 
+  **`aws s3api put-object:`** `--sse-customer-key` parameter\. 
+  **`aws kms decrypt:`** `--ciphertext-blob` parameter\. 

The following example generates a binary 256\-bit AES key using a Linux command line tool, and then provides it to Amazon S3 to encrypt an uploaded file server\-side\. 

```
$ dd if=/dev/urandom bs=1 count=32 > sse.key
32+0 records in
32+0 records out
32 bytes (32 B) copied, 0.000164441 s, 195 kB/s
$ aws s3api put-object \
    --bucket my-bucket \
    --key test.txt \
    --body test.txt \
    --sse-customer-key fileb://sse.key \
    --sse-customer-algorithm AES256
{
    "SSECustomerKeyMD5": "iVg8oWa8sy714+FjtesrJg==",
    "SSECustomerAlgorithm": "AES256",
    "ETag": "\"a6118e84b76cf98bf04bbe14b6045c6c\""
}
```

For another example referencing a file containing JSON\-formatted parameters, see [Attaching an IAM managed policy to an IAM user](cli-services-iam-policy.md)\. 