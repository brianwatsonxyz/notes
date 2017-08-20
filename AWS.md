# Amazon Web Services

## IAM
Access to AWS resources are determined by the union of an entity's permissions.
### Policies
A policy document can be user-created, or provided by Amazon. They consist of an Effect (Allow/Deny), Action (service:APIs), and Resource (ARNs). Can be attached to Users, Roles, and Groups.
### Roles
Roles are associated with a number of policies. The policies can be from the set of user created and managed policies, or provided inline.
### Users
Allow access to both resources in AWS and access to the console. A user can have both a password and access key, with the access key providing the programmatic access.
### Groups
Groups are associated with a number of policies. They provide an easier way to manage access because they allow job-based management, and if a change is needed it can be made at the group level instead of each individual user.
## S3
## EC2
## DynamoDB
## General