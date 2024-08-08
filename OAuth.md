# Background

Before OAuth, a common way to grant access to protected data was to give third parties your username and password, allowing them to act as you. This method is not secure for two main reasons:

1. Applications store users' passwords in plain text, making them targets for password theft.
2. Applications have complete access to user accounts, including the ability to change passwords.
Recognizing these issues, many services implemented their own solutions similar to OAuth 1.0. However, these solutions were incompatible with each other and often overlooked certain security considerations.

In 2007, a team working on OpenID created a mailing list to develop a standard for API access control that could be used by any system, even those not using OpenID. By August that year, the first draft of the **OAuth 1.0 specification** was published. After seven updated drafts, the OAuth Core 1.0 specification was finalized at the Internet Identity Workshop.

**OAuth 2.0** began as an effort to simplify and improve OAuth 1.0, adding support for mobile applications, which OAuth 1.0 couldn't handle safely. During development, contributors from the web and enterprise worlds had different goals, leading to disagreements. **Draft 10** was published in July 2010, and many services implemented OAuth 2.0 based on this draft.

Over the next 22 revisions, disagreements continued, so conflicting issues were separated into their own drafts, making the standard "**extensible**". By the final draft, so much core content had been moved into separate documents that the core specification became more of a **framework** rather than **protocol**. This made it harder to understand, as implementing OAuth 2.0 required synthesizing information from multiple RFCs and drafts.



# Terms
**Authentication**
: is to verify the identity of a user or system. It answers the question, "Who are you?". During authentication, a user provides credentials, such as a username and password, to prove their identity. The system checks if these credentials match the ones stored in the system.

**Authorization**
: is to determin what an authenticated user is allowed to do. It answers the question, "What are you allowed to do?". Once a user is authenticated, the system checks their permissions or roles to determine what resources or actions they are authorized to access or perform.

**Resource Owner**
: Entity that can grant access to a protected resource. Typically, this is the end-user.

**Resource Server**
: Server hosting the protected resources. This is the API you want to access.

**Authorization Server**
: Server that authenticates the Resource Owner and issues Access Tokens after getting proper authorization.

**Client**
: Application requesting access to a protected resource on behalf of the Resource Owner.

**Authorization code**
: A temporary code provided by the authorization server, which the client exchanges for an access token after the user has reviewed and granted the requested permissions.

**Flow / grant types**
: OAuth 2.0 flows, also known as grant types, are different methods for obtaining access tokens. These flows are designed for various scenarios depending on the type of client (e.g., web app, mobile app, single-page app) and the required level of security.

**Authorization request parameters**
: Parameters appended to the application's authorization endpoint in the query string to initiate the authorization process.

**State**
: The state parameter in the authorization request serves two purposes: it allows the app to store data to indicate what action to perform after authorization, and it acts as a CSRF protection mechanism.

**Access token**
: In OAuth, the access token is used for authorization and does not contain information about the user's identity.

**PKCE (Proof Key for Code Exchange)**
: An extension to the OAuth 2.0 authorization code flow that enhances security, particularly for public clients like mobile or single-page applications.

**CSRF (Cross-Site Request Forgery) protection mechanisms**
: Methods designed to prevent unauthorized actions on a web application by ensuring that requests are made by the authenticated user and not by an attacker.



# Common OAuth 2.0 Flows (Grant Types)
- **Authorization Code Flow**
    - **Use Case:** Web applications with a back-end server.
    - **Description:** This flow involves the client (application) redirecting the user to an authorization server to request an authorization code. The client exchanges this code for an access token and, optionally, a refresh token. It's secure because the client secret and tokens are never exposed to the user's browser.
    - **Security:** High.
    - **Steps:**
        1. User Authorization Request: The client redirects the user to the authorization server with parameters like client ID and redirect URI, asking for permission to access their data.
        2. User Grants Authorization: The user grants access, and the authorization server redirects them back to the client with an authorization code.
        3. Client Exchanges Authorization Code for Tokens: The client’s back-end server sends a POST request to the token endpoint with the authorization code and client credentials to exchange it for tokens.
        4. Authorization Server Issues Tokens: The server validates the request and responds with an access token and optionally a refresh token.
        5. Client Uses the Access Token: The client uses the access token to make authorized API requests to access protected resources.
