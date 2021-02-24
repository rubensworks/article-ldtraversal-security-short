## Related Work
{:#related-work}

This section lists relevant related work in the topics of LTQP and security.

### Link-Traversal-based Query Processing

More than a decade ago, [Link-Traversal-based Query Processing (LTQP)](cite:cites sparqllinktraversal, linktraversal)
has been introduced as an alternative query paradigm for enabling query execution over document-oriented interfaces.
LTQP executes queries over live data, and discover links to other documents during query execution.
This is in contrast to the typical query execution over database-oriented interfaces such as [SPARQL endpoints](cite:cites spec:sparqlprot),
where data is assumed to be loaded into the endpoint beforehand,
and no additional data is discovered during query execution.

Concretely, LTQP typically starts off with an input query and a set of seed documents.
The query engine then dereferences all seed documents via an HTTP GET request,
discovers links to other documents inside those documents,
and recursively dereferences those discovered documents.
Based on all the RDF triples that have been discovered from the discovered documents, query execution can be performed.
Since document discovery can be a very long (or infinite) process,
an [iterative approach](cite:cites Squin) was introduced that allows query execution during the discovery process.
Furthermore, since this discovery approach can lead to a huge amount of discovered documents,
different [reachability criteria](cite:cites reachability_semantics) have been introduced
as a way to restrict what links are to be followed for a given query.

So far, most research into LTQP has been in the areas of [formalization](cite:cites reachability_semantics,linktraversalfoundations), [performance improvements](cite:cites WalkingWithoutMap,linktraversalstrategies,linktraversalhybrid), and [query syntax](cite:cites LDQL).
[One work has indicated the importance of _trustworthiness_](cite:cites guidedlinktraversal)
during link traversal, as people may publish false or contradicting information,
which would need to be avoided or filtered out during query execution.
Other than this single work, no works have looked into the security implications of LTQP in general,
which is what we aim to tackle in this work.

### Vulnerabilities of RDF Query Processing 
{:#related-work-rdf-query-processing}

Research involving the security vulnerabilities of RDF query processing
has been primarily focused on injection attacks within Web applications
that internally send SPARQL queries to a SPARQL endpoint.
So far, no research has been done on vulnerabilities specific to RDF federated querying or link traversal.
As such, we list the relevant work on single-source SPARQL querying hereafter.

The most significant type of security vulnerability in Web applications is _Injection through User Input_,
of which [SQL injection attacks](cite:cites sqlinjection) are a primary example.
[Orduna et al.](cite:cites sparqlinjectionattacks) investigate this type of attack in the context of SPARQL queries,
and show that parameterized queries can help avoid this type of attacks.
The authors implemented parameterized queries in the [Jena framework](cite:cites jena) as a mitigation example.

[SemGuard](cite:cites semguard) is a system that aims to detect injection attacks in both SPARQL and SQL queries for query engines that support both.
A motivation of this work is that the use of parameterized queries is not always possible,
as systems may already have been implemented without them,
and updating them would be too expensive.
This approach is based on the automatic analysis of the incoming query's parse tree.
It will check if the parse tree only has a leaf node for the expected user input, compared to the original template query's parse tree.
If it does not have a leaf node, this means that the user is attempting to execute queries that were not intended by the application developer.

[Asdhar et al.](cite:cites insecurysemwebframework) analyzed injection attacks to Web applications
via the [SPARQL query language](cite:cites spec:sparqllang) and the [SPARQL update language](cite:cites spec:sparqlupdate).
Furthermore, they provide _SemWebGoat_, a deliberately insecure RDF-based Web application for educational purposes around security.
All of the discussed attacks involve some form of injection,
leading to retrieval or modification of unwanted data,
or denial-of-service by for example injecting the `?s ?p ?o` pattern.

### Linked Data Access Control

[Kirrane et al.](cite:cites rdfaccesscontrol) surveyed the existing approaches for achieving access control in RDF,
for both authentication and authorization.
The authors mention that only a minority of those works apply specifically to the document-oriented nature of Linked Data.
They do however mention that non-Linked-Data-specific approaches could potentially be applied to Linked Data in future work.
Hereafter, we briefly discuss the relevant aspects of access control research that applies to Linked Data.
To the best of our knowledge, no security vulnerabilities have yet been identified for any of these.

**Authentication:**

Authentication involves verifying an agent's identity through certain credentials.
A [WebID](https://www.w3.org/wiki/WebID){:.mandatory} (Web Identity and Discovery)
is a URL through which agents can be identified on the Web.
[WebID-TLS](cite:cites spec:webidtls) is a protocol that allows authentication of WebID agents via TLS certificates.
However, due to the limited support of such certificates in Web browsers, its usage is hindered.
[WebID-OIDC](cite:cites spec:webidoidc) is a more recent protocol is based
on the [OpenID Connect](cite:cites spec:oidc) protocol for authenticating WebID agents.
Due to its compability with modern Web browsers, WebID-OIDC is frequently used inside the Solid ecosystem.

**Authorization:**

Authorization involves determining who can read or write what kind of data.
[Web Access Control](cite:cites spec:wac) is an RDF-based access control system that works in a decentralized fashion.
It enables declarative access control policies for documents to be assigned to users and groups.
Due to its properties, it is being used as default access control mechanism in the Solid ecosystem.
[Sacco et al.](cite:cites accesscontrolwebofdata) extend Web Access Control to not only declare document-level access,
but also on resource, statement and graph level.
[Costabello et al.](cite:cites accesscontrolhttp) introduce the Shi3ld framework that enables access control for [Linked Data Platform](cite:cites spec:ldp).
Both SPARQL-based and a SPARQL-less variants of the framework are proposed.
[Kirrane et al.](cite:cites securemanipulationlinkeddata) introduce a framework
for enabling query-based access control via query rewriting of simple graph pattern queries.
Finally, [Steyskal et al.](cite:cites accesscontrollinkeddataodrl) provide an approach that is based on the Open Digital Rights Language.

### Web Crawlers

[Web crawling](cite:cites crawling) is a process that involves collecting information on the Web by following links between pages.
Web crawlers are typically used for Web indexing to aid search engines.
[Focused crawling](cite:cites focusedcrawling) is a special form of Web crawling that prioritizes certain Web pages,
such as Web pages about a certain topic, or domains for a certain country.
LTQP can therefore be considered as an area of focused crawling that where the priority lies in achieving query results.

Web crawlers are often used for discovering vulnerable Web sites,
for example through [_Google Dorking_](cite:cites googledorks),
which involves using Google Search to find Web sites that are misconfigured or use vulnerable software.
Furthermore, crawlers are often used to find private information on Web sites.
Such issues are however not the focus of this work.
Instead, we are interested in the security of the crawling process itself,
for which little research has been done to the best of our knowledge.

One related work in this area involves [abusing crawlers to initiate attacks on other Web sites](cite:cites crawlerattacks).
This may cause performance degradation on the attacked Web site,
or could even cause the crawling agent to be blocked by the server.
These attacks involve convincing the crawler to follow a link to a third-party Web site
that exploits a certain vulnerability, such as an SQL injection.
Additionally, this work describes a type of attack that allows vulnerable Web sites to be used
for improving the [PageRank](cite:cites pagerank) of an attacker-owned Web site via forged backlinks.

### Web Browsers

Web browsers enable users to visualize and interact with Web pages.
This interaction is closely related to LTQP,
with the main difference that LTQP works autonomously,
while Web browsers are user-driven.
Considering this close resemblance between these two domains,
we give an overview of the main security vulnerabilities in Web browsers.

**Modern Web browser architecture:**

[Silic et al.](cite:cites securitymodernwebbrowserarchitecture)
analyzed the architectures of modern Web browsers,
determined the main vulnerabilities,
and discuss how these issues are coped with.

Architecture-wise, browsers can be categorized into monolithic and modular browser architectures.
The difference between the two is that the former does not provide isolation between concurrently executed Web programs, while the latter does.
The authors argue that a modular architecture is important for security, fault-tolerance and memory management.
They focused on the security aspects of the [Chrome browser architecture](cite:cites securitychromium),
which consist of separate modules for the rendering engine, browser kernel, and plugins.
Each of these modules is isolated in its own operating system process.

Silic et al. list the following main threats for Web browsers:

1. **System compromise**: Malicious arbitrary code execution with full privileges on behalf of the user. For example, exploits in the browser or third-party plugins caused by bugs. These types of attacks are mitigated through automatic updates once exploits become known.
2. **Data theft**: Ability to steal local network or system data. For example, a Web page includes a subresource to URLs using the file scheme (`file://`). which are usually blocked.
3. **Cross domain compromise**: Code from a Fully Qualified Domain Name (FQDN) executes code (or reads data) from another FQDN. For example, a malicious domain could extract authentication cookies from your bank's website you are logged into. This is usually blocked through the same-origin policy, but can be explicitly allowed through [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}.
4. **Session hijacking**: Session tokens are compromised through theft or session token prediction. For example, [cross-domain request forgery (CSRF)](cite:cites csrf) is a type of attack that involves an attacker forcing a user logged in on another Web site to perform an action without their consent. Web browsers do not protect against these, but are typically handled by Web frameworks via the [Synchronizer Token Pattern](cite:cites synchronizertokenpattern).
5. **User interface compromise**: Manipulating the user interface to trick the user into performing an action without their knowledge. For example, placing an invisible button in front of another button. This category also includes CPU and memory hogging to block the user from taking any further actions. Web browser have limited protections for these types of attacks that involve placing limitations on user interface manipulations.

**Lessons from Google Chrome:**

[Reis et al.](cite:cites securitychromelessons) discuss on the three problems Google Chrome developers focus on to mitigate attacks:

1. **Reducing vulnerability severity**: In the real world, large projects such as Web browsers always contain bugs. Given this reality, Google Chrome consists of several sandbox layers reducing the damage should an exploit be discovered in one of the layers. The difficulty here lies in the fact that Web compatibility should be maintained, so that security restrictions do not break people's favorite Web sites.
2. **Reducing window of vulnerability**: If an exploit has been discovered, it should be patched as soon as possible. Google Chrome follows automated testing to ship security patches as soon as possible. All existing Chrome installations check for updates every five hours, and update in the background without disrupting the user experience.
3. **Reducing frequency of exposure**: In order to avoid people from visiting malicious Web sites for which the browser has not been patched yet, Google Chrome makes use of a continuously updating database of such Web sites. This will show a warning to the user before visiting such a site.

**SQL Injection**

[SQL injection attacks](cite:cites sqlinjection) are one of the most common vulnerabilities on Web sites
where (direct or indirect) user input is not properly handled, and may lead to the attacker performing unintended SQL statements on databases.
These types of attacks are typically mitigated through strong input validation, which are typically available in reusable libraries.

**Techniques for mitigating browser vulnerabilities:**

[Browser Hardening](cite:cites hardening) is based on the concept of reducing privileges of browsers to increase security.
For example, browsers can be configured to disabled JavaScript and Adobe Flash, or whitelisted to trusted Web sites.

[Fuzzing](cite:cites fuzzing) is a technique that involves generating random data as input to software.
Major Web browsers such as Google Chrom and Microsoft Edge perform extensive fuzzed testing by generating random Web pages and running them through the browser to detect crashes and other vulnerabilities.
