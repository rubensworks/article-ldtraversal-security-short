## Related Work
{:#related-work}

This section lists relevant related work in the topics of LTQP and security.

### Link-Traversal-based Query Processing

Write me
{:.todo}

### Security Vulnerabilities

In this section, we describe related work on security vulnerabilities in different areas related to LTQP.

#### Web Crawlers

Write me
{:.todo}

#### Web Browsers

Web browsers enable users to visualize and interact with Web pages.
This interaction is closely related to LTQP,
with the main difference that LTQP works autonomously,
while Web browsers are user-driven.
Considering this close resemblance between these two domains,
we give an overview of the main security vulnerabilities in Web browsers.

**Modern Web browser architecture:**

[Silic et al.](cite:cites securitymodernwebbrowserarchitecture)
analyzed the architectures of modern Web browsers,
determined the main vulnerabilities,
and discuss how these issues are coped with.

Architecture-wise, browsers can be categorized into monolithic and modular browser architectures.
The difference between the two is that the former does not provide isolation between concurrently executed Web programs, while the latter does.
The authors argue that a modular architecture is important for security, fault-tolerance and memory management.
They focused on the security aspects of the [Chrome browser architecture](cite:cites securitychromium),
which consist of separate modules for the rendering engine, browser kernel, and plugins.
Each of these modules is isolated in its own operating system process.

Silic et al. list the following main threats for Web browsers:

1. **System compromise**: Malicious arbitrary code execution with full privileges on behalf of the user. For example, exploits in the browser or third-party plugins caused by bugs. These types of attacks are mitigated through automatic updates once exploits become known.
2. **Data theft**: Ability to steal local network or system data. For example, a Web page includes a subresource to URLs using the file scheme (`file://`). which are usually blocked.
3. **Cross domain compromise**: Code from a Fully Qualified Domain Name (FQDN) executes code (or reads data) from another FQDN. For example, a malicious domain could extract authentication cookies from your bank's website you are logged into. This is usually blocked through the same-origin policy, but can be explicitly allowed through [Cross-Origin Resource Sharing (CORS)](https://fetch.spec.whatwg.org/#http-cors-protocol){:.mandatory}.
4. **Session hijacking**: Session tokens are compromised through theft or session token prediction. For example, [cross-domain request forgery (CSRF)](cite:cites csrf) is a type of attack that involves an attacker forcing a user logged in on another Web site to perform an action without their consent. Web browsers do not protect against these, but are typically handled by Web frameworks via the [Synchronizer Token Pattern](cite:cites synchronizertokenpattern).
5. **User interface compromise**: Manipulating the user interface to trick the user into performing an action without their knowledge. For example, placing an invisible button in front of another button. This category also includes CPU and memory hogging to block the user from taking any further actions. Web browser have limited protections for these types of attacks that involve placing limitations on user interface manipulations.

**Lessons from Google Chrome:**

[Reis et al.](cite:cites securitychromelessons) discuss on the three problems Google Chrome developers focus on to mitigate attacks:

1. **Reducing vulnerability severity**: In the real world, large projects such as Web browsers always contain bugs. Given this reality, Google Chrome consists of several sandbox layers reducing the damage should an exploit be discovered in one of the layers. The difficulty here lies in the fact that Web compatibility should be maintained, so that security restrictions do not break people's favorite Web sites.
2. **Reducing window of vulnerability**: If an exploit has been discovered, it should be patched as soon as possible. Google Chrome follows automated testing to ship security patches as soon as possible. All existing Chrome installations check for updates every five hours, and update in the background without disrupting the user experience.
3. **Reducing frequency of exposure**: In order to avoid people from visiting malicious Web sites for which the browser has not been patched yet, Google Chrome makes use of a continuously updating database of such Web sites. This will show a warning to the user before visiting such a site.

**SQL Injection**

[SQL injection attacks](cite:cites sqlinjection) are one of the most common vulnerabilities on Web sites
where (direct or indirect) user input is not properly handled, and may lead to the attacker performing unintended SQL statements on databases.
These types of attacks are typically mitigated through strong input validation, which are typically available in reusable libraries.

**Techniques for mitigating browser vulnerabilities:**

[Browser Hardening](cite:cites hardening) is based on the concept of reducing privileges of browsers to increase security.
For example, browsers can be configured to disabled JavaScript and Adobe Flash, or whitelisted to trusted Web sites.

[Fuzzing](cite:cites fuzzing) is a technique that involves generating random data as input to software.
Major Web browsers such as Google Chrom and Microsoft Edge perform extensive fuzzed testing by generating random Web pages and running them through the browser to detect crashes and other vulnerabilities.

#### RDF Query Processing 

Write me
{:.todo}
