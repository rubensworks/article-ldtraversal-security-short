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
Furthermore, local files could be accessed via the `file://` scheme,
which may also include sensitive data.

As this threat is similar to the _cross domain compromise_ and _data theft_ threats in Web browsers.
A possible solution to it would be in the form of the **_same-origin policy_** that is being employed in most of today's Web browsers.
In essence, this would mean that intermediate results can not be used across different Fully Qualified Domain Names (FQDN).
Such a solution would have to be carefully designed as to not lead to significant querying restrictions that would lead to fewer results.
A mechanism in the form of [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}
could be used as a workaround to explicitly allow intermediate result sharing from one a domain to another,
which could be called **Cross-Origin Intermediate Result Sharing (COIRS)**.
Another low-hanging fruit solution would be to block all requests to URLs with the `file://` scheme,
unless explicitly enabled by the user, which is the approach followed by the [Comunica query engine](cite:cites comunica).

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
This may not always be straightforward,
as hypermedia vocabularies such as [Hydra](cite:cites hydracore)
allow specifying the HTTP method that is to be used when accessing a Web API (e.g. `hydra:method`).
Given an unsecure query engine implementation, such HTTP method declarations could be exploited.
Third, **Web APIs must strictly follow the read-only semantics of HTTP GET**,
which is [not always followed by many Web APIs](cite:cites restful),
either intentional or due to software bugs.

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

Advanced crawlers such as the [Googlebot](cite:cites googlebot) allow JavaScript logic to be executed for a limit duration,
since certain HTML pages are built dynamically via JavaScript at the client-side.
In this threat, we assume a similar situation for LTQP,
where Linked Data pages may also be built client-side via a programming language such as JavaScript.
This may in fact already occur on **HTML pages that dynamically produce JSON-LD script tags or RDFa in HTML via JavaScript**.
In order to query over such dynamic Linked Data pages,
a query engine must initiate a process similar to Googlebot's JavaScript execution phase.
Such a process does however open the door to potentially major security vulnerabilities
if **malicious code** is being traversed and **executed by the query engine**.

For example, we assume a LTQP query engine that executed JavaScript on HTML pages before extracting its RDFa and JSON-LD.
Furthermore, this LTQP engine has a security flaw that allows executed JavaScript code to access and manipulate the local file system.
A malicious publisher could execute JavaScript logic that makes use of this flaw to upload all files on the local file system
to the attacker, and deletes all files afterwards so that it can hold the data for ransom.

One of the problems Google Chrome developers focus on is *reducing vulnerability severity*,
which involves running logic inside one or more sandboxes to reduce the chance of software bugs
to lead to access to more critical higher-level software APIs.
While software bugs are nearly impossible to avoid in real-world software,
a similar sandboxing approach should help mitigating the severity of attacks involving arbitrary code execution.
Such a **sandbox would only allow certain operations to be performed**,
which would not include access to the local file system.
If this sandbox would also supports performing HTTP requests,
then the _same-origin policy_ should also be employed to mitigate the risk of cross-site scripting (XSS) attacks.

#### Link Cycles

LTQP by nature depends on the ability of iteratively following links between documents.
It is however possible that such **link structures form cycles**, either intentional or unintentional.
Given this reality, LTQP query engines must be able to detect such cycles.
Otherwise, the query engine could keep following the same links,
and **result in an infinite loop**.
This could cause the query engine to never terminate,
and possibly even produce infinite results.

Link cycles could be formed in different ways.
First, at **application-level**, a document A could contain a link path to document B,
and document B could contain a link path back to document A.
Second, at **HTTP protocol-level,** a server could return for URL A an (HTTP 3xx) redirect chain to URL B,
and document B could contain a redirect chain back to URL A.
Third, at application level, a cycle structure could be **simulated via virtual pages** that always link back to similar pages, but with a different URL.
For example, the [Linked Open Numbers](cite:cites linkedopennumbers) project generates an infinite virtual sequence of natural numbers,
which could produce in an infinite loop when traversed by an LTQP query engine.

Problems with first and second form of link cycles could be mitigated by letting the query engine **keep a history of all followed URLs**,
and ignore dereferencing a URL that has already been passed before.
The third form of link cycle makes use of distinct URLs,
so this first mitigation would not be effective.
An alternative approach that would mitigate this third form –and also the first two forms at a reduced level of efficiency–,
is to **place a limit on the link path length from a given seed document**.
For example, querying from page 0 in the Linked Open Number project with a link path limit of 100 would cause the query engine not to go past page 100.
This is the approach that is employed by the recommended [JSON-LD 1.1 processing algorithm](cite:cites spec:jsonldapi)
for handling recursive `@context` references.
Different link path limit values could be applicable for different use cases,
so query engines could consider making this value configurable for the user.

#### System hogging

The _user interface compromise_ treat for Web browsers includes attacks involving **CPU and memory hogging**
through (direct or indirect) **malicious code execution** or by **exploiting software flaws**.
Such threats also exist for LTQP query engines,
especially regarding the use of different **RDF serializations**,
and their particularities with respect to parsing.

For example, RDF serializations such as [Turtle](cite:cites spec:turtle) and [JSON-LD](cite:cites spec:jsonld) place **no limits on their document sizes**.
JSON-LD even explicitly allows this through its [Streaming JSON-LD note](cite:cites spec:jsonldstreaming).
This allows malicious publishers to create infinitely long documents that are streamed to query engines,
and can lead to CPU and memory issues.
Furthermore, similar issues can occur due to very long or **infinite IRIs or literals** inside documents.
Other attacks could exist that specifically target known **flaws in RDF parsers** that cause CPU or memory issues.

Even though this is typically omitted from RDF format specifications,
implementations often **place certain limits** on maximum document, IRI and literal lengths.
For instance, [SAX parsers](cite:cites sax) typically put a limit of 1 megabyte on IRIs and literals,
and provide the option to increase this limit when certain documents would exceed this threshold.
Applying the approach of ***sandboxing*** on RDF parsers would also help mitigating such attacks,
but for example placing a time and memory limit on the parsing of a document.

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
