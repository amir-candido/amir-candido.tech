+++
author = "Amir Candido"
title = "Building a Robust Role-Based Authentication System With JWT Tokens & React Context. (Part I)"
date = "2024-08-11"
description = "Learn How to Protect Routes with JWTs, React Context, and Middleware"
draft  = false
tags = [
    "jwt",
    "react",
    "express",
    "javascript",
]
categories = [
    "React",
    "APIs",
]
series = ["Tutorials"]
image = "react-jwt-auth.png"
+++

## So What Is This Post About?

In enterprise-level applications, robust authentication systems are essential for managing user access and protecting sensitive data. In this article, we’ll explore how to implement an enterprise-grade authentication system in React and Express using a combination of modern libraries: React Context for global state management, JSON Web Tokens (JWT) for secure token-based authentication, and React Router for routing. By the end, you’ll have an understanding of how to build a scalable, enterprise-grade authentication system that can handle different user roles and permissions seamlessly.

This tutorial is in two parts; this, Part I, will deal mostly with the back-end Express API, while in another post, Part II, we will discuss the front-end of the app.
If you're in a hurry and merely need access to the code, you can download the github repo [here](https://github.com/amir-canteetu/react-express-auth-app).

## 1. Setting Up the Environment

In this section, we'll set up the development environment. We'll create separate folders for the back-end and front-end to organize the API and the React app cleanly. Both folders will be inside a main project directory named `react-jwt-auth-app`. Here’s how we’ll set it up:

```bash
react-jwt-auth-app/
├── api/             # Folder for Express API (back end)
├── client/          # Folder for React app (front end)
└── README.md
```

#### Setting Up the Back-End (API)
Initialize the API Folder: Open a terminal, navigate to the react-jwt-auth-app directory, and initialize the api folder:

```bash
mkdir react-jwt-auth-app
cd react-jwt-auth-app
mkdir api
cd api
npm init -y
```
**Install Dependencies**: In the api folder, we’ll install the necessary dependencies:

```bash
npm install axios bcryptjs cookie-parser cors dotenv express json-server jsonwebtoken lodash morgan
npm install --save-dev nodemon
```


**Nodemon Setup:** Nodemon will help us by automatically restarting the server when we make changes to our files. Add the following script to your package.json in the api folder:

```json
"scripts": {
  "start": "nodemon index.js"
}
```

## 2. Understanding Role-Based Authentication

In the context of enterprise applications, where multiple types of users (e.g., admins, managers, users) interact with the system, role-based authentication provides a scalable and manageable way to control access to resources, reduce security risks, and maintain a clear structure of permissions within the application.

