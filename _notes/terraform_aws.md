---
layout: note
title: Terraform / AWS
---

# :thinking: :pencil: :books: :cloud:

---

Embrace IAM roles!

`terraform backend` block and `provider "aws"` blocks both optionally take role ARNs.
These roles _do not need to be the same_.
Terraform will use different roles for different parts of its operation.
The provider role is used to create, update, and destroy AWS resources.
The backend role is used to read and record the state(s) of those resources before and after that work.
So, you can define a role that only has access to read and write to your Terraform remote state bucket _and do nothing else_.
We even set up `TerraformRemoteStateReadWrite` and `TerraformRemoteStateRead` (only) roles, the latter used for looking up values via `terraform_remote_state`.

---

Services defining / "owning" their own security groups has proven a useful pattern.
Even if permissions in two different groups are identical, it's good to dedicate groups to individual services.
1. This lets you easily know when a group is not in use and can be removed
2. If you open up a VPC peering connection (see below), it's easy to scope access within the peered VPC to individual services instead of opening up, say, every service in the private subnets;
you only add new ingress rules to the security group of the service that needs to be accessed.

---

A good pattern for bucket modules: create "read" and "write" policies and export their arns.
The result is a module that:
- has useful "public methods" (the policies, which multiple clients can attach to roles as needed)
- encapsulates details (no need to expose KMS keys encrypting bucket contents, policy is defined once instead of repeatedly by N clients)

---

Always prefer separate resources to inline blocks on resources.
Main example: `aws_security_group` and `aws_security_group_rule`.
Inline rule blocks and separate rule resources will clash.
Separate resources are more extensible, particularly if you need a separate repo to create the rule.
Why would you do that?
One example is VPC peering.
We manage all VPC peering connections in a single repository to 1) share boilerplate Terraform code and 2) keep track of them all in one place (which can be especially important given the potential for overlapping CIDR ranges).
Peering connections are established to serve some purpose—service A needs access to service B.
The security group rules required to support that connection are directly related to the connection itself—there is no use in having one without the other.
Therefore we want that "neutral, third-party" codebase to define the rules alongside the connection as a single, cohesive "unit".