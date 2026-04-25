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
