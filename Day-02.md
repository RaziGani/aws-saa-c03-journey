{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::company-reports", <-- the bucket itself (no slash)
        "arn:aws:s3:::company-reports/*" <-- every object inside the bucket
      ]
    }
  ]
}

The above policy Explicitly Allows GetObject and ListBucket on the S3 from the company-reports folder. (WRONG)
Corrections: company-reports is the bucket name. S3 doesn't actually have folders.


{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "ap-southeast-1"
        }
      }
    }
  ]
}

The above policy Explicitly Allow all Actions on the EC2 for everything in the account. However Explicit Deny overides the Explicit Allow when Terminating an EC2 instance from the region "ap-southeast-1" (WRONG)
Correction: Full EC2 access in every region, but I can only TERMINATE instances inside Singapore. Termination in any other region is blocked.


**The original wrong policy** 
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "Statement2",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::/*"
            ]
        }
    ]
}
Tested the policy above on IAM Policy Simulator. 
Results Returned: 
ListAllMyBuckets → denied (expected: allowed)
GetObject on iamadmin-study-bucket/anything.txt → denied (expected: allowed)
DeleteObject → denied (this one is correct)

Bug 1 — wrong action name. You wrote s3:ListBucket. The right action is s3:ListAllMyBuckets.
Bug 2 — your GetObject ARN is malformed. You wrote arn:aws:s3:::/*. The bucket name is missing.

**The corrected policy**
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBucketsInAccount",
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        },
        {
            "Sid": "ReadObjectsFromStudyBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::iamadmin-study-bucket/*"
        }
    ]
}

Results Returned: 
ListAllMyBuckets →  allowed   1 matching statements. (expected: allowed)
GetObject on iamadmin-study-bucket/anything.txt →  allowed   1 matching statements. (expected: allowed)
GetObject on other-bucket/file.txt →   denied   Implicitly denied (no matching statements). (expected: denied)
DeleteObject → denied (this one is correct)

Sequence — write a buggy policy, simulator catches it, read the error, fix the bugs, simulator confirms 

Version: This just tells AWS which policy language format you’re using so it knows how to read the policy correctly.
Statement: This is the main section where you actually define the rules—basically the “meat” of the policy.
Effect: This decides whether you’re allowing or denying something, so it’s either “Allow” or “Deny.”
Action: This specifies what operations are being controlled, like reading from a bucket or launching an instance.
Resource: This defines which specific resource (or resources) the action applies to, like a particular S3 bucket or EC2 instance.
Condition: This adds extra rules or filters, like only allowing access at certain times or from certain IP addresses.
Principal: This tells who the policy applies to, such as a user, role, or service trying to access something. Principal appears in policies attached to RESOURCES (like S3 bucket policies, role trust policies), not in policies attached to identities (like the ones we wrote today).


**Explicit Deny > Explicit Allow > Default Deny**
Explicit Deny: If a policy clearly says “deny,” it overrides everything else—this prevents accidental access even if another policy allows it.
Explicit Allow: If something is specifically allowed and there’s no deny blocking it, then access is granted—this is how permissions are normally given.
Default Deny: If you didn’t explicitly allow something, it’s automatically denied—this enforces least privilege by making sure nothing is accessible unless you say so.


"arn:aws:s3:::company-reports" (refers to the bucket itself)
arn:aws:s3:::company-reports/* (refers to every object inside the bucket)
Using only arn:aws:s3:::company-reports breaks GetObject; using only arn:aws:s3:::company-reports/* breaks ListBucket. You need both ARNs to allow listing and reading.


StringEquals: { aws:username: "alice" } → applies when username IS alice
StringNotEquals: { aws:username: "alice" } → applies when username is NOT alice
Bool: { aws:MultiFactorAuthPresent: "true" } → applies when MFA WAS used
Bool: { aws:MultiFactorAuthPresent: "false" } → applies when MFA was NOT used

**Hardest part of today.**
After creating a policy, the hardest part is debugging. I need to really need to read understand the Allow/Deny permissions in order to understand the statement.
