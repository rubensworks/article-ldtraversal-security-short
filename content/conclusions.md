## Conclusions
{:#conclusions}

In this article, we have identified ten prospective security threats related to LTQP,
inspired by known threats in related domains.
For each threat, we proposed one or more theoretical mitigations.

Some of these threats can already be partially tackled through existing security threat mitigation techniques
aimed at both *LTQP engine developers* and *data publishers*.
As such, we **recommend LTQP engine developers to**:

* apply the same-origin policy for authentication sessions ([](#threat-session-hijacking));
* only allow traversal using the HTTP GET method ([](#threat-session-hijacking));
* restrict link path lengths to avoid explicit and implicit link cycles ([](#threat-link-cycles));
* run untrusted code and RDF parsing over untrusted data in a sandbox ([](#threat-arbitrary-code-exec), [](#threat-system-hogging));
* don't let errors in the sandbox crash the query process ([](#threat-document-corruption)).

At the same time, **recommend data publishers to**:

* validate input to avoid data injection ([](#threat-cross-site-injection));
* ensure HTTP GET requests are be read-only ([](#threat-session-hijacking)).

For the following security threats, **no concrete mitigation techniques exist yet**:

* Unauthorized Statements ([](#threat-unauthorized-statements))
* Intermediate Result and Query Leakage ([](#threat-intermediate-leakage))
* Cross-query Execution Interaction ([](#threat-cross-query-interaction))
* Document Priority Modification [](#threat-doc-priority-modification)

With this prospective analysis, we hope to have illustrated the importance of more security-oriented research
in the domain on LTQP and the general handling of decentralized environments such as [Solid](cite:cites solid).
While some of these threats can be mitigated using existing techniques in related domains,
further research on them is needed to test their impact on implementation,
analyze their performance impact,
introduce more performant techniques and algorithms,
and introduce and apply attack models to test their effectiveness.
Furthermore, for the security threats for which no concrete mitigations exist yet,
research is perhaps even more critical.
Since our analysis of security threats is by no means complete,
additional research effort is needed to uncover and predict potential security vulnerabilities in LTQP.
Such future research --with our work as a first step-- is crucial for enabling a decentralized Web over which we can query securely.
