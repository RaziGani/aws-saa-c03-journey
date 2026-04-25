A role is like a security badge on a hook. Permissions are baked into the badge, not the person. Anyone trusted to use the badge can grab it, gets temporary credentials while holding it, and drops it back when done. 

Every role has TWO policies attached, answering two different questions. This is the distinction that locks roles in. Without a trust policy, nobody can assume the role.
**Trust policy** (also called the "assume role policy"): Specifies the Principal
**Permissions policy**: Same syntax as identity-based policies — Effect, Action, Resource. Same Allow/Deny rules.


**Lab: Create the role via Console.**