- **Implicit Flow**
    - **Use Case:** Single-page applications (SPAs) or applications where the client code runs entirely in the browser and doesn't have a back-end server.
    - **Description:** The client directly receives an access token from the authorization server after the user grants permission. No authorization code is involved. It's faster but less secure because the access token is exposed to the user's browser.
    - **Security:** Moderate, with increased risks if proper precautions (e.g., short-lived tokens) aren't taken.
- **Resource Owner Password Credentials Flow (Password Flow)**
    - **Use Case:** Trusted applications where the resource owner (user) has a high level of trust in the client, such as first-party apps.
    - **Description:** The user provides their username and password directly to the client, which then uses these credentials to obtain an access token. This flow is not recommended for third-party apps due to security concerns.
    - **Security:** Low, since the client handles the user's credentials directly.
- **Device Authorization Flow (Device Flow)**
    - **Use Case:** Devices with limited input capabilities, such as smart TVs or IoT devices.
    - **Description:** The user is instructed to visit a URL on a separate device (like a phone or computer) and enter a code to authorize the client. The client then polls the authorization server to obtain an access token once the user has authorized the device.
    - **Security:** Moderate, with potential risks if the code is intercepted during the process.
- **Refresh Token Flow:**
    - **Use Case:** Refreshing an access token when it expires without requiring the user to re-authenticate.
    - **Description:** The client uses a refresh token, which was previously issued along with the access token, to obtain a new access token. This flow is often used in conjunction with the Authorization Code Flow.
    - **Security:** High, assuming proper handling of refresh tokens.
- **Authorization Code Flow with PKCE**
    - **Use Case:** mobile or single-page applications
    - **Steps:**
        1. Generate Code Verifier and Code Challenge: Create a random string called the **code verifier**. Then, generate a **code challenge** by applying a transformation (usually SHA-256) to the code verifier.
        2. Redirect User for Authorization: Create a URL that redirects the user’s browser to the OAuth authorization server. This URL includes the code challenge and the method used in parameter to generate it. The authorization server will then prompt the user to authorize the application and send an authorization code to the user’s browser.
        3. Exchange Code for Access Token: Send an HTTPS POST request to the authorization server with the code verifier to exchange the authorization code for an access token.



# Steps for Building an App Using an Existing OAuth 2.0 Service with GitHub as an Example (Authorization Code Flow)

## Register a New Application
    1. Create a Developer Account: Sign up on the service's website.
    2. Enter Basic Information
        - Redirect URLs:
            These are the URLs where the OAuth 2.0 service will redirect users after authorization. They must be HTTPS to prevent intercepted during authorization process and must be registered with the service to prevent the redirection attacks. Use a local address if developing locally. Typically, create two applications: one for development and one for production.
    3. Once this is done, you will get credentials (client_id and client_secret)
        - client_id: Used to build authorization URLs. Can be included in the JavaScript source code.
        - client_secret: Must be kept confidential and usually stored in the backend.
        ![Github application created](/assets/github_application_created.png)

## Request authorization from the user: Visit an authorization request link *'https://github.com/login/oauth/authorize'* with `?action=login` in the query string to get authorization code.
* For security, the authorization page must appear in a web browser that the user is familiar with and should not be embedded in an iframe, popup, or an embedded browser within a mobile app.

