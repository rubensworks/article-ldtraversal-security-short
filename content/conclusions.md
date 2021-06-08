## Conclusions
{:#conclusions}

In this article, we have identified ten prospective security vulnerabilities related to LTQP,
inspired by known vulnerabilities in related domains.
For each vulnerability, we proposed one or more avenues for mitigations.

Some of these vulnerabilities can already be partially tackled through existing security vulnerability mitigation techniques
aimed at both *LTQP engine developers* and *data publishers*.
As such, we **recommend LTQP engine developers to**:

* apply the **same-origin policy** for authentication sessions ([](#vulnerability-session-hijacking));
* only allow traversal using the **HTTP GET** method ([](#vulnerability-session-hijacking));
* **restrict link path lengths** to avoid link traversal traps ([](#vulnerability-link-traversal-trap));
* run untrusted code and RDF parsing over untrusted data in a **sandbox** ([](#vulnerability-arbitrary-code-exec), [](#vulnerability-system-hogging));
* make errors in the sandbox **not crash the query process** ([](#vulnerability-document-corruption)).

At the same time, **recommend data publishers to**:

* **validate input** to avoid data injection ([](#vulnerability-cross-site-injection));
* ensure **HTTP GET** requests are **read-only** ([](#vulnerability-session-hijacking)).

For the following security vulnerabilities, **no concrete mitigation techniques exist yet**:

* Unauthorized Statements ([](#vulnerability-unauthorized-statements))
* Intermediate Result and Query Leakage ([](#vulnerability-intermediate-leakage))
* Cross-query Execution Interaction ([](#vulnerability-cross-query-interaction))
* Document Priority Modification [](#vulnerability-doc-priority-modification)

With this prospective analysis, we have illustrated the importance of more security-oriented research
in the domain on LTQP and the general handling of decentralized environments such as [Solid](cite:cites solid),
especially in presence of data behind authentication.
While some of these vulnerabilities can be mitigated using existing techniques in related domains,
further research on them is needed to test their impact on implementation,
analyze their performance impact,
introduce more performant techniques and algorithms,
and introduce and apply attack models to test their effectiveness.
Furthermore, for the security vulnerabilities for which no concrete mitigations exist yet,
research is perhaps even more critical.
Since our analysis of security vulnerabilities is by no means exhaustive,
additional research efforts are needed to uncover and predict potential security vulnerabilities in LTQP.
Such future research---with our work as a first step---is crucial for enabling a decentralized Web which we can query securely.
