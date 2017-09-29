---
layout: post
title:  "Setting conditions when set the AssumeRole over Account"
date:   2017-06-12 16:00:00 +0900
categories: AWS Role
comments: true
---
AWS provides Role. User can get authorities using Role.  
Furthermore, User can get authorities over AWS Account if other account makes a Role that allow access.  
Here, two account. A and B.  
`Admin A` of Account A wants to give a Role to some users of Account B.  
How does `Admin A` classify users of Account B?  
AWS suggests the External Id for solving [The Confused Deputy Problem][aws-role-external-id-doc].  
However, External Id can not use at AWS Console. It just supports application's request.  
Refer to [AWS Forum][aws-role-external-id-problem].  
  
Here is my solution instead of the External Id.  
  
{% highlight json %}
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "aws:username": "FILTERED_USER_ID"
        }
      }
    }
  ]
}
{% endhighlight %}
  
`Admin A` should set the condition statement in AssumeRole policy.  
As you know, Role requires two-way AssumeRole policy.  
One is provider's policy, the other is recipient's policy.  
I skip describe the recipient's policy. You can find documents here [AWS Document][aws-role-doc].  
Provider's policy is actually `Trust Relationship` that is second tab in details on the Role.  
This policy has auto-generated when you create the Role.
  
Role provider can classify specific users using other condition statments also.  
Refer to this policy simulator. [PolicyGen][aws-policy-gen]  

[aws-role-doc]: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_permissions-to-switch.html
[aws-role-external-id-doc]: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html
[aws-role-external-id-problem]: https://forums.aws.amazon.com/thread.jspa?threadID=174265
[aws-policy-gen]: https://awspolicygen.s3.amazonaws.com/policygen.html
