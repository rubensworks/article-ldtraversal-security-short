## Data-driven Threats
{:#threats}

We consider two main types of security threats to LTQP engines:
<span class="comment" data-author="RV">Why, do we have any precedents for this? Is this what other query literature does? Are we being exhaustive (to the best of our knowledge), or did we just pick two. Is this driven from related work?</span>

1. **Data-driven**: attacks are targeted at the data that is being processed during query execution.
1. **Query-driven**: attacks are targeted at the query that will be used as input to the engine before query execution.

<span class="comment" data-author="RV">The phrasing <q>targeted at</q> is incorrect, I think. The data is not the target of the attack, but the means of attacking (another target). So on the one hand, we want to change the phrasing; on the other hand, I wonder whether we also need to have a separate list of actual targets. Who are we attacking here?</span>

We consider the query-driven threats to LTQP similar to existing RDF query processing as discussed in [](#related-work-rdf-query-processing),
<span class="comment" data-author="RV">Can we make an argument here as to why? Perhaps we still want to enumerate each of those, and explain a) whether they apply b) how they apply differently.</span>
and we therefore purely focus on data-driven threats for the remainder of this work.

{:.todo}
We could still have a separate section of query-driven threats,
but I suspect it to be much shorter and less interesting than this one.

<span class="rephrase" data-author="RV">Inspired by</span>
<span class="comment" data-author="RV">Perhaps more something along <q>We follow the categorization of XXX to define/organize…</q></span>
known security vulnerabilities of [Web browsers and crawlers](#related-work),
we decompose data-driven security threats into the following axes:

1. **Result Compromise**: attacks that lead <ins class="comment" data-author="RV">the client to return</ins> to incorrect query results.
<span class="comment" data-author="RV">So here the client, or the one using the client, seems to be the target.</span>
2. **Malicious Linking**: attacks that cause links to be followed that cause <ins class="comment" data-author="RV">the client to perform</ins> unintended actions <del class="comment" data-author="RV">to be performed</del>.
<span class="comment" data-author="RV">Interesting point here as to attacker/attacked/vehicle: it's an attacker taking control of a client, which then serves as an unwitting vehicle to attack a third party</span>
3. **System Compromise**: attacks that cause the query engine to perform undesired operations.
<span class="comment" data-author="RV">As currently written, 3 is a superset containing 1 and 2. What is the difference? Or is <q>other</q> missing?</span>

[](#threats-overview) gives an overview of all threats that we consider in this article,
and to what threat axes they apply.

<figure id="threats-overview" class="table" markdown="1">

| Threat                                | Result Compromise | Malicious Linking | System Compromise |
|---------------------------------------|-------------------|-------------------|-------------------|
| Unauthorized Statements 				| ✓                 |                   |                   |
| Intermediate Result and Query Leakage |                   | ✓                 |                   |
| Session Hijacking 				    |                   | ✓                 |                   |
| Cross-site Data Injection     	    | ✓                 | ✓                 |                   |
| Arbitrary Code Execution 				|                   |                   | ✓                 |
| Link Cycles            				|                   | ✓                 | ✓                 |
| System hogging        				|                   |                   | ✓                 |
| Document Corruption.                  |                   |                   | ✓                 |
| Cross-query Execution Interaction		|                   |                   | ✓                 |
| Document Priority Modification.  		|                   |                   | ✓                 |

<figcaption markdown="block">
An overview of all threats related to LTQP that are considered in this article.
They are decomposed into the different threat axes to which they apply.
</figcaption>
</figure>

Hereafter, we explain each threat in more detail.
For each threat, we first describe the _issue_,
then we illustrate it with at least one _example_,
after which we sketch possible solutions for _mitigation_.

Unless mentioned otherwise, we do not make any assumptions about specific forms or semantics of LTQP,
which can influence which links are considered.
The only general assumption we make is that we have an LTQP query engine that follows links in any way,
and executes queries over the union of the discovered documents.

### Unauthorized Statements
{:#threat-unauthorized-statements}

<span class="comment" data-author="RV">I don't think we need the bolding here</span>

<span class="comment" data-author="RV">I think the title is a misnomer; it makes me think about (client) authorization, so whether the client was allowed to see it. Whereas this is about the statements being <em>untrusted</em>—because they were not issued by an authority. I think you mean <q>Unauthoritative</q>.</span>

<span class="comment" data-author="RV">So, reading the first case here, I think there are a couple of structured elements we might want to add to every case. I have added some examples.</span>

A consequence of the [open-world assumption](cite:cites owa) where anyone can say anything about anything,
is that **both valid and invalid (and possibly malicious) things can be said**.
When a query engine is traversing the Web,
it is therefore possible that it can encounter information that **impacts the query results in an undesired manner**.
This information could be [_untrusted_](cite:cites guidedlinktraversal), _contradicting_, or _incorrect_.

Attacker
: Any data publisher

Victim
: Result processor

Cause
: Mistaken authority

Consequence/impact
: Untrusted query results

Difficulty of attack
: Easy

Difficulty of mitigation
: Currently hard

**Example**

Given our use case, Carol could for instance decide to add one additional triple to her profile,
such as: `<https://bob.pods.org/profile#me> :name "Dave"`.
She would therefore indicate that Bob's name is "Dave".
This is obviously false, but she is "allowed" to state this under the open world assumption.
However, this means that if Alice would naively query for all her friend's names via LTQP,
she would have two names for Bob appear in her results,
namely "Bob" and "Dave", where this second result may be undesired.

<span class="comment" data-author="RV">This made me think of another attack, where very long, possibly endless (!) data documents are served. Like 1GB literals or so. The victim there would be the query engine, which would run out of memory. Different mitigation as well. So I think this is a different attack.</span>

**Mitigation**

One solution to this threat [has been proposed](cite:cites guidedlinktraversal),
whereby the concept of _Content Policies_ are introduced.
These **policies can capture the notion of what one considers authoritative**,
which can vary between different people or agents.
In our example, Alice could for example decide to only trust her contacts to make statements about themselves,
and exclude all other information they express during query processing.
Such a policy would enable Alice's query for contact names to not produce Carol's false name for Bob.
This concept of Content Policies does however only exist in theory,
so no concrete mitigation to this threat exist yet.

<span class="comment" data-author="RV">Perhaps reason about the consequences? As such, LTQP currenty cannot deliver trusted results?</span>

<span class="comment" data-author="RV">Another mitigation is provenance: include the results, but track where they come from. I wonder whether Olaf has already written about this, given that he has worked on both LTQP and PROV. I think you can describe this as: one direction is limiting what information is incorporated from what sources, another is documenting thee sources information came from.</span>

### Intermediate Result and Query Leakage
{:#threat-intermediate-leakage}

This threat assumes the existence of a **_hybrid_ LTQP query engine** that primarily traverses links,
but can exploit database-oriented interfaces such as SPARQL endpoints if they are detected in favour of a range of documents.
Query engines typically decompose queries into smaller sub-queries,
and join these intermediate results together afterwards.
In the case of a hybrid LTQP engine,
intermediate results that are obtained from the traversal process
could be joined with data from a discovered SPARQL endpoint.
An attacker could therefore setup an interface that acts as a SPARQL endpoint,
but is in fact a **malicious interface that intercepts intermediate results** from LTQP engines.

**Example**

Based on our use case, Carol could include a triple with a link to the SPARQL endpoint at `http://attacker.com/sparql`.
If Alice makes use of a hybrid LTQP engine, the internal query planner could decide to make use of this malicious endpoint
once it has been discovered.
Depending on the query planner, this could mean that intermediate results from the traversal process such as Bob's telephone 
are used as input to the malicious SPARQL endpoint.
Other query planning algorithms could even decide to send the full original SPARQL query into the malicious endpoint.
Depending on the engine and its query plan,
this could give the attacker knowledge of intermediate results,
or even the full query.
This threat enables attackers to do obtain insights to user behaviour, which is a privacy concern.
A more critical problem is when private data is being leaked that normally exists behind access control, such as bank account numbers.

**Mitigation**

As this threat is similar to the _cross domain compromise_ and _data theft_ threats in Web browsers.
A possible solution to it would be in the form of the **_same-origin policy_** that is being employed in most of today's Web browsers.
In essence, this would mean that intermediate results can not be used across different Fully Qualified Domain Names (FQDN).
Such a solution would have to be carefully designed as to not lead to significant querying restrictions that would lead to fewer relevant query results.
A mechanism in the form of [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}
could be used as a workaround to explicitly allow intermediate result sharing from one a domain to another,
which could be called **Cross-Origin Intermediate Result Sharing (COIRS)**.
Such a workaround should be designed carefully, as not to suffer from the same [issues as CORS](cite:cites studyofcors).
Related to this, just like Web browsers, query engines may provide queryable access to local files using the `file://` scheme.
Web browsers typically block requests to these from remote locations due to their sensitive nature.
Similarly, query engines may decide to also block requests to URLs using the `file://` scheme,
unless explicitly enabled by the user.
This approach is for example followed by the [Comunica query engine](cite:cites comunica).

### Session Hijacking
{:#threat-session-hijacking}

In this threat, we assume the pressence of some form of authentication,
–such as [WebID-OIDC](cite:cites spec:webidoidc)– that leads to an active authenticated session.
This threat is similar to that of Web browsers,
where the session token can be compromised through theft or session token prediction.
Such a treat could lead to [cross-domain request forgery (CSRF)](cite:cites csrf) attacks,
where an **attacker forces the user to perform an action while authenticated** without the user's consent.

**Example**

For example, we assume that Alice has a naive SPARQL endpoint running at `http://my-endpoint.com/sparql`,
which requires Alice's session for accepting read and write queries.
Alice's query engine may have Alice's session stored by default for when she wants to query against her own endpoint.
If Carol knows this, she could a malicious triple with a link to `http://my-endpoint.com/sparql?query=DELETE * WHERE { ?s ?p ?o }` in her profile.
While the [SPARQL protocol](cite:cites spec:sparqlprot) only allows update queries via HTTP POST,
Alice's naive query engine could implement this incorrectly so that update queries are also accepted via HTTP GET.
If Alice executes a query over her address book, the query engine could dereference this link
with her session enabled, which would cause her endpoint to be cleared.
This threat is however not specific to SPARQL endpoints,
but may occur on any type of Web API that allows modifying data via HTTP GET requests.

**Mitigation**

This threat should be tackled on different fronts, and primarily requires secure and well-tested software implementations.
First, it is important that authentication-enabled **query engines do not leak sessions across different domains**.
Second, **traversal should only be allowed using the HTTP GET method**.
This may not always be straightforward,
as hypermedia vocabularies such as [Hydra](cite:cites hydracore)
allow specifying the HTTP method that is to be used when accessing a Web API (e.g. `hydra:method`).
Given an unsecure query engine implementation, such HTTP method declarations could be exploited.
Third, **Web APIs must strictly follow the read-only semantics of HTTP GET**,
which is [not always followed by many Web APIs](cite:cites restful),
either intentional or due to software bugs.

### Cross-site Data Injection
{:#threat-cross-site-injection}

This threat concerns ways by which **attackers can inject data or links into documents**.
For instance, **HTTP GET parameters** are often used to **parameterize the contents of documents**.
If such parameters are not properly validated or escaped,
they can be used by attackers to include malicious data or links.

**Example**

For example, assuming Alice executes a query over a page from Carol,
and a compromised API `http://trusted.org/?name` that dynamically creates RDF responses based on the `?name` HTTP GET parameters.
In this case, the API simply has a Turtle document template into which the name is filled in as a literal value,
but it does not do any escaping.
We assume Alice decides to fully trust all links from `http://trusted.org/` to other pages,
but only trust information directly on Carol's page or links to other trusted domains.
If Carol includes a link to `<http://trusted.org/?name=Bob". <> rdfs:seeAlso <http://hacker.com/invalid-data>. <> foaf:name "abc>`,
then this would cause the API to produce a Turtle document that contains a link to `http://hacker.com/invalid-data`,
which would lead to unwanted data to be included in the query results.

**Mitigation**

No single technique can fully mitigate this threat.
Just like [SQL injection attacks](cite:cites sqlinjection) on Web sites,
**Web APIs should take care of input validation**, preferably via reusable and rigorously tested software libraries.
On the side of query engines, this threat may partially mitigated by **carefully designing trust policies**.
In the case of our example, defining a policy that enables the full range of (direct and indirect) links
to be followed from a single domain can be considered unsafe.
Instead, more restrictive policies may be enforced, at the cost of expressivity and flexibility.

### Arbitrary Code Execution
{:#threat-arbitrary-code-exec}

Advanced crawlers such as the [Googlebot](cite:cites googlebot) allow JavaScript logic to be executed for a limit duration,
since certain HTML pages are built dynamically via JavaScript at the client-side.
In this threat, we assume a similar situation for LTQP,
where Linked Data pages may also be created client-side via an expressive programming language such as JavaScript.
This would in fact already be applicable to **HTML pages that dynamically produce JSON-LD script tags or RDFa in HTML via JavaScript**.
In order to query over such dynamic Linked Data pages,
a query engine must initiate a process similar to Googlebot's JavaScript execution phase.
Such a process does however open the door to potentially major security vulnerabilities
if **malicious code** is being read and **executed by the query engine** during traversal.

**Example**

For example, we assume that Alice's LTQP query engine executes JavaScript on HTML pages before extracting its RDFa and JSON-LD.
Furthermore, this LTQP engine has a security flaw that allows executed JavaScript code to access and manipulate the local file system.
Carol could include a malicious piece of JavaScript code in her profile that makes use of this flaw to upload all files on the local file system
to the attacker, and deletes all files afterwards so that she can hold Alice's data for ransom.

**Mitigation**

One of the problems Google Chrome developers focus on is *reducing vulnerability severity*,
which involves running logic inside one or more sandboxes to reduce the chance of software bugs
to lead to access to more critical higher-level software APIs.
While software bugs are nearly impossible to avoid in real-world software,
a similar sandboxing approach helps reducing the severity of attacks involving arbitrary code execution.
Such a **sandbox would only allow certain operations to be performed**,
which would not include access to the local file system.
If this sandbox would also support performing HTTP requests,
then the _same-origin policy_ should also be employed to mitigate the risk of cross-site scripting (XSS) attacks.

### Link Cycles
{:#threat-link-cycles}

LTQP by nature depends on the ability of iteratively following links between documents.
It is however possible that such **link structures form cycles**, either intentional or unintentional,
just like crawler traps.
Given this reality, LTQP query engines must be able to detect such cycles.
Otherwise, the query engine could keep following the same links,
and **result in an infinite loop**.
This could cause the query engine to never terminate,
and possibly even produce infinite results.

**Example**

Link cycles could be formed in different ways.
First, at **application-level**, Carol's profile could contain a link path to document X,
and document X could contain a link path back to Carol's profile.
Second, at **HTTP protocol-level,** Carol's server could return for her profile's URL an (HTTP 3xx) redirect chain to URL X,
and URL X could contain a redirect chain back to the URL of her profile.
Third, at application level, a cycle structure could be **simulated via virtual pages** that always link back to similar pages, but with a different URL.
For example, the [Linked Open Numbers](cite:cites linkedopennumbers) project generates an infinite virtual sequence of natural numbers,
which could produce in an infinite loop when traversed by an LTQP query engine.

**Mitigation**

Problems with first and second form of link cycles could be mitigated by letting the query engine **keep a history of all followed URLs**,
and not dereference a URL that has already been passed before.
The third form of link cycle makes use of distinct URLs,
so this first mitigation would not be effective.
An alternative approach that would mitigate this third form –and also the first two forms at a reduced level of efficiency–,
is to **place a limit on the link path length from a given seed document**.
For example, querying from page 0 in the Linked Open Number project with a link path limit of 100 would cause the query engine not to go past page 100.
This is the approach that is employed by the recommended [JSON-LD 1.1 processing algorithm](cite:cites spec:jsonldapi)
for handling recursive `@context` references in JSON-LD documents.
Different link path limit values could be applicable for different use cases,
so query engines could consider making this value configurable for the user.
Other more advanced techniques from the domain of crawler trap mitigation could be extended,
such as [the one that measures similarities between documents to detect crawler traps](cite:cites crawlertrapsdetection).

### System hogging
{:#threat-system-hogging}

The _user interface compromise_ treat for Web browsers includes attacks involving **CPU and memory hogging**
through (direct or indirect) **malicious code execution** or by **exploiting software flaws**.
Such threats also exist for LTQP query engines,
especially regarding the use of different **RDF serializations**,
and their particularities with respect to parsing.

**Example**

For example, RDF serializations such as [Turtle](cite:cites spec:turtle) and [JSON-LD](cite:cites spec:jsonld) place **no limits on their document sizes**.
JSON-LD even explicitly allows this through its [Streaming JSON-LD note](cite:cites spec:jsonldstreaming).
This allows malicious publishers to create infinitely long documents that are streamed to query engines,
and can lead to CPU and memory issues.
Furthermore, similar issues can occur due to very long or **infinite IRIs or literals** inside documents.
Other attacks could exist that specifically target known **flaws in RDF parsers** that cause CPU or memory issues.

**Mitigation**

Even though it is typically omitted from RDF format specifications,
implementations often **place certain limits** on maximum document, IRI and literal lengths.
For instance, [SAX parsers](cite:cites sax) typically put a limit of 1 megabyte on IRIs and literals,
and provide the option to increase this limit when certain documents would exceed this threshold.
Applying the approach of **sandboxing** on RDF parsers would also help mitigating such attacks,
but for example placing a time and memory limit on the parsing of a document.
If LTQP engines would allow arbitrary code execution, then more extensive system hogging mitigations
would be needed just like in [Web browsers](cite:cites securitymodernwebbrowserarchitecture).

### Document Corruption
{:#threat-document-corruption}

Since the Web is not a centrally controlled system,
it is possible that documents are **incorrectly formatted**,
either intentional or unintentional.
RDF formats typically prescribe a restrictive syntax,
which require parsers to emit an error when it encounters illegal syntax.
When an LTQP engine discovers and parses a large number of RDF documents,
possibly in an uncontrolled manner,
it is undesired that a syntax error in just a single RDF document can cause
the whole **query process to terminate with an error**.
Furthermore, **links may go dead** (HTTP 404) at any point in time,
while finding a link to a URL that produces a 404 response should also not cause the query engine to terminate.

**Example**

For example, Carol could decide to introduce a syntax error in her profile document,
or she could simply remove it to produce a 404 response.
This would could cause Alice's queries over her friends from that point on to fail.

**Mitigation**

The **sandbox** approach is well-suited for handling these types of attacks.
RDF parsing for each document can run in a sandbox,
where errors in this document would simply cause parsing of this document to end without crashing the query engine.
Optionally, a **warning** could be emitted to the user.
The same approach could be followed for HTTP errors on the protocol level, such as HTTP 404's.
This approach is followed by the [Comunica query engine](cite:cites comunica) via its [lenient execution mode](https://comunica.dev/docs/query/advanced/context/#4--lenient-execution).
An alternative would be to create more **lenient RDF parsers** that accept syntax errors and attempts to derive the intended meaning,
similar as to how (non-XHTML) HTML parsers are created.
The downside of this is that such parsers would not strictly adhere to their specifications.

### Cross-query Execution Interaction
{:#threat-cross-query-interaction}

Query engines of all forms typically make use of **caching techniques** to improve performance of query execution.
LTQP query engines can leverage caching techniques for document retrieval.
Within a single query execution, or across multiple query executions,
the documents may be reused, which could reduce the overall number of HTTP requests.
Such forms of caching can lead to threats based on **information leaking across different query executions**.
We therefore make the assumption of caching-enabled LTQP engines in this threat.

**Examples**

A first example of this threat is an attack that enables Carol to gain knowledge about
whether or not Bob's profile has been requested before by Alice.
We assume that the Alice's engine issues a query over a document from Carol listing all her pictures.
We also assume that Bob's profile contains a link to Carol's profile.
If Carol includes a link from her pictures document to Bob's profile, and Bob's profile already links to Carol's profile,
then the query engine could fetch these three documents in sequence (Carol's pictures, Bob's profile, Carol's profile).
Since Carol's pictures and profile are in control of Carol,
she could perform a [**timing attack**](cite:cites timingattack) to derive how long the Alice's query engine took to process Bob's profile.
Since HTTP delays typically form the bottleneck in LTQP,
Carol could thereby derive if Bob's profile was fetched from a cache or not.
This would enable Carol to gain knowledge about prior document lookups,
which could for example lead to privacy issues with respect to the user's interests.

A second example assumes the presence of a **software bug** inside Alice's LTQP query engine
that makes document caches ignore authorization information.
This example is also a form of the *Intermediate Result and Query Leakage* threat that was explained before,
for which we assume the existence of a *hybrid* LTQP query engine.
If Alice queries a private file containing her passwords from a server using its authentication key,
this can cause this passwords file to be cached.
If Carol has a query endpoint that is being queried by Alice,
and Carol is aware of the location of Alice's passwords,
then she could maliciously introduce a link to Alice's passwords file.
Even if the query was not executed with Alice's authentication key,
the bug in Alice's query engine would cause the passwords file to be fetched in full from the cache,
which could cause parts of it to be leaked to Carol's query endpoint.

**Mitigation**

In order to mitigate this threat, the [**isolation model** that is used in Web browsers](cite:cites securitymodernwebbrowserarchitecture) could be reused.
When applied to LTQP query engines, this could mean that **each query would be executed in a separate sandbox**,
so that information can not leak across different query executions.
A downside of this approach is that this may cause a significant **performance impact**
when similar queries are executed in sequence, and would cause identical documents to not be reused from the cache anymore.
In order to mitigate this drawback, solutions may be possible to **allow "related queries" to be executed inside the same sandbox**.

### Document Priority Modification
{:#threat-doc-priority-modification}

Different techniques are possible to [determine the priority of documents](cite:cites WalkingWithoutMap) during query processing.
If queries don't specify a custom ordering, this **prioritization will impact the ordering of query results**.
Some of these techniques are purely graph-based, such as [PageRank](cite:cites pagerank), and can therefore suffer from purely data-driven attacks.
This threat involves **attacks that can influence the priority of documents**,
and thereby maliciously influence what query **results come in earlier or later**.

**Example**

One example of such an attack is similar to [the attack to modify priorities within crawlers](cite:cites crawlerattacks).
We assume that Alice issues a query that returns grocery stores in the local area,
which is executed via a LTQP query engine that makes use of PageRank to prioritize documents.
Furthermore, we assume a highly-scoring, but vulnerable API that accepts HTTP GET parameters
that can be abused to inject custom URLs inside the API responses.
If Carol aims to increase the ranking of her grocery store within Alice's query for better visibility,
then she could exploit this vulnerable API.
Concretely, Carol could place links from the grocery store's page to this vulnerable API
using GET parameters that would cause it to link back to Carol's grocery store.
Such an attack would lead to a higher PageRank for Carol's grocery store,
and therefore an earlier handling and result representation
of Carol's grocery store.

**Mitigation**

Several mitigations have been [proposed for these types of attacks](cite:cites crawlerattacks).
A first solution is to place **responsibility at the API**, and expecting it to patch the exploit.
A second solution involves publishers to expose **policies that explicitly authorize what links should be considered legitimate**,
and LTQP query engines inspecting these policies when determining document priorities.
A third solution is to use **machine-learning to distinguishing non-legitimate from legitimate links**.
A combination of the three approaches can be used to mitigate this threat.
