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
is that both valid and invalid (and possibly malicious) things can be said.
When a query engine is traversing over the Web,
it is therefore possible that it can encounter information negatively impacts the query results.
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
These policies can capture the notion of what one considers authoritative,
which can vary between different people or agents.
In our example, Alice could for example decide to only trust her contacts to make statements about themselves,
and exclude all other information they express during query processing.
Such a policy would enable Alice's query for contact names to not produce Carol's false name for Bob.
This concept of Content Policies does however only exist in theory,
so no concrete mitigation to this threat exist yet.

### Malicious Linking

#### Intermediate Result and Query Leakage

#### Session Hijacking

Link to `http://my-sparql-endpoint?query=DELETE * WHERE { ?s ?p ?o }`, assuming that the user is actively authenticated for that endpoint during query execution.
{:.todo}

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
