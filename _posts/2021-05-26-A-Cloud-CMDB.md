---
layout: post
title: A Cloud CMDB
---
We’ve recently been working with a large customer who has transitioned fully to the cloud and wishes to update their support systems and process. As part of this project we were asked to look at their CMDB system.

Before they moved to the cloud, their CMDB was part of their ITIL solution and this ITIL solution was front and centre of their systems management and provisioning. That is to say, no system could be provisioned, modified or decommissioned unless the relevant change process had been completed and all of the associated ITIL processes enacted.

Following the move to the cloud, infrastructure is now delivered as code. Many DevOps teams use workflow in their development systems, such as the git PR process to manage change and approvals. Automation and Auto Scaling are employed to ensure the appropriate number of resources are deployed at any given time to match the workload.

Attempting to adhere to the “single source of truth” principle presents challenges with a Cloud  CMDB, because the single source of truth of a resource definition lives in the code base, and the single source of truth of how many resources are in existence at any given time is the Cloud Provider who is provisioning these resources.

Exploring this concept further we began to question what information we needed to store in our CMDB.

One of the realisations was that the CMDB was not provided for the DevOps teams to use, but for support teams such as change and incident management, finance, etc.

The move to cloud had changed the internal dynamic, for example, DevOps teams were responsible for the support and incident management of their products (you built it, you run it).
Automation and the ability to scale up or down on demand had also changed how we thought about our CMDB. We realised we were no longer interested in information such as the number of servers running at any given time, but started to think about systems as a whole and the health of these systems.

## Cattle Not Pets

This change in thinking aligns with the [Cattle not pets](http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/) analogy 

>“Arrays of more than two servers, that are built using automated tools, and are designed for failure, where no one, two, or even three servers are irreplaceable.”


We also realised that in the current CMDB we only tracked information to a certain degree of detail. For example, we would record certain information about a server, disk size, memory, CPU, etc, but we did not log every single component, capacitor, resistor, etc - that would be crazy.

We wondered what would happen if we treated systems in the same way. For example, we might define a system to serve web content. We would define some information, such as the number of concurrent users we expect it to serve and the required response times, but we realised we were not concerned about the number of servers this system ran on. Nor even if it ran on servers at all. We were embracing Serverless technology and SaaS, which further supported our hypothesis that we were less and less interested in physical hardware specs and more interested in systems and their compliance with desired behaviour.

![graph showing change in resources over time](/images/resource_ts.png)
The above graph shows how deployment of a particular resource changes over time.


Having gained experience of running workloads in the cloud, we observed that for most activities including support, incident resolution and cost control we were still not interested in the number of servers or resources deployed at any given time, but how this number compared to the typical behaviour of the system under similar conditions. We noticed we were increasingly looking at graphs of system behaviour over time and identifying differences, rather than discrete values.

## Ebb and Flow

This led us to the realisation that our next CMDB needed to be a time series representing the changes to state of our cloud infrastructure. We could represent any resource as a single point of telemetry, because the definition of this resource was encapsulated in code. 

Duplicating this into a CMDB was time consuming and expensive - both in terms of Engineering effort and in terms of the amount of duplicate data we would need to store and search, compared to a single telemetry point to represent the presence of any defined resource.

Individual DevOps teams were already used to using the metrics provided by their Cloud Provider to see this data for the accounts and infrastructure they maintain, but we still wanted a global view of all cloud assets.

To deliver this we chose a SaaS product called Wavefront (now Tanzu Observability). To describe it as a SaaS time series database would be massively understating it, as it has powerful data analytic functions built in, allowing us to query and alert on any of the data we ingest into it in a number of different ways.

Initially the new CMDB was met with some scepticism, especially amongst those parts of the business that traditionally had more hands off roles. However, for our DevOps teams it has turned out to be exactly the right tool, containing the correct information they require to manage a large and complex estate of cloud based resources.

The fears of the centralised teams such as finance and governance have not been released either, infact these areas have probably benefited most from the new system. By not focusing on the minute detail we have moved all of our teams further up the technology stack and closer to the customer allowing them to focus their effort more intelligently.
