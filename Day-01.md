# Day 1: IAM Foundation

**Date:** 24 April 2026
**Time spent:** [2 hours]

## What I did today
-Secured root account user: Enable MFA, removed any access keys on root account
-Set billing alarms
-Created iamadmin user account, added to Admins group, enabled MFA, configured CLI with access key

## Shared Responsibility Model
AWS secures the cloud itself, customers secures what they upload into the cloud.
For Example:
-AWS patches the hypervisor running customer's EC2.
-Customers patches the OS on their running EC2.

## IAM: the four primitives

### User
An identity for human or application to gain access to AWS services.

### Group
A container for users.

### Role
An identity with no permanent credentials, assumed temporarily

### Policy
A JSON document that defines what actions are allowed or denied, on what resources and under what conditions

## The golden rule
If root account user's credentials are leaked, the entire AWS account is compromised.

## What was hardest or most confusing today
Understanding "Role" and how it is being used.
