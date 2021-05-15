---
layout: post
title: GitLab 8.4.4 S3 Backup Issues
date: '2016-02-16 03:26:44'
permalink: "/gitlab-8-4-4-s3-backup-issues/"
---

According to https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/raketasks/backup_restore.md, GitLab currently use [Fog](https://github.com/fog/fog) gem to support many backup backends. Underlying is some issues which I met while setting up the Amazon S3 backup. 

1. IAM Permission Setup

  Incorrect IAM permission setup may lead to upload failures. 

  The solution is quite simple, visit https://console.aws.amazon.com/iam/home?#users and paste example policy JSON under **Inline Policies**. If you get *Statement is missing required element - Statement "$ID" is missing "Principal" element. *, it means that you have mistakenly mixed S3 Policy with User Permission Policy. Try the above link. 

2.  OpenSSL certificate invalid

  This issue may occur when you use the regions name in http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html. Code like `us-east-1` will cause Fog to generate such a URL: https://your-bucket-name-in-s3.s3-us-east-1.amazonaws.com. But this URL now perform a permanent redirect to https://your-bucket-name-in-s3.s3-ap-northeast-1.amazonaws.com. This means AWS preferred to use the code `ap-northeast-1` instead of `us-east-1`. This may related to a minor update in S3, but it used to return `s3.amazonaws.com` in the `<Endpoint>` field of XML contents. Fog just concat your bucket name and append that endpoint URL, generating such URL like https://your-bucket-name-in-s3.your-bucket-name-in-s3.s3-ap-northeast-1.amazonaws.com. And that cause this certificate issue. 

  Solution is quite simple as well, directly use final code such as `ap-northeast-1` and access the bucket without redirection. 