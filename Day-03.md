A role is like a security badge on a hook. Permissions are baked into the badge, not the person. Anyone trusted to use the badge can grab it, gets temporary credentials while holding it, and drops it back when done. 


**The two policies that define a role (trust vs permissions)**
Each role is governed by two separate policies, each serving a distinct purpose—that separation is what defines how the role operates.
**Trust policy** (also known as the assume role policy): Defines who (which Principal) is allowed to assume the role. Without this, the role is effectively unusable because no entity has permission to take it on.
**Permissions policy**: Uses the same structure as standard identity-based policies—Effect, Action, Resource—to define what the role is allowed to do once it has been assumed.


**AssumeRole flow**
When an entity assumes a role:
It invokes the sts:AssumeRole API.
AWS evaluates the role’s trust policy to verify the caller is an allowed Principal.
If authorized, STS (Security Token Service) issues temporary credentials:
AccessKeyId — temporary access key
SecretAccessKey — temporary secret
SessionToken — indicates the credentials are STS-issued
These credentials are short-lived, with a configurable duration (default 1 hour, range 15 minutes to 12 hours).
While valid, they can be used just like standard credentials, but with permissions defined by the role.


**AccessKeyId prefix is a giveaway:**
ASIA... → STS-issued temporary credentials
AKIA... → long-term IAM user access key
This distinction is a common exam signal. If you see ASIA, you’re dealing with temporary credentials; if you see AKIA, they’re permanent.


**The two-way handshake**
The role’s trust policy might state that an account or user is trusted—but that alone doesn’t grant access.
The caller must also have explicit permission in their own identity policy to invoke sts:AssumeRole on that role.
The role’s trust policy says: “I trust this account/user.”
The user’s identity policy must say: “I’m allowed to call sts:AssumeRole on this specific role ARN.”


C:\Users\USER>aws sts get-caller-identity --profile s3reader
{
    "UserId": "AROAY4KHUSFPF2KSHUL5Z:day3-test",
    "Account": "xxxxxxxxxxxx",
    "Arn": "arn:aws:sts::xxxxxxxxxxxx:assumed-role/S3-Reader-Role/day3-test"
}


C:\Users\USER>aws iam list-users --profile s3reader

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:sts::610572472670:assumed-role/S3-Reader-Role/day3-test is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::610572472670:user/ because no identity-based policy allows the iam:ListUsers action
This is denied because the credentials being used belong to an assumed role session (S3-Reader-Role), and that role does not have permission to call iam:ListUsers.


Today was not hard, quite straight forward to understand roles now.