```
    // Start the login process by sending the user to GitHub's authorization page

    // Remove access token from session storage
    sessionStorage.removeItem('access_token');

    // Generate a random state and store it in session storage
    const state = crypto.getRandomValues(new Uint8Array(16)).join('');
    sessionStorage.setItem('state', state);

    // Define the parameters for GitHub's authorization page
    const queryParams = new URLSearchParams({
        response_type: 'code',      // return an authorization code as the response
        client_id: githubClientID,
        redirect_uri: `${window.location.origin}${window.location.pathname}`,
        scope: 'user public_repo',
        state: state
    });

    // Redirect the user to GitHub's authorization page
    window.location.href = `${authorizeURL}?${queryParams.toString()}`;
```

The last line of code will open up the github OAuth authorizarion propmt.
![Github oauth prompt](/assets/github_oauth_prompt.png)

After the user approves the request, the server will redirect back to the app with code and state parameters in url like *"http://localhost:3000/?code=3a9db23f832d7f7128ef&state=82831621252225151913920917055165101170"*

* Ensure access token and state are stored in session storage to avoid losing them when the component unmounts.
* The "State" parameter here is used to verify if it matches with the state stored in the session. It is used to protect the client from CSRF attacks.

## Obtain access token: Send a POST request to Github's token endpoint *'https://github.com/login/oauth/access_token'* to exchange the authorization code for an access token.
The service will require the client to authenticate itself when making this request. Typically, services support client authentication using HTTP Basic Auth with the client’s client_id and client_secret. If an app uses the authorization code grant but cannot securely protect its client secret, then the client secret is not required, and PKCE must be used. In such cases, the client-side JavaScript code may need a companion server-side component to perform the OAuth flow securely.

```
    // Verify the state matches our stored state
    const storedState = sessionStorage.getItem('state');
    if (!state || storedState !== state) {
        window.location.href = `${baseURL}?error=invalid_state`;
        return;
    }

    // Prepare the data to exchange the auth code for an access token
    const tokenRequestData = new URLSearchParams({
        grant_type: 'authorization_code',
        client_id: githubClientID,
        client_secret: githubClientSecret,
        redirect_uri: baseURL,
        code: code
    });

    // Make a POST request to exchange the code for an access token
    fetch('https://github.com/login/oauth/access_token', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: tokenRequestData
    })
    .then(response => response.json())
    .then(data => {
        if (data.access_token) {
            // Store the access token in session storage
            sessionStorage.setItem('access_token', data.access_token);

            // Redirect to the base URL
            window.location.href = baseURL;
        }
    })
```

The response will look like below:
```
{
    "access_token": "e2f8c8e136c73b1e909bb1021b3b4c29",
    "token_type": "Bearer",
    "scope": "public_repo,user"
}
```

## Make API request with access token
The user's browser never makes a direct request to the API server. Instead, all requests go through the client first, as the access token is maintained in the backend.
```
// Use access token to get the lists of repo for the user
const url = 'https://api.github.com/user/repos?sort=created&direction=desc';

// Auth header included accept and user-agent that github required and with access token
const headers = new Headers({
    'Accept': 'application/vnd.github.v3+json',
    'User-Agent': 'http://localhost:3000/',
    'Authorization': 'Bearer e2f8c8e136c73b1e909bb1021b3b4c29'
});

// Make authenticate request
fetch(url, {
    method: 'GET',
    headers: headers
})
.then(response => response.json())
.then(repos => {
    console.log(repos); // Handle the response data
});
```

There are several ways to identify a user:

1. The common approach is for the API to provide "user info."
2. A more advanced and standardized method is using OpenID Connect, an extension of OAuth 2.0.

## Possible Errors During Authorization and How to Handle Them
When errors occur, the user will be redirected back to the provided redirect URL with additional parameters error_uri and error_description in the query string. You should present your own user-friendly error message instead of directly displaying the error_description.

- Invalid redirect URL
- Unrecognized client_id
- User denies the request
- Invalid parameters
- Invalid request
- Unauthorized_client
- Unsupported_response_type
- Invalid_scope
- Server_error
- Temporarily_unavailable



# Others
