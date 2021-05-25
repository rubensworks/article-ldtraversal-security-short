## Data-driven Vulnerabilities
{:#vulnerabilities}

As shown before in [](#related-work-rdf-query-processing),
most research on identifying security vulnerabilities within RDF query processing
focuses on the query itself as a means of attacking, mostly through injection techniques.
Since LTQP engines also accepts queries as input,
these existing techniques will therefore also apply to LTQP engines.

In this work, we acknowledge the importance of these vulnerabilities,
but we instead place our attention onto a new class of vulnerabilities
that are specific to LTQP engines as a consequence of the open and uncontrolled nature of data on the Web.
Concretely, we consider two main classes of security vulnerabilities to LTQP engines:

1. **Query-driven**: vulnerabilities that are caused by modifying queries that are the input to certain query engines.
2. **Data-driven**: vulnerabilities that are caused by the presence, structuring, or method of publishing data on the Web.

To the best of our knowledge, all existing work on security vulnerabilities within RDF query processing
has focused on query-driven vulnerabilities.
Given its importance for LTQP engines,
we purely focus on data-driven vulnerabilities for the remainder of this work.

{:.todo}
We could still have a separate section of query-driven vulnerabilities,
but I suspect it to be much shorter and less interesting than this one.

We identify three main axes for security vulnerabilities, based on their exploit's potential impact area:

1. **Query Results**: vulnerabilities that lead to exploits regarding query results.
1. **Data Integrity**: vulnerabilities that lead to exploits regarding one or more user's data.
1. **Query Process**: vulnerabilities that lead to exploits regarding the stability of the query engine's process.

[](#vulnerabilities-overview) gives an overview of all vulnerabilities that we consider in this article,
and to what vulnerability axes they apply.

<figure id="vulnerabilities-overview" class="table" markdown="1">

| Threat                                | Query Results     | Data Integrity    | Query Process     |
|---------------------------------------|-------------------|-------------------|-------------------|
| Unauthorized Statements 				| ✓                 |                   |                   |
| Intermediate Result and Query Leakage | ✓                 |                   |                   |
| Session Hijacking 				    |                   | ✓                 |                   |
| Cross-site Data Injection     	    | ✓                 | ✓                 |                   |
| Arbitrary Code Execution 				|                   | ✓                 | ✓                 |
| Link Traversal Trap      				|                   |                   | ✓                 |
| System hogging        				|                   |                   | ✓                 |
| Document Corruption                   |                   |                   | ✓                 |
| Cross-query Execution Interaction		|                   | ✓                 |                   |
| Document Priority Modification.  		| ✓                 |                   |                   |

<figcaption markdown="block">
An overview of all vulnerabilities related to LTQP that are considered in this article.
They are decomposed into the different vulnerability axes to which they apply.
</figcaption>
</figure>

Hereafter, we explain and classify each vulnerability using the classification method from [](#classification).
For each vulnerability, we provide at least one possible example of an _exploit_ based on our use case,
and sketch at least one possible _mitigation_.

Unless mentioned otherwise, we do not make any assumptions about specific forms or semantics of LTQP,
which can influence which links are considered.
The only general assumption we make is that we have an LTQP query engine that follows links in any way,
and executes queries over the union of the discovered documents.

### Unauthoritative Statements
{:#vulnerability-unauthorized-statements}

<span class="comment" data-author="RV">I don't think we need the bolding on sentence fragments as applied below</span>

A consequence of the [open-world assumption](cite:cites owa) where anyone can say anything about anything,
is that **both valid and invalid (and possibly malicious) things can be said**.
When a query engine is traversing the Web,
it is therefore possible that it can encounter information that **impacts the query results in an undesired manner**.
This information could be [_untrusted_](cite:cites guidedlinktraversal, linktraversaldiverse), _contradicting_, or _incorrect_.
Without mitigations to this vulnerability, query results from an LTQP can therefore never be really trusted,
which brings the practical broad use of LTQP into question.

**Exploit: producing untrusted query results by adding unauthoritative triples**

Given our use case, Carol could for instance decide to add one additional triple to her profile,
such as: `<https://bob.pods.org/profile#me> :name "Dave"`.
She would therefore indicate that Bob's name is "Dave".
This is obviously false, but she is "allowed" to state this under the open world assumption.
However, this means that if Alice would naively query for all her friend's names via LTQP,
she would have two names for Bob appear in her results,
namely "Bob" and "Dave", where this second result may be undesired.

Attacker
: Data publisher (Carol)

Victim
: Query results from the LTQP engine of Alice

Impact
: Untrusted query results

Difficulty
: Easy (adding triples to an RDF document)

**Mitigation 1: applying content policies**

One solution to this vulnerability [has been proposed](cite:cites guidedlinktraversal),
whereby the concept of _Content Policies_ are introduced.
These **policies can capture the notion of what one considers authoritative**,
which can vary between different people or agents.
In our example, Alice could for example decide to only trust her contacts to make statements about themselves,
and exclude all other information they express during query processing.
Such a policy would enable Alice's query for contact names to not produce Carol's false name for Bob.
This concept of Content Policies does however only exist in theory,
so no concrete mitigation to this vulnerability exist yet.

Location
: Data publishers and LTQP engines

Difficulty
: Currently hard (content policy implementations do not exist yet)

**Mitigation 2: tracking provenance**

Another solution to this vulnerability [has been suggested](cite:cites linktraversaldiverse)
to make use of [data provenance](cite:cites spec:provo).
In contrast to the previous mitigation,
this approach would not limit what is incorporated from what sources,
but instead it would document the sources information came from.
The end-user can then decide afterwards what provenance trails it seems trustworthy.

Location
: LTQP engines

Difficulty
: Medium

### Intermediate Result and Query Leakage
{:#vulnerability-intermediate-leakage}

This vulnerability assumes the existence of a **_hybrid_ LTQP query engine** that primarily traverses links,
but can exploit database-oriented interfaces such as SPARQL endpoints if they are detected in favour of a range of documents.
Furthermore, we assume a range of documents that require authentication,
as their contents are not accessible to everyone.
Query engines typically decompose queries into smaller sub-queries,
and join these intermediate results together afterwards.
In the case of a hybrid LTQP engine,
intermediate results that are obtained from the traversal process from non-public documents
could be joined with data from a discovered SPARQL endpoint.
An attacker could therefore set up an interface that acts as a SPARQL endpoint,
but is in fact a **malicious interface that intercepts intermediate results** from LTQP engines.

**Exploit: capturing intermediary results via malicious SPARQL endpoint**

Based on our use case, Carol could include a triple with a link to the SPARQL endpoint at `http://attacker.com/sparql`.
If Alice makes use of a hybrid LTQP engine with an adaptive query planner, this internal query planner could decide to make use of this malicious endpoint
once it has been discovered.
Depending on the query planner, this could mean that non-public intermediate results from the traversal process such as Bob's telephone 
are used as input to the malicious SPARQL endpoint.
Other query planning algorithms could even decide to send the full original SPARQL query into the malicious endpoint.
Depending on the engine and its query plan,
this could give the attacker knowledge of intermediate results,
or even the full query.
This vulnerability enables attackers to do obtain insights to user behaviour, which is a privacy concern.
A more critical problem is when private data is being leaked that normally exists behind access control, such as bank account numbers.

Attacker
: SPARQL endpoint publisher (Carol)

Victim
: Intermediary results of the LTQP engine of Alice

Impact
: Leakage of (intermediary) query results

Difficulty
: Medium (setting up a malicious SPARQL endpoint)

**Mitigation: Same-origin policy**

As this vulnerability is similar to the _cross-domain compromise_ and _data theft_ vulnerabilities in Web browsers.
<span class="comment" data-author="RV">Refs?</span>
A possible solution to it would be in the form of the **_same-origin policy_** that is being employed in most of today's Web browsers.
In essence, this would mean that intermediate results can not be used across different Fully Qualified Domain Names (FQDN).
Such a solution would have to be carefully designed as to not lead to significant querying restrictions that would lead to fewer relevant query results.
<span class="comment" data-author="RV">Or performance issues?</span>
A mechanism in the form of [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}
could be used as a workaround to explicitly allow intermediate result sharing from one a domain to another.
Such a workaround should be designed carefully, as not to suffer from the same [issues as CORS](cite:cites studyofcors).
Related to this, just like Web browsers, query engines may provide queryable access to local files using the `file://` scheme.
Web browsers typically block requests to these from remote locations due to their sensitive nature.
Similarly, query engines may decide to also block requests to URLs using the `file://` scheme,
unless explicitly enabled by the user.
This approach is for example followed by the [Comunica query engine](cite:cites comunica).

Location
: LTQP engines

Difficulty
: Easy (hard with CORS-like workaround)

### Session Hijacking
{:#vulnerability-session-hijacking}

In this vulnerability, we assume the presence of some form of authentication
(such as [WebID-OIDC](cite:cites spec:webidoidc)) that leads to an active authenticated session.
This vulnerability is similar to that of Web browsers,
where the session token can be compromised through theft or session token prediction.
Such a vulnerability could lead to [cross-domain request forgery (CSRF)](cite:cites csrf) attacks,
where an **attacker forces the user to perform an action while authenticated** without the user's consent.

**Exploit: triggering unintended operations on SPARQL endpoint behind access control**

For example, we assume that Alice has a flawed SPARQL endpoint running at `http://my-endpoint.com/sparql`,
which requires Alice's session for accepting read and write queries.
Alice's query engine may have Alice's session stored by default for when she wants to query against her own endpoint.
If Carol knows this, she could a malicious triple with a link to `http://my-endpoint.com/sparql?query=DELETE * WHERE { ?s ?p ?o }` in her profile.
While the [SPARQL protocol](cite:cites spec:sparqlprot) only allows update queries via HTTP POST,
Alice's flawed query engine could implement this incorrectly so that update queries are also accepted via HTTP GET.
If Alice executes a query over her address book, the query engine could dereference this link
with her session enabled, which would cause her endpoint to be cleared.
This vulnerability is however not specific to SPARQL endpoints,
but may occur on any type of Web API that allows modifying data via HTTP GET requests.

Attacker
: Data publisher (Carol)

Victim
: Alice's stored data

Impact
: Removal or modification of Alice's stored data

Difficulty
: Easy (adding a malicious link to flawed endpoint)

**Mitigation: same-origin policy**

This vulnerability should be tackled on different fronts, and primarily requires secure and well-tested software implementations.
First, it is important that authentication-enabled **query engines do not leak sessions across different origins**.
<span class="comment" data-author="RV">Can you be more concrete here? How do I specifically implement this?</span>

Location
: LTQP engines

Difficulty
: Medium

**Mitigation: only handle HTTP GET during traversal**

A second mitigation is that **traversal should only be allowed using the HTTP GET method**.
This may not always be straightforward,
as hypermedia vocabularies such as [Hydra](cite:cites hydracore)
allow specifying the HTTP method that is to be used when accessing a Web API (e.g. `hydra:method`).
Given an unsecure query engine implementation, such HTTP method declarations could be exploited.

Location
: LTQP engines

Difficulty
: Medium

**Mitigation: adhere to read-only semantics of HTTP GET**

A third mitigation is that **Web APIs must strictly follow the read-only semantics of HTTP GET**,
which is [not always followed by many Web APIs](cite:cites restful),
either intentionally or due to software bugs.

Location
: Data publishers

Difficulty
: Medium

### Cross-site Data Injection
{:#vulnerability-cross-site-injection}

This vulnerability concerns ways by which **attackers can inject data or links into documents**.
For instance, **HTTP GET parameters** are often used to **parameterize the contents of documents**.
If such parameters are not properly validated or escaped,
they can be used by attackers to include malicious data or links.

**Exploit: injecting untrusted links via flawed trusted API**

For example, assuming Alice executes a query over a page from Carol,
and a compromised API `http://trusted.org/?name` that dynamically creates RDF responses based on the `?name` HTTP GET parameters.
In this case, the API simply has a Turtle document template into which the name is filled in as a literal value,
but it does not do any escaping.
We assume Alice decides to fully trust all links from `http://trusted.org/` to other pages,
but only trust information directly on Carol's page or links to other trusted domains.
If Carol includes a link to `<http://trusted.org/?name=Bob". <> rdfs:seeAlso <http://hacker.com/invalid-data>. <> foaf:name "abc""`,
then this would cause the API to produce a Turtle document that contains a link to `http://hacker.com/invalid-data`,
which would lead to unwanted data to be included in the query results.

Attacker
: Data publisher (Carol)

Victim
: Query results from the LTQP engine of Alice

Impact
: Untrusted query results

Difficulty
: Easy (adding triples to an RDF document)

**Mitigation: validate API parameters**

No single technique can fully mitigate this vulnerability.
Just like [SQL injection attacks](cite:cites sqlinjection) on Web sites,
**Web APIs should take care of input validation**, preferably via reusable and rigorously tested software libraries.

Location
: Data publishers

Difficulty
: Medium

**Mitigation: expressive content policies**

On the side of query engines, this vulnerability may partially mitigated by **carefully designing content policies**.
In the case of our example, defining a policy that enables the full range of (direct and indirect) links
to be followed from a single domain can be considered unsafe.
Instead, more restrictive policies may be enforced, at the cost of expressivity and flexibility.

Location
: LTQP engines

Difficulty
: Currently hard (content policies do not exist yet)

### Arbitrary Code Execution
{:#vulnerability-arbitrary-code-exec}

Advanced crawlers such as the [Googlebot](cite:cites googlebot) allow JavaScript logic to be executed for a limit duration,
since certain HTML pages are built dynamically via JavaScript at the client-side.
In this vulnerability, we assume a similar situation for LTQP,
where Linked Data pages may also be created client-side via an expressive programming language such as JavaScript.
This would in fact already be applicable to **HTML pages that dynamically produce JSON-LD script tags or RDFa in HTML via JavaScript**.
In order to query over such dynamic Linked Data pages,
a query engine must initiate a process similar to Googlebot's JavaScript execution phase.
Such a process does however open the door to potentially major security vulnerabilities
if **malicious code** is being read and **executed by the query engine** during traversal.

**Exploit: manipulate local files via overprivileged JavaScript execution**

For example, we assume that Alice's LTQP query engine executes JavaScript on HTML pages before extracting its RDFa and JSON-LD.
Furthermore, this LTQP engine has a security flaw that allows executed JavaScript code to access and manipulate the local file system.
Carol could include a malicious piece of JavaScript code in her profile that makes use of this flaw to upload all files on the local file system
to the attacker, and deletes all files afterwards so that she can hold Alice's data for ransom.

Attacker
: Data publisher (Carol)

Victim
: Files on machine in which Alice's query engine runs

Impact
: Removal or modification of files on Alice's machine

Difficulty
: Easy (adding JavaScript code to a document)

**Mitigation: sandbox code execution**

One of the problems Google Chrome developers focus on is *reducing vulnerability severity*,
which involves running logic inside one or more sandboxes to reduce the chance of software bugs
to lead to access to more critical higher-level software APIs.
While software bugs are nearly impossible to avoid in real-world software,
a similar sandboxing approach helps reducing the severity of attacks involving arbitrary code execution.
Such a **sandbox would only allow certain operations to be performed**,
which would not include access to the local file system.
If this sandbox would also support performing HTTP requests,
then the _same-origin policy_ should also be employed to mitigate the risk of cross-site scripting (XSS) attacks.

Location
: LTQP engines

Difficulty
: Medium

### Link Traversal Trap
{:#vulnerability-link-traversal-trap}

LTQP by nature depends on the ability of iteratively following links between documents.
It is however possible that such **link structures cause infinite traversal paths** and make the traversal engine get trapped,
either intentionally or unintentionally, just like crawler traps.
Given this reality, LTQP query engines must be able to detect such traps.
Otherwise, query engines could never terminate,
and possibly even produce infinite results.

**Exploit: forming a link cycle**

A link cycle is a simple form of link traversal trap that could be formed in different ways.
First, at **application-level**, Carol's profile could contain a link path to document X,
and document X could contain a link path back to Carol's profile.
Second, at **HTTP protocol-level,** Carol's server could return for her profile's URL an (HTTP 3xx) redirect chain to URL X,
and URL X could contain a redirect chain back to the URL of her profile.
Third, at application level, a cycle structure could be **simulated via virtual pages** that always link back to similar pages, but with a different URL.
For example, the [Linked Open Numbers](cite:cites linkedopennumbers) project generates a near-infinite virtual sequence of natural numbers,
which could produce a bottleneck when traversed by an LTQP query engine.
<span class="comment" data-author="RV">LON is actually finite.</span>

Attacker
: Data publisher (Carol)

Victim
: Query process of Alice's query engine

Impact
: Unresponsiveness of Alice's query engine

Difficulty
: Easy

**Mitigation: tracking history of links**

Problems with first and second form of link cycles could be mitigated by letting the query engine **keep a history of all followed URLs**,
and not dereference a URL that has already been passed before.
The third form of link cycle makes use of distinct URLs,
so this first mitigation would not be effective.

Location
: LTQP engines

Difficulty
: Easy

**Mitigation: limit link path length**

<span class="comment" data-author="RV">Note also that HTTP libraries typically limit the number of redirects (the protocol case above)</span>

An alternative approach that would mitigate this third form –and also the first two forms at a reduced level of efficiency–,
is to **place a limit on the link path length from a given seed document**.
For example, querying from page 0 in the Linked Open Number project with a link path limit of 100 would cause the query engine not to go past page 100.
This is the approach that is employed by the recommended [JSON-LD 1.1 processing algorithm](cite:cites spec:jsonldapi)
for handling recursive `@context` references in JSON-LD documents.
Different link path limit values could be applicable for different use cases,
so query engines could consider making this value configurable for the user.

Location
: LTQP engines

Difficulty
: Easy

**Mitigation: measuring document similarity**

Other more advanced mitigation techniques from the domain of crawler trap mitigation could be extended,
such as [the one that measures similarities between documents to detect crawler traps](cite:cites crawlertrapsdetection).
<span class="comment" data-author="RV">Although I can just fill my document with crap (perhaps make links to next)?</span>

Location
: LTQP engines

Difficulty
: Hard

### System hogging
{:#vulnerability-system-hogging}

The _user interface compromise_ vulnerability for Web browsers includes attacks involving **CPU and memory hogging**
through (direct or indirect) **malicious code execution** or by **exploiting software flaws**.
Such vulnerabilities also exist for LTQP query engines,
especially regarding the use of different **RDF serializations**,
and their particularities with respect to parsing.

**Exploit: producing infinite RDF documents**

For example, RDF serializations such as [Turtle](cite:cites spec:turtle) and [JSON-LD](cite:cites spec:jsonld) place **no limits on their document sizes**.
<span class="comment" data-author="RV">No serialization does? I don't understand.</span>
<span class="comment" data-author="RV">I think the point is rather that several formats were designed for streaming parsing (JSON-LD being indeed explicit, but the others have it in their DNA; notably the line-based N-Triples).</span>
JSON-LD even explicitly allows this through its [Streaming JSON-LD note](cite:cites spec:jsonldstreaming).
<span class="rephrase" data-author="RV">This allows malicious publishers to create infinitely long documents</span> that are streamed to query engines,
<span class="comment" data-author="RV">More precisely, publishers can always generated infinite documents! It's whether or not a client would be tempted to start working on them that matters</span>
and can lead to CPU and memory issues.
Furthermore, similar issues can occur due to very long or **infinite IRIs or literals** inside documents.
Other attacks could exist that specifically target known **flaws in RDF parsers** that cause CPU or memory issues.

Attacker
: Data publisher (Carol)

Victim
: Machine in which Alice's query engine runs

Impact
: Unresponsiveness or crashing of Alice's query engine or machine

Difficulty
: Easy

**Mitigation: placing limits for RDF syntaxes**

Even though typically omitted from RDF format specifications,
implementations often **place certain limits** on maximum document, IRI and literal lengths.
For instance, [SAX parsers](cite:cites sax) typically put a limit of 1 megabyte on IRIs and literals,
and provide the option to increase this limit when certain documents would exceed this threshold.

Location
: LTQP engines

Difficulty
: Medium (identifying all possible limits may not be trivial)

**Mitigation: sandbox RDF parsing**

Applying the approach of **sandboxing** on RDF parsers would also help mitigate such attacks,
by for example placing a time and memory limit on the parsing of a document.
If LTQP engines would allow arbitrary code execution, then more extensive system hogging mitigations
would be needed just like in [Web browsers](cite:cites securitymodernwebbrowserarchitecture).

Location
: LTQP engines

Difficulty
: Medium

### Document Corruption
{:#vulnerability-document-corruption}

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
<span class="comment" data-author="RV">Reference Link Rot here, which not only shows content drift but also regular 404</span>
while finding a link to a URL that produces a 404 response should also not cause the query engine to terminate.
<span class="comment" data-author="RV">Or at least not in all cases; it might be desired behavior sometimes</span>

**Exploit: publishing an invalid RDF document**

For example, Carol could decide to introduce a syntax error in her profile document,
or she could simply remove it to produce a 404 response.
This would could cause Alice's queries over her friends from that point on to fail.

Attacker
: Data publisher (Carol)

Victim
: Alice's query engine

Impact
: Crashing of Alice's query engine

Difficulty
: Easy

**Mitigation: sandbox RDF parsing**

The **sandbox** approach is well-suited for handling these types of attacks.
RDF parsing for each document can run in a sandbox,
where errors in this document would simply cause parsing of this document to end without crashing the query engine.
Optionally, a **warning** could be emitted to the user.
The same approach could be followed for HTTP errors on the protocol level, such as HTTP 404's.
This approach is followed by the [Comunica query engine](cite:cites comunica) via its [lenient execution mode](https://comunica.dev/docs/query/advanced/context/#4--lenient-execution).

Location
: LTQP engines

Difficulty
: Medium

**Mitigation: lenient RDF parsing**

An alternative mitigation would be to create more **lenient RDF parsers** that accept syntax errors and attempts to derive the intended meaning,
similar as to how (non-XHTML) HTML parsers are created.
The downside of this is that such parsers would not strictly adhere to their specifications.

Location
: LTQP engines

Difficulty
: Hard

### Cross-query Execution Interaction
{:#vulnerability-cross-query-interaction}

Query engines of all forms typically make use of **caching techniques** to improve performance of query execution.
LTQP query engines can leverage caching techniques for document retrieval.
Within a single query execution, or across multiple query executions,
the documents may be reused, which could reduce the overall number of HTTP requests.
Such forms of caching can lead to vulnerabilities based on **information leaking across different query executions**.
We therefore make the assumption of caching-enabled LTQP engines in this vulnerability.

**Exploit: timing attack to determine prior knowledge**

A first exploit of this vulnerability is an attack that enables Carol to gain knowledge about
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

Attacker
: Data publisher (Carol)

Victim
: Privacy about Alice's document usage

Impact
: Alice's document usage becomes known to Carol

Difficulty
: Hard

**Exploit: unauthenticated cache reuse**

A second exploit assumes the presence of a **software flaw** inside Alice's LTQP query engine
that makes document caches ignore authorization information.
This example is also a form of the *Intermediate Result and Query Leakage* vulnerability that was explained before,
for which we assume the existence of a *hybrid* LTQP query engine.
If Alice queries a private file containing her passwords from a server using its authentication key,
this can cause this passwords file to be cached.
If Carol has a query endpoint that is being queried by Alice,
and Carol is aware of the location of Alice's passwords,
then she could maliciously introduce a link to Alice's passwords file.
Even if the query was not executed with Alice's authentication key,
the bug in Alice's query engine would cause the passwords file to be fetched in full from the cache,
which could cause parts of it to be leaked to Carol's query endpoint.

Attacker
: Data publisher (Carol)

Victim
: Alice's private data

Impact
: Alice's private data is leaked

Difficulty
: Easy (if cache is flawed)

**Mitigation: sandboxing query execution**

In order to mitigate this vulnerability, the [**isolation model** that is used in Web browsers](cite:cites securitymodernwebbrowserarchitecture) could be reused.
When applied to LTQP query engines, this could mean that **each query would be executed in a separate sandbox**,
so that information can not leak across different query executions.
A downside of this approach is that this may cause a significant **performance impact**
when similar queries are executed in sequence, and would cause identical documents to not be reused from the cache anymore.
In order to mitigate this drawback, solutions may be possible to **allow "related queries" to be executed inside the same sandbox**.

Location
: LTQP engines

Difficulty
: Medium

### Document Priority Modification
{:#vulnerability-doc-priority-modification}

Different techniques are possible to [determine the priority of documents](cite:cites WalkingWithoutMap) during query processing.
If queries do not specify a custom ordering, this **prioritization will impact the ordering of query results**.
Some of these techniques are purely graph-based, such as [PageRank](cite:cites pagerank), and can therefore suffer from purely data-driven attacks.
This vulnerability involves **attacks that can influence the priority of documents**,
and thereby maliciously influence what query **results come in earlier or later**.

**Exploit: malicious PageRank prioritization of documents**

One possible exploit is similar to [the attack to modify priorities within crawlers](cite:cites crawlerattacks).
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

Attacker
: Data publisher (Carol)

Victim
: Ranking of documents <span class="comment" data-author="RV">This should be a party though</span>

Impact
: Carol's page is ranked higher

Difficulty
: Medium

**Mitigation: validate API parameters**

Several mitigations have been [proposed for these types of attacks](cite:cites crawlerattacks).
A first solution is to place **responsibility at the API**, and expecting it to patch the exploit.

Location
: Data publishers

Difficulty
: Medium

**Mitigation: content policies**

A second mitigation involves publishers to expose **policies that explicitly authorize what links should be considered legitimate**,
and LTQP query engines inspecting these policies when determining document priorities.

Location
: Data publishers and LTQP engines

Difficulty
: Currently hard (content policies do not exist yet)

**Mitigation: automated learning of legitimate links**

A third mitigation is to use **machine-learning to distinguishing non-legitimate from legitimate links**.
A combination of the three approaches can be used to mitigate this vulnerability.

Location
: LTQP engines

Difficulty
: Hard
