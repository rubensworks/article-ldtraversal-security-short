## Abstract
<!-- Context      -->
Decentralization efforts require us to rethink how we query over data,
as centralized querying approaches are often not fully usable.
Link-Traversal-based Query Processing (LTQP) is a well-suited approach
for querying over such decentralized environments,
where the query engine runs client-side by traversing links in this decentralized Web.
Since decentralized enviroments are potentially unsafe due to their non-centrally controlled nature,
<!-- Need         -->
there is a need for LTQP query engines to be stable enough
as to not lead to security issues with the user's machine and data.
<!-- Task         -->
As such, we aim to initiate an analysis of potential security vulnerabilities of LTQP.
<!-- Object       -->
Concretely, this article provides an overview of security threats in related domains,
which are used as inspiration for the identification of ten LTQP security threats.
Each threat is explained, together with an example, and one or more theoretical mitigations are proposed.
<!-- Findings     -->
<!-- Conclusion   -->
We conclude with several concrete recommendations for LTQP query engine developers and data publishers
as a first step to mitigate some of these issues.
<!-- Perspectives -->
With this work, we aim to trigger more research interest in this domain,
so that all of the discussed threats can be mitigated,
and that other potential threats may be uncovered.
This --and future work-- will be crucial to enable stable and secure querying over a decentralized Web.