The approach we are taking in this article leverages [React Context](https://react.dev/learn/passing-data-deeply-with-context) for state management, JWT (JSON Web Tokens) for handling user authentication, and role-based access control (RBAC) to dynamically control user permissions. This method aligns well with best practices in enterprise-level applications for several reasons:

1. **Security:** By using JWT tokens, sensitive information about users and their roles can be securely encoded and verified. Tokens are issued by a trusted server and signed with a secret key, ensuring that user data cannot be easily tampered with.

2. **Scalability:** With role-based authentication, you can easily scale your application as new roles or user types are introduced. Instead of hardcoding permissions throughout your application, roles and permissions are managed centrally, allowing for greater flexibility and ease of maintenance.

3. **Separation of Concerns:** By separating concerns between authentication, authorization, and user management, the system becomes more modular. JWTs handle authentication, and React Context manages global user states and roles.

4. **Performance:** Role checks using JWTs are highly performant. Since JWT tokens are stored on the client-side, no extra database lookup is needed for each user request. This improves the speed of authorization decisions, making it suitable for high-performance, enterprise applications.

5. **Consistency with Industry Standards:** Role-based authentication using JWT tokens follows widely adopted security standards, ensuring compatibility with other systems.

But how does JWT token authentication actually work? Where exactly should the JWT token be stored on the client side? And which storage option is most secure?  

### JWT Token Authentication Overview

**JWT** is an open standard that defines a compact, self-contained method for securely transmitting information between parties as a JSON object. In the context of authentication, JWT is used to identify users after they log in, allowing them to access protected routes without needing to log in again during the token's lifetime.

A JWT token typically consists of three parts:

1. **Header:** Contains metadata about the token, such as the type (JWT) and the signing algorithm (e.g., ES256).
2. **Payload:** Contains the claims or data you want to transmit, such as user information (e.g., id, role). 
3. **Signature:** The signature ensures the token’s integrity by combining the encoded header, payload, and a secret key, ensuring that any modification of the token would render it invalid.

When a user logs in, our code will generate a JWT token and will include the user's `role` in the payload. The server will then send this token (`accessToken`) back to the client, which stores it and sends it in the `Authorization` header ***with each subsequent request***. The server will verify the token and will check the user's `role` before allowing access to role-protected routes.

### Why JWT is Ideal for Role-Based Authentication

1. **Stateless and Scalable:** JWT does not require server-side storage of session data, making it ideal for scalable enterprise applications where millions of users may need to be authenticated.
2. **Self-Contained Data:** Since the JWT payload contains user information (like roles), the server can easily verify and authorize users based on their token, without querying a database for every request.
3. **Flexibility:** JWT can be easily extended to include additional metadata, like additional roles or permissions, making it a perfect fit for role-based access control (RBAC) systems.

### Token Storage Options and Security Considerations

Once a JWT token is issued, the client needs to store it securely. There are two common storage options: **localStorage** and **cookies**.

1. **localStorage:**
   
    **Pros:**

* Easy to implement.
* Persistent even if the user refreshes the page or closes the browser.
  
  **Cons:**
* Vulnerable to XSS (Cross-Site Scripting) attacks. If an attacker injects malicious JavaScript, they can access the token stored in localStorage.

2. **Cookies:**
   
    **Pros:**

* If used with the `HttpOnly` flag, the cookie is inaccessible to JavaScript, making it more secure against XSS attacks.
* Cookies can be sent with the `Secure` flag to ensure they are transmitted only over HTTPS.
* Cookies can be automatically sent with each request, reducing the need to manually include the token in headers.
  
  **Cons:**

* Vulnerable to CSRF (Cross-Site Request Forgery) attacks, although this can be mitigated using the `SameSite` attribute.

#### Which Option is More Secure?
Cookies with `HttpOnly` and `Secure` Flags are generally considered the most secure option for storing JWT tokens, particularly for sensitive applications such as enterprise systems. Storing JWTs in cookies protects them from being accessed by client-side JavaScript, mitigating the risk of XSS attacks. Combining this with `SameSite` and other CSRF protection techniques provides a robust solution for enterprise-level security.

#### Best Practices for JWT Authentication

1. **Use Short Token Expiry:** JWT tokens should have a short lifespan (e.g., 15 minutes to an hour) to limit the damage in case of theft.
2. **Refresh Tokens:** Use refresh tokens with longer expiration periods to renew access tokens without forcing the user to log in again.
3. **Signature Verification:** Always verify the JWT signature on the server-side to ensure the token’s integrity and authenticity.
4. **Encrypt Sensitive Data:** While JWT is not encrypted by default, encrypt sensitive claims if needed.

In the next section, we will explore how to implement JWT-based authentication on the back-end of our app.

## 3. Backend Setup for Authentication

### Setting Up the Mock JSON Server

We will create a `db.json` file that will act as our "database," containing a collection of users. This file should be placed in the `/api` directory of your project.

The data should be of the form

```json
{
  "users": [
    {
      "id": "1",
      "username": "admin",
      "password": "$2a$10$xRltr79V66a0PaWqGKM62ebiYs.V9J1Vvv0k6a.CtDK.G0ym9BbzS", // hashed password
      "role": "admin",
      "email": "admin@example.com",
      "favColor": "#1A535C",
      "notifications": {
        "alerts": 6,
        "mail": 15
      }
    },
    {
      "id": "2",
      "username": "user",
      "password": "$2a$10$xRltr79V66a0PaWqGKM62ebiYs.V9J1Vvv0k6a.CtDK.G0ym9BbzS", // hashed password
      "role": "user",
      "email": "user@example.com",
      "favColor": "#FF6F61",
      "notifications": {
        "alerts": 9,
        "mail": 2
      }
    }
  ]
}

```
You can then run the mock server with:

```
npx json-server --watch db.json --port 3001
```
This will simulate a REST API where `/users` is an endpoint for accessing user data.

### Express Server Setup

#### Configuring Environment Variables

Let's now set up our environment variables. Create a `.env` file in the root of the api and specify the environment variables:

```
# Server configuration
PORT=3000
CLIENT_URL=http://localhost:5173

# Token configuration
accessTokenExpiresIn=15m
refreshTokenExpiresIn=7d

# JSON Server URL for user data
APIUSERSURL=http://localhost:3001/users
```

Each environment variable in this file will be accessed in `index.js` through the `dotenv` package, which loads these variables into `process.env`. `PORT` is
self-explanatory. `CLIENT_URL` is needed for our CORS configuration. `accessTokenExpiresIn` defines how long access tokens are valid. We’re using a short-lived duration (`15m`) for access tokens, ensuring that if an access token is compromised, it expires quickly. `refreshTokenExpiresIn` refers to the refresh token duration (`7d`), which is longer than that of the access token, allowing users to maintain their sessions without needing to re-authenticate frequently. This expiration is adjustable depending on your application’s security needs.

### Loading Environment Variables with dotenv

In the `index.js` file, the following line loads these environment variables:

```JavaScript
import dotenv from 'dotenv';
dotenv.config();

```

The `dotenv.config()` function reads from the `.env` file and makes each variable accessible via `process.env`. This setup ensures sensitive information, like API endpoints and token configurations, is handled securely. Additionally, if a variable is missing, default values are provided directly in the code:

```JavaScript
const PORT                  = process.env.PORT || 3000;
const requestOrigin         = process.env.CLIENT_URL || "http://localhost:5173";
const apiUserEndpoint       = process.env.APIUSERSURL || "http://localhost:3001/users";
const accessTokenExpiresIn  = process.env.accessTokenExpiresIn || '15m';
const refreshTokenExpiresIn = process.env.refreshTokenExpiresIn || '7d';

```

Using environment variables in this way enhances flexibility, making the application easily configurable across different environments, and reduces the risk of exposing sensitive information directly in the codebase. When deploying to production, remember to set up these environment variables on the server to ensure the backend functions correctly without relying on hard-coded values. Also remember to add your `.env` file to your gitignore!

### Setting Up Dependencies and Middleware

In this section, we’ll set up the key dependencies and middleware in our Express server to handle authentication, parsing of requests, security configurations, and logging for our role-based authentication system.

#### Key Dependencies

Let’s take a closer look at the core dependencies in `package.json`, each serving a specific purpose in building a secure and scalable authentication API:

* **bcryptjs:** This library helps securely hash user passwords before storing them. Hashing passwords makes it difficult for attackers to decipher if the database is compromised. With `bcrypt`, we can add salt to each password, making identical passwords have different hashes.

* **cookie-parser:** This middleware enables our server to parse cookies attached to the client’s HTTP requests. By using cookies to store our refresh tokens, we can leverage the security of HTTP-only cookies, which are inaccessible to client-side JavaScript, protecting them from common attacks like XSS.

* **cors:** Cross-Origin Resource Sharing (CORS) is essential when developing a front end and back end that interact across different origins. Since our React front end and Express back end run on different ports, CORS enables our API to accept requests from the front-end app's URL (`CLIENT_URL`), enhancing security by only allowing trusted origins.

* **dotenv:** We use `dotenv` to load environment variables from a `.env` file, keeping sensitive information (such as API keys and secrets) out of our source code. This setup ensures that our tokens, keys, and other private configurations are not directly exposed in the codebase.

* **jsonwebtoken:** This library generates and verifies JSON Web Tokens (JWTs). Here, JWTs will store user information, enabling the front end to verify users without needing to constantly query the database.

* **lodash:** This utility library includes functions that simplify operations such as picking specific fields from objects, which we’ll use to exclude sensitive fields like passwords from the response.

* **morgan:**(optional) Morgan logs each HTTP request, providing insights into incoming traffic and errors. This is particularly helpful in development and debugging, as it provides immediate feedback on every interaction with the API.

#### Adding Middleware

To properly handle requests and manage sessions, we’ll configure several middleware functions in our index.js file:

```JavaScript
app.use(express.json()); // Parses incoming JSON data
app.use(cookieParser()); // Enables cookie-based session management
app.use(cors({ origin: requestOrigin, credentials: true })); // Restricts CORS to trusted origin
app.use(morgan('combined')); // Logs all HTTP requests in a detailed format

```

Here’s an overview of each middleware and why it’s important:

* **express.json():** This middleware parses the JSON payload of incoming requests. Without this, our server wouldn’t be able to understand or respond to JSON data.

* **cookieParser():** The cookie-parser middleware lets us access cookies in incoming requests. 

* **cors():** CORS is configured with options to allow requests only from a specific origin, `requestOrigin` (the URL of our React front end). Setting `credentials: true` allows cookies to be included in requests, which is necessary for our authentication flow to access refresh tokens stored in cookies. This setup enhances security by preventing unauthorized domains from interacting with our API.

* **morgan():** Morgan logs each HTTP request, capturing details like the HTTP method, URL, status code, and response time. In development, we use the `combined` format, which provides detailed logs to help track user actions and pinpoint errors. 


By setting up these middleware functions, we ensure our API is secure, well-structured, and prepared to handle JSON requests, manage cookies, support cross-origin requests, and log interactions. 

### Loading Private and Public Keys

To securely generate and verify JWT tokens, we use asymmetric encryption with an Elliptic Curve algorithm. This setup involves two keys: a private key for signing tokens and a public key for verifying them. The private key remains on the server, ensuring that only our backend can create tokens, while the public key is used to verify these tokens.

#### Step 1: Generate the Key Pair

First, we need to generate an Elliptic Curve (EC) key pair if we haven’t already. This can be done using OpenSSL:

```bash
openssl ecparam -genkey -name prime256v1 -noout -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem

```

This creates two files:

* `ec_private.pem`: the private key, used only by the server for signing tokens.
* `ec_public.pem:` the public key, which the server uses to verify tokens and can be shared with other services if needed.

Place these files in a secure `config` folder within your project's `api/` directory.

#### Step 2: Load Keys in the Code

We load these keys in `index.js` to be accessible for JWT token signing and verification. Here’s how it’s set up:

```JavaScript
import fs from 'fs';

// Load the private and public keys
const privateKey  = fs.readFileSync('config/ec_private.pem', 'utf8');
const publicKey   = fs.readFileSync('config/ec_public.pem', 'utf8');
```

* `fs.readFileSync:` This function reads the key files synchronously. Since these files are small and read only once during server startup, `readFileSync` is efficient and ensures the keys are loaded before any requests are handled.
* `privateKey:` This variable stores the private key for signing tokens. It’s essential to restrict access to this key, as any exposure would compromise the security of our token generation.
* `publicKey`: This stores the public key, which we use to verify the authenticity of JWTs created by our server.

#### Step 3: Using Keys for Token Management

With these keys loaded, we can now create functions for generating and verifying tokens:

* **Token Generation**: The `generateAccessToken` and `generateRefreshToken` functions in `index.js` use `jwt.sign()` to create JWTs, signing them with the private key and an expiration time.


```JavaScript
const generateAccessToken = (user) => {
  return jwt.sign({ id: user.id, username: user.username, role: user.role }, privateKey, { algorithm: 'ES256', expiresIn: accessTokenExpiresIn });
};

const generateRefreshToken = (user) => {
  return jwt.sign({ id: user.id, username: user.username, role: user.role }, privateKey, { algorithm: 'ES256', expiresIn: refreshTokenExpiresIn });
};
```

This asymmetric encryption setup with private and public keys helps ensure that only our server can create valid tokens, while verification can happen securely using the public key. Let's now continue and create the Middleware to be used to verify tokens and roles.

### Authentication and Authorization Middleware

Create a `api/authMiddleware.js` file in the API which will contain two essential middleware functions, `verifyToken` and `verifyRole`. These will work together to enforce authentication and authorization requirements for protected routes. 

```JavaScript
import jwt from 'jsonwebtoken';
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';

// Resolve __dirname equivalent
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const publicKeyPath = path.join(__dirname, '../config/ec_public.pem');
const publicKey = fs.readFileSync(publicKeyPath, 'utf8');

export const verifyToken = (req, res, next) => {

  const authHeader  = req.headers['authorization'];
  const token       = authHeader && authHeader.split(' ')[1]; 

  if (!token) return res.status(401).json({ message: 'Access token missing' });

  jwt.verify(token, publicKey, { algorithms: ['ES256'] }, (err, decoded) => {
    if (err) {
      return res.status(401).json({ message: 'Invalid or expired access token' });
    }

    req.user = decoded;
    next();
  });

};

export const verifyRole = (role) => {
  return (req, res, next) => {
    if (!req.user || req.user.role !== role) {
      return res.status(403).json({ message: 'Access forbidden: insufficient permissions' });
    }
    next();
  };
};

```

How `verifyToken` Works


1. Retrieve the Access Token: The middleware checks for the `Authorization` header in the request. If it exists, the token is extracted from it. If it’s missing, the middleware immediately returns a `401 Unauthorized` error, preventing access.

2. Verify the Token: Using `jsonwebtoken.verify`, the token is decoded and validated against the stored `publicKey`, which is loaded from a PEM file. The `ES256` algorithm is specified to match the one used when signing the token.

3. Handle Verification Results:
If the token is valid, the decoded payload (containing the user’s ID, role, and other information) is added to `req.user`, making it accessible to downstream route handlers.
If the token is invalid or expired, a `401 Unauthorized` response is sent to the client.

1. Proceed or Block: If verification is successful, the middleware calls `next()`, allowing the request to proceed to the next middleware or endpoint handler. If verification fails, access is denied.

By ensuring that each protected route runs `verifyToken`, only authenticated requests are allowed through. 


How `verifyRole` Works

The `verifyRole` middleware provides additional authorization by restricting access based on the user’s role. It checks if the user has the necessary permissions (e.g., `admin` role) to access certain routes, such as administrative settings.

The middleware first confirms that `req.user` exists, which indicates the user is authenticated and the `verifyToken` middleware has run successfully.
The user’s role (decoded from the token and stored in `req.user.role`) is then checked against the required role passed to `verifyRole`. For instance, if `verifyRole('admin')` is applied to a route, only users with an admin role can proceed.
If the user has the required role, `next()` is called, allowing access to the route.
If the user’s role doesn’t match the required role, a `403 Forbidden` response is sent, indicating insufficient permissions.

How the Middleware Functions Together

**Authentication:** `verifyToken` ensures the request comes from a logged-in user with a valid token.
**Authorization:** `verifyRole` provides a fine-grained control layer by checking if the authenticated user has the correct role to access the resource.
By chaining these middleware functions, we create a robust access control system. For example, an admin route could be protected like this:

```JavaScript
app.get('/admin/settings', verifyToken, verifyRole('admin'), (req, res) => {
  res.json({ message: "Welcome to the admin settings page!" });
});

```

In this setup:

1. The request is checked for a valid access token (`verifyToken`).
2. Only users with the `admin` role are allowed to proceed (`verifyRole('admin')`).

Together, these middlewares effectively secure protected resources, ensuring both authenticated access and appropriate authorization. 


### User Login Endpoint (/auth/login)

The `/auth/login` endpoint enables registered users to log into the application by verifying their credentials and, upon successful validation, issuing access and refresh tokens for ongoing authenticated access.

Here’s a step-by-step breakdown of the login process:

#### Step 1: Retrieve User Credentials
The endpoint expects `email` and `password` from the client in the request body. Upon receiving these credentials, the server tries to retrieve the user’s data from the database:

```JavaScript
const { email, password } = req.body;
const { data: users }     = await axios.get(apiUserEndpoint);
const user                = users.find(u => u.email === email);
```

If the provided email does not match any records, the server returns a `404 Not Found` response, indicating the user does not exist.

#### Step 2: Validate the Password
If the email exists in the database, the server compares the submitted password with the hashed password stored for the user. `bcrypt.compare()` is used to handle this:


```JavaScript
const isPasswordValid = await bcrypt.compare(password, user.password);
```

If the passwords do not match, the server responds with a `401 Unauthorized` status and an error message, as this indicates invalid credentials.

#### Step 3: Generate Tokens
Once authenticated, the server generates both an access token and a refresh token:

```JavaScript
const accessToken   = generateAccessToken(user);
const refreshToken  = generateRefreshToken(user);

```

* **Access Token**: Used for authentication on secured routes. This token contains user details and a role, expiring after a short period.
* **Refresh Token**: Allows the user to refresh the session without logging in again. This token is stored as an HTTP-only cookie, adding an extra layer of security.


#### Step 4: Send Tokens to the Client
The server sets the refresh token as a secure, HTTP-only cookie and returns the access token in the response:

```JavaScript
res.cookie('refresh-token', refreshToken, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: process.env.NODE_ENV === 'production' ? 'None' : 'Lax',
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
});
```

This configuration ensures the refresh token is accessible only through HTTP requests, minimizing the risk of XSS attacks. The `secure` and `sameSite` attributes provide additional security in production.

The response body includes the access token and user details (excluding the password) to enable immediate access on the client side:

```JavaScript
const userWithoutPsswd = _.pick(user, ['id', 'username', 'role', 'email', 'notifications']);
res.json({
  message: 'Login successful',
  accessToken,
  user: userWithoutPsswd
});

```

Having completed the functionality for the `/auth/login` endpoint, let's now proceed to discuss how our server will refresh tokens.

### Token Refresh Endpoint (`/auth/refresh-token`)

The `/auth/refresh-token` endpoint is responsible for generating a new access token for users whose current access token has expired but still possess a valid refresh token. This process allows users to continue using the application without having to re-authenticate.

Here's the code for this endpoint;

```JavaScript
app.post('/auth/refresh-token', (req, res) => {
        const refreshToken = req.cookies['refresh-token'];

        if (!refreshToken) {
          return res.status(401).json({ message: 'No refresh token provided' });
        }

        jwt.verify(refreshToken, publicKey, { algorithms: ['ES256'] }, (err, user) => {
          if (err) {
            return res.status(403).json({ message: 'Invalid or expired refresh token' });
          }

          const newAccessToken = generateAccessToken(user);

          res.json({
            accessToken: newAccessToken,
            user: user
          });
        });
});

```

Here’s how the endpoint works step-by-step:

#### Step 1: Retrieve and Validate the Refresh Token
The endpoint checks for the presence of a refresh token in the cookies. If the cookie is missing, the server responds with a `401 Unauthorized` error:


```JavaScript
const refreshToken = req.cookies['refresh-token'];

if (!refreshToken) {
  return res.status(401).json({ message: 'No refresh token provided' });
}
```

This setup ensures that only users who already have a valid session can attempt to refresh their tokens.

#### Step 2: Verify the Refresh Token
If a refresh token is present, the server verifies its validity using the public key and the ES256 algorithm. This ensures that the token has not expired and is untampered. If the token is invalid or expired, the server responds with a `403 Forbidden` error:

```JavaScript
jwt.verify(refreshToken, publicKey, { algorithms: ['ES256'] }, (err, user) => {
  if (err) {
    return res.status(403).json({ message: 'Invalid or expired refresh token' });
  }

  // Token is valid; proceed with issuing a new access token.
});

```

#### Step 3: Generate a New Access Token
Once the refresh token is verified, the server generates a new access token using the `generateAccessToken` function and the user data decoded from the refresh token:

```JavaScript
const newAccessToken = generateAccessToken(user);

```

The new access token enables the user to continue accessing protected resources without interruption.

#### Step 4: Send the New Access Token to the Client
The server sends the new access token in the response body to the client, along with user data for immediate use:


```JavaScript
res.json({
  accessToken: newAccessToken,
  user: user
});

```

The `/auth/refresh-token` endpoint is integral to maintaining a secure, uninterrupted user session by issuing new access tokens upon expiration. This setup minimizes the need for repeated logins and offers a seamless experience for authenticated users.




### Logout Endpoint (/auth/logout)


The `/auth/logout` endpoint is a straightforward endpoint designed to log users out securely by clearing their refresh token cookie. By removing this token, we prevent unauthorized access, as the refresh token is essential for generating new access tokens after the current one expires. Here’s a breakdown of how this endpoint operates:

When the `/auth/logout` endpoint is hit, the server clears the `refresh-token` cookie by setting its value to empty and its expiration to the past. This makes the cookie invalid for future requests, effectively logging the user out.

```JavaScript
res.clearCookie('refresh-token'); // Clear refresh token cookie on logout
```

The use of `httpOnly`, `secure`, and `sameSite` attributes, which were set when the cookie was created, ensures that the refresh token is inaccessible via client-side JavaScript and only transferred over HTTPS in production environments.



### Protecting Routes

To secure sensitive data, such as user profiles and application settings, we implement protected routes in our Express API. These routes require users to be authenticated, meaning only users with valid access tokens can access them. Additionally, we’ll implement role-based restrictions for administrative settings, ensuring only users with appropriate permissions can access specific resources.


In our application, we define two main protected routes:

1. User Profile Route (`/app/profile`): Requires the user to be authenticated and ensures they can only access their own profile.
2. Admin Settings Route (`/app/settings`): Requires both authentication and authorization, limiting access to users with an administrative role.

Let's explore the implementation of each route.

#### User Profile Endpoint (`/app/profile`)

The `/app/profile` endpoint serves authenticated users' profile data. In this route, we apply the `verifyToken` middleware, which we've discussed above, to check if the access token is valid. Then, we compare the `id` from the token payload with the requested user profile's `id` to ensure users can only view their own profile.

Here’s the code for the profile endpoint:

```JavaScript
app.get('/app/profile', verifyToken, async (req, res) => {
  const id = req.query.id;

  // Check if the requested profile matches the logged-in user's ID
  if (id !== req.user.id) {
    return res.status(403).json({ message: "Access forbidden: You can't view profiles you don't own." });
  }

  const { data: users } = await axios.get(apiUserEndpoint);
  const user = users.find(u => u.id === id);

  if (!user) {
    return res.status(404).send('User not found');
  }

  res.json({
    message: `Hello, ${user.username}!`,
    userId: user.id,
    role: user.role,
    favColor: user.favColor,
    username: user.username,
  });
});
```

In this code:

The `verifyToken` middleware checks the validity of the access token.
The `id` from the request is compared with the `id` in the token payload, preventing users from accessing other users' profiles.
If the `id` matches, the user’s profile data is retrieved and sent in the response. If not, a 403 Forbidden error is returned.

#### Admin Settings Endpoint (/app/settings)

The `/app/settings` endpoint is a protected route specifically for users with an administrative role. We apply both the `verifyToken` middleware (to check if the user is authenticated) and the `verifyRole`('admin') middleware (to ensure the user has admin privileges).

Here’s the code for the settings endpoint:

```JavaScript
app.get('/app/settings', verifyToken, verifyRole('admin'), (req, res) => {
  res.json({
    message: "Welcome to the admin settings page!",
    settings: {
      theme: "dark",
      notifications: true,
    },
  });
});

```

In this code:

* The `verifyToken` middleware confirms that the user is authenticated.
* The `verifyRole`('admin') middleware checks if the authenticated user has an admin role.
* If both conditions are met, the endpoint responds with the settings data. Otherwise, it returns a 403 error, preventing unauthorized users from accessing this resource.

That completes our setup of the back-end of our app. [Part II](/post/react-context-authentication-p2/). of this tutorial discusses the front-end.


###



```JavaScript

```
















































---------------------------------------------
