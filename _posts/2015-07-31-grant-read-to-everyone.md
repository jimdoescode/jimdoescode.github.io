---
layout: post
title:  "Grant Read Access to Everyone in S3"
date:   2015-07-31 15:30:47
categories: aws s3 permissions
---

Just a quick note on how to do this as it was a little tough to find.

If you need to upload something to S3 and you are using the AWS CLI tool. You can make the file readable by everyone using the following command:

```
$ aws s3 cp file.txt s3://bucket-name/ --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers
```

This is useful if you are storing web assets in S3 and want them to be accessed by anyone who visits your website.



