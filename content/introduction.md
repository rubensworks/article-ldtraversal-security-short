## Introduction
{:#introduction}

Semantic Web technologies enable Knowledge Graphs to be exposed on the Web,
which is typically done using [SPARQL endpoints](cite:cites spec:sparqlprot).
Such endpoints act as *centralized* repositories for Knowledge Graphs,
which can lead to scalability issues, or even privacy and legal issues when they capture personal data.
Initiatives such as [Solid](cite:cites solid) propose a radical *decentralization* of Knowledge Graphs across *personal data vaults*.
Such data vaults are built upon the concepts of [Linked Data](cite:cites linkeddata),
where documents are linked to each other, and more information about certain things can be found by dereferencing links via the *follow-your-nose principle*.
This decentralization of information leads to difficulties for query execution,
since no single endpoint can be consulted for resolving queries.
Fortunately, a technique called [Link-Traversal-based Query Processing (LTQP)](cite:cites sparqllinktraversal, linktraversal)
has been introduced that is able to query over an interlinked set of Linked Data documents.
An LTQP query engine does not depend on a pre-indexed dataset,
but instead executes a query over a set of *seed* documents,
and continuously follows links to other documents.

Since LTQP is still a relative young area of research,
there are still a number of open problems that need to be tackled
related to [result completeness and query termination](cite:cites linktraversal).
Aside from these known issues, **we investigate in this article security issues related to LTQP**.
Specifically, we focus on security issues that are inherent to LTQP due to the fact that it requires a query engine to follow links on the Web,
which is an uncontrolled, unpredictable and potentially unsafe environment.

Since LTQP is still mostly an academic topic, and is not being used in real-world applications,
we can not learn from security issues that arose in existing systems.
Instead, we draw inspiration from the domains of crawling and Web browsers,
and draw links to what impact these known security issues will have on LTQP query engines.
