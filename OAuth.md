# Background

Before OAuth, a common way to grant access to protected data was to give third parties your username and password, allowing them to act as you. This method is not secure for two main reasons:

1. Applications store users' passwords in plain text, making them targets for password theft.
2. Applications have complete access to user accounts, including the ability to change passwords.
Recognizing these issues, many services implemented their own solutions similar to OAuth 1.0. However, these solutions were incompatible with each other and often overlooked certain security considerations.

In 2007, a team working on OpenID created a mailing list to develop a standard for API access control that could be used by any system, even those not using OpenID. By August that year, the first draft of the **OAuth 1.0 specification** was published. After seven updated drafts, the OAuth Core 1.0 specification was finalized at the Internet Identity Workshop.

**OAuth 2.0** began as an effort to simplify and improve OAuth 1.0, adding support for mobile applications, which OAuth 1.0 couldn't handle safely. During development, contributors from the web and enterprise worlds had different goals, leading to disagreements. **Draft 10** was published in July 2010, and many services implemented OAuth 2.0 based on this draft.

Over the next 22 revisions, disagreements continued, so conflicting issues were separated into their own drafts, making the standard "**extensible**". By the final draft, so much core content had been moved into separate documents that the core specification became more of a **framework** rather than **protocol**. This made it harder to understand, as implementing OAuth 2.0 required synthesizing information from multiple RFCs and drafts.

# Steps for Building an App Using an existing OAuth 2.0 service (with GitHub as an Example)

## Register a New Application
    1. Create a Developer Account: Sign up on the service's website.
    2. Enter Basic Information
        - Redirect URLs:
            These are the URLs where the OAuth 2.0 service will redirect users after authorization. They must be HTTPS to prevent intercepted during authorization process and must be registered with the service to prevent the redirection attacks. Use a local address if developing locally. Typically, create two applications: one for development and one for production.
    3. Once this is done, you will get credentials (client_id and client_secret)
        - client_id: Used to build authorization URLs. Can be included in the JavaScript source code.
        - client_secret: Must be kept confidential and usually stored in the backend.
        ![Github application created](/assets/github_application_created.png)

## Initiate Login Process: Visit *'https://github.com/login/oauth/authorize'* with `?action=login` in the query string to get authorization code.

```
    // Start the login process by sending the user to GitHub's authorization page

    // Remove access token from session storage
    sessionStorage.removeItem('access_token');

    // Generate a random state and store it in session storage
    const state = crypto.getRandomValues(new Uint8Array(16)).join('');
    sessionStorage.setItem('state', state);

    // Define the parameters for GitHub's authorization page
    const queryParams = new URLSearchParams({
        response_type: 'code',
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

After the user approves the request, they will be redirected back with code and state parameters in url like *"http://localhost:3000/?code=3a9db23f832d7f7128ef&state=82831621252225151913920917055165101170"*

* Ensure access token and state are stored in session storage to avoid losing them when the component unmounts.
* The "State" parameter here is used to verify if it matches with the state stored in the session. It is used to protect the client from CSRF attacks.

## Obtain access token: Send a request to Github's token endpoint *'https://github.com/login/oauth/access_token'* to exchange the authorization code for an access token.
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
