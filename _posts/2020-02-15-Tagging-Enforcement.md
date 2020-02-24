---
layout: post
title: Tagging Enforcement!
---
Recently we embarked on a project to build a new cloud hosting solution in both AWS and Azure. We wanted to set some basic standards from day one and where possible to enforce them at resource creation time. One of those standards was to define a set of mandatory resource tags for cost attribution and resource identification and management.

It was important that immediate feedback of any violation of the tagging policy was provided, that is, we needed to enforce the tagging standard at resource creation time and not have a separate process audit and terminate non-compliant resources after the event.

Each cloud has a slightly different way of implementing this functionality. 

## Lets look at AWS first...

We can use an AWS [Service Control Policy](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scp.html) (SCP) to specify which tags must be present at resource creation time. The SCP allows us to specify the tag names, but it does not allow us to define any controls over the values. Some of our tag values, such as cost code, should confirm to a known pattern, so we could, in theory, use a regular expression to validate these.

Our initial SCP was to mandate tagging for all AWS services using a simple catch all wildcard. However not all AWS services support tagging on creation. S3 for example, allows you to tag objects on creation, but not [buckets](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CreateBucket.html). For our use case bucket level tagging was all that was required, but as it was not possible, we attempted to implement object level mandatory tagging.

We use Terraform to manage our infrastructure and it stores state in S3. Unfortunately it does not provide support to tag s3 objects on creation and so by implementing this policy we broke our automated infrastructure management tooling. 

S3 is not the only service that doesn't support tagging on creation, and worse still there does not seem to be a definitive list. For AWS we have had to compromise. We are now deploying SCP's that mandate tagging on a per service basis, EC2 and RDS being our first two use-cases, and we will use tools such as AWS Config and Cloud Custodian to audit tag compliance on other services. If appropriate we will deploy mandatory tagging for other services based on our audit results.

## And so to Azure...

Azure has a similar concept to AWS, simply called [Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview). Initially we thought mandating tagging in Azure would be simpler than AWS as the policy engine is intelligent enough to exclude resources that do not support tagging on creation from enforcement. Therefore is it possible to mandate tagging on create for all resources and Azure will waive this policy enforcement if necessary.

However, we soon tripped over a different problem with Azure Kubernetes Service (AKS). An AKS cluster consists of a *Control Plane* and one or more *Node Pools* of VMs. You can annotate your "jobs" in K8s with things like "I need 2 cores minimum" or "I need x RAM" and the scheduler takes care of placing the work in the correct node pool. However the tags we applied when creating the cluster are not passed to the node pool and our Policy denies their creation causing our cluster to fail.

This is a known bug and is discussed [here](https://github.com/Azure/AKS/issues/3)

We attempted to work around this by excluding internal Azure resources, prefixed "MC_" from our policy, however this also failed because the Azure API would sometimes return this as uppercase and sometimes lowercase. Adding in both cases to our exclusion still fails as at least one of the resources in the group, a public IP, is only identified by a GUID, and whitelisting this would be far too permissive and render our policy ineffective.

There was also concern that this behaviour may be exhibited by other Azure services, for example [Databricks](https://azure.microsoft.com/en-gb/free/services/databricks/), although we did not validate this, which would also require whitelisting. Our concern here was that without a definitive list of problematic services any blanket policy we implement may cause unknown and unexpected problems. We are attempting to implement guardrails that drive good behaviour without blocking development and therefore can not risk implementing overly restrictive policies.

 As with AWS we have had to fall back to deploying tag enforcement policies on a per service basis and regular audit and reporting of compliance for the rest of the estate.

It's important to note that our SCP's only apply to newly created infrastructure. Existing infrastructure that does not comply with the SCP will not be effected until such time it should be relaunched.

The major disadvantage of a per service approach is that should we wish to roll out SCP's for additional services we have to ensure all users of those services are compliant with our tagging policy before we do so, otherwise we may impact their service, especially for events such as autoscale which may fail when they are required causing an incident sceenario. 
