## Abstract
<!-- Context      -->
The societal and economical consequences surrounding Big Data-driven platforms
have increased the call for decentralized solutions.
However, retrieving and querying data in more decentralized environments
requires fundamentally different approaches,
whose properties are not yet well understood.
Link-Traversal-based Query Processing (LTQP) is a technique
for querying over decentralized data networks,
in which the query engine <span class="rephrase" data-author="RV">runs client-side by traversing links</span> in this decentralized Web.
Since decentralized environments are potentially unsafe due to their non-centrally controlled nature,
<!-- Need         -->
there is a need for LTQP query engines to be <span class="rephrase" data-author="RV">stable</span> enough
as to not lead to security issues with the <span class="rephrase" data-author="RV">user</span>'s machine and data.
<span class="comment" data-author="RV">Interesting as to whom is the user here; might need to define roles (publisher, querier, …); maybe not in abstract but in article.</span>
<!-- Task         -->
As such, we have performed an analysis of potential security vulnerabilities of LTQP.
<!-- Object       -->
This article provides an overview of security threats in related domains,
which are used as inspiration for the identification of 10 LTQP security threats.
Each threat is explained, together with an example, and one or more theoretical <span class="comment" data-author="RV">why theoretical? perhaps rather <q>avenues for …</q>?</span> mitigations are proposed.
<!-- Findings     -->
<!-- Conclusion   -->
We conclude with several concrete recommendations for LTQP query engine developers and data publishers
as a first step to mitigate some of these issues.
<!-- Perspectives -->
With this work, <span class="rephrase" data-author="RV">we aim to trigger more research interest in this domain</span>,
<span class="comment" data-author="RV">I think projected impact should be wider. Perhaps in terms of unknowns being filled in (cfr. non-understood properties in the first two sentences)</span>
so that all of the discussed threats can be mitigated,
and that other potential threats may be uncovered.
<span class="comment" data-author="RV">Wider: in terms of missing building blocks for decentralization</span>
<del class="comment" data-author="RV">This---and future work---will be crucial to enable stable and secure querying over a decentralized Web.</del>
