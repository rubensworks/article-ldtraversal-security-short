## Introduction
{:#introduction}

Semantic Web technologies enable Knowledge Graphs to be exposed on the Web,
which is often done using [SPARQL endpoints](cite:cites spec:sparqlprot).
Such endpoints act as *centralized* repositories for Knowledge Graphs,
which can lead to scalability issues, or even privacy and legal issues when they capture personal data.
<span class="comment" data-author="RV">Especially in the first paragraph, I would keep the discussion wider (such that it also applies to non-RDF KGs). Note that the abstract also is not RDF-specific. The discussion I would suggest here is APIs over large-scale aggregators versus over smaller KGs.</span>
Initiatives such as [Solid](cite:cites solid) propose a radical *decentralization* of Knowledge Graphs across *personal data vaults*.
Such data vaults are built upon the concepts of [Linked Data](cite:cites linkeddata),
where documents are linked to each other, and more information about certain things can be found by dereferencing links via the *follow-your-nose principle*.
<span class="comment" data-author="RV">Be a bit more precise in the before/after description. What does data look like now, what do personal data vaults make it look like? Also, vaults should not be the argument for needing all of this (still niche), but rather an example. It shows that traversal-based querying is not just for open data / experimentation.</span>
This decentralization of information leads to difficulties for traditional query processing,
since no single endpoint can be consulted for resolving queries.
Fortunately, a technique called [Link-Traversal-based Query Processing (LTQP)](cite:cites sparqllinktraversal, linktraversal)
is able to query over an interlinked set of Linked Data documents.
An LTQP query engine does not depend on a pre-indexed dataset,
but instead executes a query over a set of *seed* documents,
and continuously follows links to other documents.

<span class="comment" data-author="RV">If it were me, I would start with a general paragraph of the field, and its unknowns. Second paragraph, query. Third, security (= the one below)</span>

Since LTQP is still a relative young area of research,
there are still a number of open problems that need to be tackled,
notably [result completeness and query termination](cite:cites linktraversal).
Aside from these known issues, **we investigate in this article security issues related to LTQP engines**,
which may threaten the integrity of the user's data, machine, and user experience.
<span class="comment" data-author="RV">WHY? Because that will be needed if we're also applying this to private data. More whys?</span>
Specifically, we focus on data-driven security issues that are inherent to LTQP due to the fact that it requires a query engine to follow links on the Web,
which is an uncontrolled, unpredictable and potentially unsafe environment.
Instead of analyzing a single security threat in-depth,
we perform a broader high-level analysis of multiple security threats.
<del class="comment" data-author="RV">This high-level analysis may serve as a basis for more in-depth research on LTQP security threats in the future.</del>

Since LTQP is <span class="rephrase" data-author="RV">still mostly an academic topic, and is not being used in real-world applications,</span>
<span class="comment" data-author="RV">Be careful not to undermine the relevance and impact of your work. Make still sure why it <em>will</em> become more relevant.</span>
we can not learn from security issues that arose in existing systems.
Instead, we draw inspiration from the domains of crawling and Web browsers in [](#related-work),
and draw links to what impact these known security issues will have on LTQP query engines.
In [](#use-case), we introduce a guiding use case that will be used to illustrate different threats with.
Next, we list 10 data-driven security vulnerabilities related to LTQP in [](#threats),
which are derived from known vulnerabilities in similar domains.
For each vulnerability, we provide examples, and sketch possible high-level mitigations.
Finally, we discuss the future of LTQP security and conclude in [](#conclusions).
