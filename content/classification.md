## Classification of Security Vulnerabilities
{:#classification}

In this section, we first introduce the background on classifying security vulnerabilities in software.
After that, we introduce a classification method specifically for the LTQP domain,
to assess the validity of our work.

### Background

Security vulnerabilities in software [can be classified using many different methods](cite:cites classifyingsecurityvulnerabilities, formalmodelingvulnerability).
Generic classification methods often result in very large taxonomies,
which are shown to [result in practical problems](cite:cites classifyingsecurityvulnerabilities) because of their size and complexity.

[Seacord et al.](cite:cites classifyingsecurityvulnerabilities)
claim that classification methods must be based on engineering analysis of the problem domain,
instead of being too generic.
For this, they suggest the use of domain-specific attributes for classifying security vulnerabilities for each domain separately.
Furthermore, they introduce the following terminology for security vulnerabilities,
by building upon [earlier formal definitions of vulnerabilities](cite:cites formalmodelingvulnerability):

Security flaw
: A defect in a software application or component that, when combined with the necessary conditions, can lead to a software vulnerability.

Vulnerability
: A set of conditions that allows violation of an explicit or implicit security policy.

Exploit
: A technique that takes advantage of a security vulnerability to violate an explicit or implicit security policy.

Mitigation
: Techniques to prevent or limit exploits against vulnerabilities.

For the remainder of this article, we will make use of this terminology,
and we adopt a method hereafter for classifying software vulnerabilities specific to the LTQP domain
as recommended by [Seacord et al.](cite:cites classifyingsecurityvulnerabilities).

### Classification Method

Our classification method considers the listing of several security _vulnerabilities_.
Each vulnerability has one or more possible _exploits_, which may take advantage of this vulnerability.
The different properties of each exploit are shown in [](#table-exploit-properties).

<figure id="table-exploit-properties" markdown="1" class="table">

| Attribute                             | Values     |
|---------------------------------------|------------|
| Attacker 				                | Data publisher, ... |
| Victim 				                | LTQP engine, query initiator, data publisher, ... |
| Impact 				                | Incorrect query results, system crash, ... |
| Difficulty 				            | Easy, medium, hard |

<figcaption markdown="block">
Exploit properties specific to LTQP, with several possible values for each attribute.
</figcaption>
</figure>
