## Introduction
{:#introduction}

Contrary to the Web's initial design as a _decentralized_ ecosystem,
the Web has grown to be a very centralized place,
as large parts of the Web are currently made up of [a few large Bid Data-driven centralized platforms](cite:cites solid).
This large-scale centralization has lead to a number of problems related to [personal information abuse](https://www.theguardian.com/technology/live/2018/apr/10/mark-zuckerberg-testimony-live-congress-facebook-cambridge-analytica),
and other [economic and societal problems](https://fs.blog/2017/07/filter-bubbles/).
In order to solve these problems, there are calls to go back to the original vision of a decentralized Web.
The leading effort to achieve this decentralization is [Solid](cite:cites solid).
Solid proposes a radical *decentralization* of data across *personal data vaults*,
where everyone is in full control of its own personal data vault.
This vault can contain any number of documents,
where its owner can determine who or what can access what parts of this data.
In contrast to the current state of the Web where data primarily resides in a small number of huge data sources,
Solid leads to a a Web where data is spread over a huge number of data sources.

Our focus in this article is not on decentralizing data,
but on finding data after it has been decentralized,
which can be done via _query processing_.
The issue of query processing over data has been primarily tackled from a Big Data standpoint so far.
However, if decentralization efforts such as Solid will become a reality,
we need to be prepared for the need to query over a huge number of data sources.
For example, decentralized social networking applications will need to be able to query over networks of friends containing hundreds or thousands of data documents.
As such, we need new query techniques that are specifically designed for such levels of decentralization.
One of the most promising techniques that could achieve this is called [Link-Traversal-based Query Processing (LTQP)](cite:cites linktraversal, sparqllinktraversal).
LTQP is able to query over a set of documents that are connected to each other via _links_.
An LTQP query engine typically starts from one or more documents,
and _traverses_ links between them in a crawling-manner in order to resolve the given query.

Since LTQP is still a relative young area of research,
in which there are still a number of open problems that need to be tackled,
notably [result completeness and query termination](cite:cites linktraversal).
Aside from these known issues,
we also state the importance of _security_.
Security is [a highly important and well-investigated topic in the context of Web applications](cite:cites sqlinjection, sparqlinjectionattacks),
but it has not yet been investigated in the context of LTQP.
As such, **we investigate in this article security issues related to LTQP engines**,
which may threaten the integrity of the user's data, machine, and user experience,
but also lead to privacy issues if personal data is unintentionally leaked.
Specifically, we focus on data-driven security issues that are inherent to LTQP
due to the fact that it requires a query engine to follow links on the Web,
which is an uncontrolled, unpredictable and potentially unsafe environment.
Instead of analyzing a single security threat in-depth,
we perform a broader high-level analysis of multiple security threats.

Since LTQP is still a relatively new area of research,
its real-world applications are currently limited.
As such, we can not learn from security issues that arose in existing systems.
Instead of waiting for --potentially unsafe-- widespread applications of LTQP,
we draw inspiration from related domains that _are_ already well-established.
Specifically, we draw inspiration from the domains of crawling and Web browsers in [](#related-work),
and draw links to what impact these known security issues will have on LTQP query engines.
In [](#use-case), we introduce a guiding use case that will be used to illustrate different threats with.
After that, we discuss our method of categorizing vulnerabilities in [](#classification).
Next, we list 10 data-driven security vulnerabilities related to LTQP in [](#threats),
which are derived from known vulnerabilities in similar domains.
For each vulnerability, we provide examples, and sketch possible high-level mitigations.
Finally, we discuss the future of LTQP security and conclude in [](#conclusions).
