# background

Before OAuth, a common way to grant access to protected data was to give third parties your username and password, allowing them to act as you. This method is not secure for two main reasons:

1. Applications store users' passwords in plain text, making them targets for password theft.
2. Applications have complete access to user accounts, including the ability to change passwords.
Recognizing these issues, many services implemented their own solutions similar to OAuth 1.0. However, these solutions were incompatible with each other and often overlooked certain security considerations.

In 2007, a team working on OpenID created a mailing list to develop a standard for API access control that could be used by any system, even those not using OpenID. By August that year, the first draft of the **OAuth 1.0 specification** was published. After seven updated drafts, the OAuth Core 1.0 specification was finalized at the Internet Identity Workshop.

**OAuth 2.0** began as an effort to simplify and improve OAuth 1.0, adding support for mobile applications, which OAuth 1.0 couldn't handle safely. During development, contributors from the web and enterprise worlds had different goals, leading to disagreements. **Draft 10** was published in July 2010, and many services implemented OAuth 2.0 based on this draft.

Over the next 22 revisions, disagreements continued, so conflicting issues were separated into their own drafts, making the standard "**extensible**". By the final draft, so much core content had been moved into separate documents that the core specification became more of a **framework** rather than **protocol**. This made it harder to understand, as implementing OAuth 2.0 required synthesizing information from multiple RFCs and drafts.