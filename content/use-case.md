## Use Case
{:#use-case}

In this section, we introduce a use case
that will be used to illustrate the security threats discussed throughout this article.

We assume a Web with public and private information,
which may for instance be achieved via personal data vaults
following the principles of the [Solid ecosystem](cite:cites solid).
This data vault is in full control of the owner,
and they can host any kind of file in here, such as Linked Data files.

<figure id="figure-use-case">
<img src="img/use-case.svg" alt="[Personal Address Book]" class="figure-width-twothird">
<figcaption markdown="block">
Overview of the address book use case
in which Alice has an address book with links to the profiles of Carol and Bob,
which contain further details.
</figcaption>
</figure>

For this use case, we assume the existence of three people (Alice, Bob, and Carol),
each having their own personal data vault.
Alice uses her vault to store an address book containing the people she knows.
Instead of storing contact details directly in the address book,
she stores _links_ to the profiles of her contacts (Bob and Carol).
Bob and Carol can then self-define their own contact details.
[](#figure-use-case) shows an illustration of this setup.

The LTQP paradigm is well-suited to handle query execution over such setups.
If Alice for instance would like to obtain the names of all her contacts,
she could initiate a query starting from her address book as seed document,
and the query engine would follow the links to her contacts,
and obtain the names from their respective profiles.
Some documents may require authentication before they can be accessed,
for which Alice's query engine makes use of Alice's identity.
In all threats throughout this article,
we assume that Carol has malicious intentions that Alice is unaware of.

In this use case, two main roles can be identified.
The first is the role of data publisher,
which is taken up by Alice, Bob, and Carol though their person data vaults.
The second is the role of the query initiator,
which here applies to Alice, as she issues a query over her contacts.
