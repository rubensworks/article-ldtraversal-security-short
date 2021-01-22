## Data-driven Threats
{:#threats}

We consider two main types of security threats to LTQP:

1. **Data-driven**: attacks are targeted at the data that is being processed during query execution.
1. **Query-driven**: attacks are targeted at the query that will be used as input to the engine before query execution.

We consider the query-driven threats to LTQP similar to existing RDF query processing as discussed in [](#related-work-rdf-query-processing),
and we therefore purely focus on data-driven threats for the remainder of this work.

{:.todo}
We could still have a separate section of query-driven threats,
but I suspect it to be much shorter and less interesting than this one.

Inspired by security vulnerabilities of [Web browsers](#related-work-security-vulnerabilities),
we decompose data-driven security threats into the following categories:

1. **Result Compromise**: Threats that lead to incorrect query results.
2. **Malicious Linking**: Threats that cause links to be followed that cause unintended actions to be performed.
3. **System Compromise**: Threats that cause the query engine to perform undesired operations.

Hereafter, we list several security threats across each of these categories.
For each threat, we first describe the _issue_,
then we illustrate it with an _example_,
after which we sketch possibly solutions for _mitigation_.

In these listings, we do not make any assumptions about specific forms or semantics of LTQP, unless otherwise mentioned.
The only general assumption we make is that we have an LTQP query engine that follows links in any way,
and executes queries over the union of the discovered documents.

### Result Compromise

#### Unauthorized Statements

A consequence of the [open world assumption](cite:cites owa) where anyone can say anything about anything,
is that **both valid and invalid (and possibly malicious) things can be said**.
When a query engine is traversing over the Web,
it is therefore possible that it can encounter information **impacts the query results in an undesired manner**.
This information could be [_untrusted_](cite:cites guidedlinktraversal), _contradicting_, or _incorrect_.

Given our use case, Carol could for instance decide to add one additional triple to her profile,
such as: `<https://bob.pods.org/profile#me> :name "Dave"`.
She would therefore indicate that Bob's name is "Dave".
This is obviously false, but she is "allowed" to state this under the open world assumption.
However, this means that if Alice would naively query for all her friend's names via LTQP,
she would have two names for Bob appear in her results,
namely "Bob" and "Dave", where this second result may be undesired.

One solution to this threat [has already been proposed](cite:cites guidedlinktraversal),
whereby the concept of _Content Policies_ are introduced.
These **policies can capture the notion of what one considers authoritative**,
which can vary between different people or agents.
In our example, Alice could for example decide to only trust her contacts to make statements about themselves,
and exclude all other information they express during query processing.
Such a policy would enable Alice's query for contact names to not produce Carol's false name for Bob.
This concept of Content Policies does however only exist in theory,
so no concrete mitigation to this threat exist yet.

### Malicious Linking

#### Intermediate Result and Query Leakage

This threat assumes the existence of a **_hybrid_ LTQP query engine** that primarily traverses links,
but can exploit database-oriented interfaces such as SPARQL endpoints if they are detected.
Query engines typically decompose queries into smaller sub-queries,
and join these intermediate results together afterwards.
In the case of a hybrid LTQP engine,
intermediate results that are obtained from the traversal process
could be joined with data from a discovered SPARQL endpoint.
An attacker could therefore setup an interface that acts as a SPARQL endpoint,
but is in fact a **malicious interface that intercepts intermediate results** from LTQP engines.

Based on our use case, Carol could include a triple with a link to a SPARQL endpoint `http://attacker.com/sparql`.
If Alice makes use of a hybrid LTQP engine, the internal query planner could decide to make use of this malicious endpoint
once it has been discovered.
Depending on the query planner, this could mean that intermediate results from the traversal process such as Bob's telephone 
are used as input to the malicious SPARQL endpoint.
Other query planning algorithms could even decide to send the full original SPARQL query into the malicious endpoint.
This could give the attacker knowledge of intermediate results, or even the full query.
This threat enables attackers to do obtain insights to user behaviour, which is a privacy concern.
A more critical problem is when private data is being leaked that normally exists behind access control, such as bank account numbers.

As this threat is similar to the _cross domain compromise_ threat in Web browsers,
a possible solution to it would be in the form of the **_same-origin policy_** that is being employed in most of today's Web browsers.
In essence, this would mean that intermediate results can not be used across different Fully Qualified Domain Names (FQDN).
Such a solution would have to be carefully designed as to not lead to significant querying restrictions that would lead to fewer results.
A mechanism in the form of [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}
could be used as a workaround to explicitly allow intermediate result sharing from one a domain to another,
which could be called **Cross-Origin Intermediate Result Sharing (COIRS)**.

#### Session Hijacking

In this threat, we assume the pressence of some form of authentication,
–such as [WebID-OIDC](cite:cites spec:webidoidc)– that leads to an active authenticated session.
This threat is identical to that of Web browsers,
where the session token can be compromised through theft or session token prediction.
Such a treat could lead to [cross-domain request forgery (CSRF)](cite:cites csrf) attacks,
where an **attacker forces the user to perform an action while authenticated** without the user's consent.

For example, we assume that Alice has a naive SPARQL endpoint running at `http://my-sparql-endpoint/`,
which requires Alice's session for accepting read and write queries.
Alice's query engine may have Alice's session stored by default for when she wants to query against her own endpoint.
If Carol knows this, she could a malicious triple with a link to `http://my-sparql-endpoint?query=DELETE * WHERE { ?s ?p ?o }` in her profile.
While the [SPARQL protocol](cite:cites spec:sparqlprot) only allows update queries via HTTP POST,
Alice's naive implementation could implement this incorrectly so that update queries are also accepted via HTTP GET.
If Alice executes a query over her address book, the query engine could dereference this link
with her session enabled, which would cause her endpoint to be cleared.
This threat is however not specific to SPARQL endpoints,
but may occur on any type of Web API that allows modifying data via HTTP GET requests.

This threat should be tackled on different fronts, and primarily requires secure and correct software implementations.
First, it is important that authentication-enabled **query engines do not leak sessions across different domains**.
Second, **traversal should only be allowed using the HTTP GET method**.
This may not always be straightfoward,
as hypermedia vocabularies such as [Hydra](cite:cites hydracore)
allow specifying the HTTP method that is to be used when accessing a Web API (e.g. `hydra:method`).
Given an unsecure query engine implementation, such HTTP method declarations could be exploited.
Third, **Web APIs must strictly follow the read-only semantics of HTTP GET**,
which is [not always followed by many Web APIs](cite:cites restful),
either intentional or due to software bugs.

#### Local Linking

Links to URLs on localhost, such as `http://localhost:1234/changePassword=123&callback=http://attacker.com/`.
{:.todo}

Links to URLs on LAN, such as `http://192.168.0.1/router/open-port=*`.
{:.todo}

Links to URLs using the file scheme, such as `file:///etc/passwd`.
{:.todo}

#### Cross-Site Data Injection

Assuming a compromised API (`http://trusted.org/`) that is trusted by the user, but dynamically creates RDF responses based on URL parameters.
A malicious link to `http://trusted.org/?name=Bob". <> rdfs:seeAlso <http://hacker.com/getSessionIds>. <> foaf:name "abc`
could result in data injection and the query engine to follow a link to `http://hacker.com/getSessionIds`,
which may allow the attacker to steal the user's session id's.
{:.todo}

This is related to CORS
{:.todo}

### System Compromise

#### Arbitrary Code Execution

e.g. JavaScript, which would be needed for dynamic HTML pages that inject JSON-LD, also leads to XSS
{:.todo}

#### Link Cycles

Links that cause a traversal cycle.
{:.todo}

#### System hogging

CPU/memory hogging, for example due to specifics of RDF serializations (long URLs, flaws in parsers, ...).
Or infinitely long documents
{:.todo}

#### Data Corruption

Crash due to syntax errors in documents
{:.todo}

#### Cross-query execution interaction

Potentially malicious interactions between parallel query executions.
For example, corruption or exploitation of shared caches.
{:.todo}

Discuss the isolation model from [3] in [](cite:cites securitymodernwebbrowserarchitecture) that can be adapted to querying?
{:.todo}

#### Lookup Priority Modification

Attacks that change scores like pagerank (impact on walking without a map)
Like [here](cite:cites crawlerattacks).
{:.todo}
