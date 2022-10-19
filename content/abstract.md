## Abstract
<!-- Context      -->
The societal and economic consequences surrounding Big Data-driven platforms
have increased the call for decentralized solutions.
However, retrieving and querying data in more decentralized environments
requires fundamentally different approaches,
whose properties are not yet well understood.
Link-Traversal-based Query Processing (LTQP) is a technique
for querying over decentralized data networks,
in which a client-side query engine discovers data by traversing links between documents.
Since decentralized environments are potentially unsafe due to their non-centrally controlled nature,
<!-- Need         -->
there is a need for client-side LTQP query engines to be resistant against security threats
aimed at the query engine's host machine or the query initiator's personal data.
<!-- Task         -->
As such, we have performed an analysis of potential security vulnerabilities of LTQP.
<!-- Object       -->
This article provides an overview of security threats in related domains,
which are used as inspiration for the identification of 10 LTQP security threats.
<!-- Findings     -->
<!-- Conclusion   -->
This list of security threats forms a basis for future work in which mitigations for each of these threats need to be developed
and tested for their effectiveness.
<!-- Perspectives -->
With this work, we start filling the unknowns for enabling query execution over decentralized environments.
Aside from future work on security, wider research will be needed to uncover missing building blocks for enabling true data decentralization.

<span id="keywords" rel="schema:about"><span class="title">Keywords</span>
<a href="https://en.wikipedia.org/wiki/Linked_Data" resource="http://dbpedia.org/resource/Linked_Data">Linked Data</a>,
<a href="https://en.wikipedia.org/wiki/Resource_Description_Framework" resource="http://dbpedia.org/resource/Resource_Description_Framework">RDF</a>,
Link Traversal,
SPARQL,
Security,
Solid
</span>

<span class="printonly firstpagefooter">
<span class="firstpagefootertop">&nbsp;</span>
<span class="footnotecopyright">
<span style="font-style:italic">6th Workshop on Storing, Querying and Benchmarking Knowledge Graphs (QuWeDa) at ISWC 2022, virtual</span><br />
* Corresponding author.<br />
<img src="img/mail.png" width="12px" /> ruben.taelman@ugent.be (R. Taelman); <img src="img/mail.png" width="12px" /> ruben.verborgh@ugent.be (R. Verborgh)<br />
<img src="img/cc-by.png" width="50px" /><span style="font-size: 0.75em">© 2021 Copyright for this paper by its authors. Use permitted under Creative Commons License Attribution 4.0 International (CC BY 4.0).</span><br />
<img src="img/ceur-ws-logo.png" width="50px" />CEUR Workshop Proceedings (<a href="https://CEUR-WS.org">CEUR-WS.org</a>)<br />
</span>
</span>