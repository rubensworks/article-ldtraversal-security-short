## Related Work
{:#related-work}

This section lists relevant related work in the topics of LTQP and security.

### Link-Traversal-based Query Processing

More than a decade ago, [Link-Traversal-based Query Processing (LTQP)](cite:cites sparqllinktraversal, linktraversal)
was been introduced as an alternative query paradigm for enabling query execution over document-oriented interfaces.
These documents are usually [Linked Data](cite:cites linkeddata) serialized using any RDF serialization.
RDF is suitable for LTQP and decentralization because of its global semantics,
which allows queries to be written independently of the schemas of specific documents.
In order to execute these queries, LTQP processing occurs over live data,
and discovers links to other documents via the *follow-your-nose principle* during query execution.
This is in contrast to the typical query execution over centralized database-oriented interfaces such as SPARQL endpoints,
where data is assumed to be loaded into the endpoint beforehand,
and no additional data is discovered during query execution.

Concretely, LTQP typically starts off with an input query and a set of seed documents.
The query engine then dereferences all seed documents via an HTTP `GET` request,
discovers links to other documents inside those documents,
and recursively dereferences those discovered documents.
Since document discovery can be a very long (or infinite) process,
query execution happens during the discovery process
based on all the RDF triples that are extracted from the discovered documents.
This is typically done by implementing these processes in an [iterative pipeline](cite:cites Squin).
Furthermore, since this discovery approach can lead to a large number of discovered documents,
different [reachability criteria](cite:cites reachability_semantics) have been introduced
as a way to restrict what links are to be followed for a given query.

So far, LTQP research in the area of security has been limited.
[One work has indicated the importance of _trustworthiness_](cite:cites guidedlinktraversal)
during link traversal, as people may publish false or contradicting information,
which would need to be avoided or filtered out during query execution.
[Another work mentioned the need for LTQP engines to adhere to `robots.txt` files](cite:cites datasummaries)
in order to not lead to unintentional denial of service attacks of data publishers.
Given the focus of our work on data-driven security vulnerabilities related to LTQP engines,
we only consider this issue of _trustworthiness_ further in this work,
and omit the security vulnerabilities from a data publisher's perspective.

### Vulnerabilities of RDF Query Processing 
{:#related-work-rdf-query-processing}

Research involving the security vulnerabilities of RDF query processing
has been primarily focused on [injection attacks within Web applications that internally send SPARQL queries to a SPARQL endpoint](cite:cites sqlinjection, sparqlinjectionattacks, semguard, insecurysemwebframework).
So far, no research has been done on vulnerabilities specific to RDF federated querying or link traversal.

### Linked Data Access Control

[Kirrane et al.](cite:cites rdfaccesscontrol) surveyed the existing approaches for achieving access control in RDF,
for both authentication and authorization.
The authors mention that only a minority of those works apply specifically to the document-oriented nature of Linked Data.
They do however mention that non-Linked-Data-specific approaches could potentially be applied to Linked Data in future work.
To the best of our knowledge, no security vulnerabilities have yet been identified for any of these.

### Web Crawlers

[Web crawling](cite:cites crawling) is a process that involves collecting information on the Web by following links between pages.
Web crawlers are typically used for Web indexing to aid search engines.
[Focused crawling](cite:cites focusedcrawling) is a special form of Web crawling that prioritizes certain Web pages,
such as Web pages about a certain topic, or domains for a certain country.
LTQP can therefore be considered as an area of focused crawling where the priority lies in achieving query results.

One related work in this area involves [abusing crawlers to initiate attacks on other Web sites](cite:cites crawlerattacks).
This may cause performance degradation on the attacked Web site,
or could even cause the crawling agent to be blocked by the server.
These attacks involve convincing the crawler to follow a link to a third-party Web site
that exploits a certain vulnerability, such as an SQL injection.
Additionally, this work describes a type of attack that allows vulnerable Web sites to be used
for improving the [PageRank](cite:cites pagerank) of an attacker-owned Web site via forged backlinks.

Some other works focus on mitigation of so-called [_crawler traps_](cite:cites crawlertraps, mercatorcrawler) or _spider traps_.
These are sets of URLs that cause an infinite crawling process,
which can either be intentional or accidental.
Such crawler traps can have multiple causes:

* Links between dynamic pages that are based on URLs with query parameters;
* Infinite redirection loops via using the HTTP 3xx range;
* Links to search APIs;
* Infinitely paged resources, such as calendars;
* Incorrect relative URLs that continuously increase the URL length.

Crawler traps are mostly discovered through human intervention when many documents in a single domain are discovered.
Recently, [a new detection technique was introduced](cite:cites crawlertrapsdetection)
that attempts to measure the _distance_ between documents,
and rejects links to documents that are too similar.

### Web Browsers

Web browsers enable users to visualize and interact with Web pages.
This interaction is closely related to LTQP,
with the main difference that LTQP works autonomously,
while Web browsers are user-driven.
[Silic et al.](cite:cites securitymodernwebbrowserarchitecture)
analyzed the architectures of modern Web browsers,
determined the main vulnerabilities,
and discuss how these issues are handled.
They list the following main threats for Web browsers:

1. **System compromise**: Malicious arbitrary code execution with full privileges on behalf of the user. For example, exploits in the browser or third-party plugins caused by bugs. These types of attacks are mitigated through automatic updates once exploits become known.
2. **Data theft**: Ability to steal local network or system data. For example, a Web page includes a subresource to URLs using the file scheme (`file://`), which are usually blocked.
3. **Cross domain compromise**: Code from a Fully Qualified Domain Name (FQDN) executes code (or reads data) from another FQDN. For example, a malicious domain could extract authentication cookies from your bank's website you are logged into. This is usually blocked through the same-origin policy, but can be explicitly allowed through [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}.
4. **Session hijacking**: Session tokens are compromised through theft or session token prediction. For example, [cross-domain request forgery (CSRF)](cite:cites csrf) is a type of attack that involves an attacker forcing a user logged in on another Web site to perform an action without their consent. Web browsers do not protect against these attacks, but they are typically handled by Web frameworks via the [Synchronizer Token Pattern](cite:cites synchronizertokenpattern).
5. **User interface compromise**: Manipulating the user interface to trick the user into performing an action without their knowledge. For example, placing an invisible button in front of another button. This category also includes CPU and memory hogging to block the user from taking any further actions. Web browsers have limited protections for these types of attacks that involve placing limitations on user interface manipulations.
