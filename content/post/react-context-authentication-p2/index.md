+++
author = "Amir Candido"
title = "Building a Robust Role-Based Authentication System With JWT Tokens & React Context. (Part II)"
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

In this part of the tutorial, we’ll build the front-end of our role-based authentication system using React. By leveraging JWT tokens and the React Context API, our goal is to securely manage authentication and authorization for different user roles.

Our front-end app will use React Router to handle routing and context for managing global state. With this setup, we’ll create protected routes for authorized users, ensure that access and refresh tokens are securely managed, and implement an efficient system for maintaining user sessions. Additionally, we'll cover how to create login and registration forms that communicate with our backend API to authenticate users and set up role-based access.

At the end of this section, you’ll have a robust front-end architecture that not only provides secure access to resources but also facilitates a seamless experience for users with different roles and permissions. Let's start by setting up the project structure and configuring our environment.

## 1. Setting Up the Project Structure

To start, we’ll organize the React front-end project within our main react-jwt-auth-app directory, which also contains our backend API. By keeping the front-end and back-end in the same overarching directory, we can manage both parts of the application more easily. Here’s how we’ll structure the front-end project and initialize it using Vite, which offers a fast development experience and optimized build setup for React.

### 1.1 Creating the React Project with Vite

#### 1. Navigate to the react-jwt-auth-app directory if you’re not already in it.

```bash
cd react-jwt-auth-app
```


#### 2. Create the React project using Vite with the following command:

```bash
npm create vite@latest client --template react
```

#### 3. Install necessary dependencies:

Move into the newly created front-end folder and install the dependencies:

```bash
cd client
npm install axios react-router-dom jwt-decode 
```

### 1.2 Project Folder Structure
Here’s the folder structure we’ll create within react-auth-app to keep the code organized:

```bash
react-jwt-auth-app/
├── api/                      # Backend (Node/Express) API
├── client/                   # Frontend React app
    ├── public/               # Public assets
    ├── src/
    │   ├── api/              # Axios requests and API helpers
    │   ├── components/       # Reusable components (e.g., buttons, forms)
    │   ├── context/          # React context for Auth and Role management
    │   ├── pages/            # Page components (e.g., Login, Register, Dashboard)
    │   ├── App.jsx           # Main App component
    │   ├── main.jsx          # Entry point
    │   └── config.js         # Environment-specific configuration (e.g., API URLs)
    └── vite.config.js        # Vite configuration

```

This structure separates the components, pages, and context into distinct folders, making the project easier to navigate and maintain as it grows.

### 1.3 Configuring Environment Variables
In the `react-jwt-auth-app/src/` folder, add a `config.js` file to define environment variables for the front-end application:


```Javascript
const config = {
    apiBaseUrl: 'http://localhost:3000',
    refreshTokenEndpoint: '/auth/refresh-token',
  };
  
  export default config;
```


With our project structure and environment variables in place, we’re ready to build the core features of our authentication system. The next steps involve creating the Auth Context, setting up secure API requests with Axios, and implementing user interface components for login and registration.





## 2. Creating the Authentication Context

In this section, we’ll set up an authentication context using React Context API, which will allow us to manage user authentication and roles across the application. By centralizing authentication logic in a context, we can easily access user state and authentication functions from any component without having to pass props deeply through the component tree.


### 2.1 Setting Up the AuthContext

The AuthContext allows us to manage user authentication data (like tokens and user details) across the app, making it accessible from any component without passing down props manually. In this file, we create the AuthContext and set up essential functions for authentication, token management, and state updates.

#### Creating the AuthContext

Let's start by creating a `client/src/context/AuthContext.jsx `file:

1. Creating the Context and Auth Provider: We start by defining `AuthContext` using React’s `createContext` and a custom hook `useAuth` for easy access. The `AuthProvider` component wraps the app, holding the main authentication state.



```Javascript
import { createContext, useContext, useState, useEffect, useMemo } from 'react';
import { axiosInstance } from '../api/apiService';
import config from '../config';

const AuthContext = createContext();
export const useAuth = () => useContext(AuthContext);

```

2. Defining the authState and authLoading State:

* `authState` holds the core authentication details: `isAuthenticated`, `accessToken`, and `user`.
* `authLoading` manages the loading state while the app checks or updates authentication status.


```Javascript
export const AuthProvider = ({ children }) => {
  const [authState, setAuthState] = useState({
    isAuthenticated: false,
    accessToken: null,
    user: null,
  });
  
  const [authLoading, setAuthLoading] = useState(true);
```


3. The `checkAuth` Function: The `checkAuth` function verifies and refreshes the authentication status when the app loads. It sends a request to the refresh token endpoint to renew the access token if available, then updates `authState` accordingly.


```Javascript
  const checkAuth = async () => {
    try {
      const response = await axiosInstance.post(config.refreshTokenEndpoint, {}, { withCredentials: true });
      setAuthState({
        isAuthenticated: true,
        accessToken: response.data.accessToken,
        user: response.data.user,
      });
    } catch (error) {
      console.error("Error refreshing token:", error);
      setAuthState({ isAuthenticated: false, user: null, accessToken: null });
    } finally {
      setAuthLoading(false);
    }
  };

```


* Automatic Check: The `useEffect` hook triggers `checkAuth` on the initial load to confirm if a valid session already exists.


```Javascript
  useEffect(() => {
    checkAuth();
  }, []);

```

4. The `login` and `logout` Functions:

* `login`: This function directly sets the user's authentication data upon successful login, updating the `authState`.
* `logout`: Clears all authentication details from the `authState` to securely log out the user.



```Javascript
  const login = ({ accessToken, user }) => {
    setAuthState({
      isAuthenticated: true,
      accessToken,
      user,
    });
  };

  const logout = () => {
    setAuthState({
      isAuthenticated: false,
      user: null,
      accessToken: null,
    });
  };

```


5. The `refreshToken` Function: The `refreshToken` function requests a new access token when the current token expires. It updates the `accessToken` in `authState` or logs the user out if the refresh attempt fails.



```Javascript
  const refreshToken = async () => {
    try {
      const response = await axiosInstance.post(config.refreshTokenEndpoint);
      setAuthState(prevState => ({
        ...prevState,
        accessToken: response.data.accessToken,
      }));
      return response.data.accessToken;
    } catch (error) {
      console.error("Token refresh failed:", error);
      logout();
    }
  };

```


6. Providing the Context Value: The `useMemo` hook ensures that we only recompute the value object when `authState` or `authLoading` changes, optimizing performance.


```Javascript
  const value = useMemo(
    () => ({
      ...authState,
      authLoading,
      login,
      logout,
      refreshToken,
    }),
    [authState, authLoading, login, logout, refreshToken]
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
```


This setup in `AuthContext` enables components to access the `user` state, check authentication status, and handle login/logout actions with ease. In the next section, we’ll cover how to create and protect routes based on this authentication context, ensuring secure and role-based access across the app.





## 3. Using Axios Interceptors to Maintain Session Integrity in JWT Auth


In this section, we'll look at how to configure Axios to ensure secure and seamless interactions between the client and server in our role-based authentication system. In our setup, the access token has a short lifespan to minimize exposure, while the refresh token, stored in an HTTP-only cookie, allows us to request a new access token without forcing the user to log in repeatedly.

To handle this, we’ll set up Axios interceptors, which play a key role in managing the lifecycle of tokens. By attaching the access token to each request, the interceptor ensures that protected routes are accessible only to authenticated users. Additionally, if an access token expires, the interceptor will attempt to refresh it using the refresh token, providing a smooth user experience without interruptions.

Through these interceptors, we automate token handling while maintaining robust session security.

Create the file `/src/api/apiService.tsx`; this will have the code we will use to manage secure API calls within the app.

Here, JWTs are used with two components: a short-lived access token that provides quick, verifiable access, and a refresh token, stored as an HTTP-only cookie with a longer expiration, to renew the access token as needed. This setup requires precise handling of API requests and responses to ensure that expired tokens don’t disrupt the user experience and that the user’s session remains secure.

The `apiService.tsx` file accomplishes this by configuring a custom axios instance with interceptors that handle token management tasks automatically, namely:


1. **Setting up Authorization Headers on Requests:** The access token must be added to the headers of ***each request***, allowing the server to authenticate the user for every API call.

2.  **Refreshing Tokens on Expired Access Tokens:** If the access token expires (indicated by a 401 Unauthorized response from the server), this file automatically triggers a token refresh using the refresh token.


3. **Error Handling and Session Management::** If refreshing fails, the interceptor logs the user out to prevent unauthorized access, enforcing session integrity.


Let’s walk through some code that achieves these functions:

### Key Components and Setup in apiService.tsx


1. Creating the axiosInstance:

```Javascript
export const axiosInstance = axios.create({
  baseURL: appConfig.apiBaseUrl,
});
```

The `axiosInstance` is initialized with a base URL from the app’s configuration, ensuring all API requests follow this centralized base path.


2. **Custom Hook:** `useAxiosWithRefresh`: This hook, which wraps the `axiosInstance`, sets up two types of interceptors—one for requests and one for responses. It relies on data from the `AuthContext` (i.e., the current `accessToken`, `refreshToken` method, and `logout` method) to manage token-related functionality.

By calling `useAuth`, it gains access to the latest authentication state and functions to help manage session persistence.

3. Request Interceptor:


```Javascript
useEffect(() => {
  const requestInterceptor = axiosInstance.interceptors.request.use(
    (config) => {
      if (accessToken) {
        config.headers.Authorization = `Bearer ${accessToken}`;
        config.withCredentials = true;
      }
      return config;
    },
    (error) => Promise.reject(error)
  );

  return () => {
    axiosInstance.interceptors.request.eject(requestInterceptor);
  };
}, [accessToken]);

```

* **Purpose:** The request interceptor adds the current `accessToken` as a Bearer token in the Authorization header of every outgoing request. It also enables `withCredentials`, allowing the browser to send the refresh token cookie with requests.
* **Update on Token Change:** By using the `accessToken` dependency in the `useEffect` hook, this interceptor is refreshed every time `accessToken` changes, ensuring the latest valid token is always used.

### 4. Response Interceptor for Token Refreshing:

```Javascript
useEffect(() => {
  const responseInterceptor = axiosInstance.interceptors.response.use(
    (response) => response,
    async (error) => {
      const originalRequest = error.config;
      
      if (error.response?.status === 401 && !originalRequest._retry) {
        originalRequest._retry = true;
        try {
          const newAccessToken = await refreshToken();
          if (!newAccessToken) {throw new Error("Token refresh failed");}

          originalRequest.headers['Authorization'] = `Bearer ${newAccessToken}`;
          return axiosInstance(originalRequest);
        } catch (refreshError) {
          console.error("Failed to refresh access token:", refreshError);
          logout();
          return Promise.reject(refreshError);
        }
      }
      
      return Promise.reject(error);
    }
  );

  return () => {
    axiosInstance.interceptors.response.eject(responseInterceptor);
  };
}, [refreshToken, logout]);


```

* **Purpose:** The response interceptor monitors for `401 Unauthorized` responses, which indicate an expired or invalid access token.
* **Automatic Token Refreshing:** If a 401 error is detected, it checks if the request has already been retried (by setting an _retry flag). If not, it calls the `refreshToken` function to obtain a new access token.
  
     - If `refreshToken` successfully returns a new access token, the interceptor updates the `Authorization` header and retries the original request with the new token.
     - If refreshing fails (e.g., the refresh token itself is invalid or expired), the `logout` function is triggered, logging the user out to prevent any unauthorized access attempts.

This response interceptor is refreshed on changes to `refreshToken` and logout to ensure it always uses the latest version of these functions.

### 5. **Returning the Enhanced `axiosInstance`:** 

Finally, `useAxiosWithRefresh` returns `axiosInstance`, now fully configured to manage tokens automatically across all requests. This customized instance can then be imported in any component or service file that needs to make authenticated API calls, removing the need to handle token logic at each request point manually.

### Summary
By using this `apiService.tsx` setup:
- The app can transparently manage the lifecycle of access and refresh tokens.
- The `useAxiosWithRefresh` hook automates error handling and re-authentication by leveraging interceptors, which improves code reusability and maintains a clean API call structure.
- This setup helps secure the app by automatically logging users out if their session can’t be refreshed, maintaining the integrity and security of the user’s experience. 

In the next steps, we can integrate this custom `axiosInstance` into various front-end services to interact with protected routes securely.


#### Providing the AuthContext

Next, we’ll use the `AuthProvider` to wrap the root of our application, making the `AuthContext` accessible to any component.

Wrap the App component:

In `src/main.jsx`, wrap the `App` component with `AuthProvider`:

```Javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import AppRouter from './AppRouter';  
import { AuthProvider } from './context/AuthContext';  
 

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
      <AuthProvider>
          <AppRouter /> 
      </AuthProvider>
  </React.StrictMode>
);
```

#### Using `AuthContext` in Components

Now that our `AuthContext` is set up, we can easily access authentication-related data and functions within any component using useContext.

#### Example: Accessing the `AuthContext` in `AppRouter`:

As a first example, let's see how `AppRouter` utilizes React Router alongside `AuthContext` to control access to routes, ensuring only authenticated users can view protected content.


```Javascript
import { createBrowserRouter, RouterProvider, Navigate } from "react-router-dom";
import { useAuth } from "./context/AuthContext"; 
import ErrorPage from "./pages/error-page";
import Home from "./pages/Home";
import Login from "./pages/auth/Login";
import Logout from "./pages/auth/Logout";
import Register from "./pages/auth/Register";
import AdminSettings from "./pages/AdminSettings";
import Userprofile from "./pages/Userprofile";
import Index from "./pages/Index";
import App from "./App";

const ProtectedRoute = ({ children }) => {
    const { isAuthenticated, authLoading } = useAuth(); 

    if (!isAuthenticated && !authLoading) {
      return <Navigate to="/login" replace />;
    }

  return children;  
};


const router = createBrowserRouter([
    {
      path: "/",
      element: <Home />,
    },
    {
      path: "/register",
      element: <Register />,
    },
    {
      path: "/login",
      element: <Login />,
    },
    {
      path: "/logout",
      element: <Logout />,
    },
    {
      path: "/app",
      element: (
        <ProtectedRoute>
          <App />
        </ProtectedRoute>
      ),
      children: [
        {
          
          errorElement: <ErrorPage />,  
          children: [
            { index: true, element: <Index /> },
            {
              path: "settings",
              element: <AdminSettings />,
            },
            {
              path: "profile",
              element: <Userprofile />
            },
          ],
        },
      ],
    },
]);

export default function AppRouter() {
  return <RouterProvider router={router} />;
}
```

Here's how `AppRouter` works in detail:

1. **Creating Routes**: The `AppRouter` defines routes for all main pages, including the landing page (`Home`), authentication pages (`Login`, `Register`, `Logout`), and app-specific pages like `AdminSettings` and `UserProfile`. Each route has a corresponding component, specifying what the user will see when they navigate to that path.

2. **ProtectedRoute Component**: To secure routes that require authentication, `ProtectedRoute` acts as a wrapper component. It checks the `isAuthenticated` status and `authLoading` flag from `AuthContext`. If the user is not authenticated and loading is complete, `ProtectedRoute` redirects them to the login page. Otherwise, it allows access to the protected route. This structure ensures that any attempt to directly access a protected route will redirect unauthenticated users to the login page, reinforcing session security.

3. **Nested Routes**: The `AppRouter` also implements nested routing. For example, the `/app` route includes child routes for specific sections like `settings` and `profile`. These nested routes provide a clean structure and allow the main `App` layout to remain consistent while users navigate between protected pages like `AdminSettings` and `UserProfile`.

By using `AuthContext` within `ProtectedRoute`, `AppRouter` dynamically manages user access based on their authentication state, streamlining the process of securing pages and ensuring only the correct users can view sensitive content. 



#### Example: Accessing the AuthContext in Login:


Just as we did in the `AppRouter`, we can use the `useContext` hook in other components to access values that the `AuthContext` makes available to child components. 
The Login component uses the AuthContext's `login` to update the user's login state:



```Javascript
import { useState, useCallback } from "react";
import { useAuth } from '../../context/AuthContext';
import { Link } from "react-router-dom";
import { axiosInstance } from  "../../api/apiService";

/*Other imports...*/

export default function Login() {

      const { login } = useAuth();

      const handleClickShowPassword = useCallback(() => {
        setShowPassword((show) => !show);
      }, []);  
      
      const [errors, setErrors]             = useState(null);
      const [showPassword, setShowPassword] = useState(false);
      const navigate                        = useNavigate();
      const [isLoading, setisLoading]       = useState(false);

      const formik = useFormik({
        initialValues: {
          password: "",
          email: "",          
        },
        validationSchema: Yup.object({
          password:       Yup.string().required("Required"),
          email:          Yup.string().email("Invalid email address").required("Required"),          
        }),
        onSubmit: async (values) => {
          setisLoading(true);
          try {
            const res = await axiosInstance.post( "/auth/login", values, { withCredentials: true });
            const { accessToken, user  }  = res.data;
            login({ user, accessToken });    
            navigate('/app');
          } catch (err) {
            const errorMessages = err?.response?.data?.errors || [{ msg: "Something went wrong. Please try again." }];
            setErrors(errorMessages);     
          }finally {
            setisLoading(false);
          }
        },
      });

      return (
        <Container component="main" maxWidth="xs" disableGutters>
            /*Login Form*/              
        </Container>  

      );
}      

```

In this example, by calling `useAuth()`, the `Login` component gains access to the `login` function, which is part of our `AuthContext`. This function is used to update the global authentication state once the user successfully logs in.

Here’s how it works step-by-step:

1. Form Handling: The component utilizes Formik for form handling and validation, making it easy to capture the user’s email and password while checking for required fields.

2. Submit Logic: On form submission, the component sends a `POST` request to the `login` endpoint (/auth/login) using our configured axiosInstance. We pass the user’s credentials (email and password) with the request.

3. Response Handling: If the login is successful, the API responds with an `accessToken` and user data. We then call `login({ user, accessToken })`, which updates the `AuthContext` to reflect the user’s authenticated state, including storing the access token and user information.

4. Navigation and Error Handling: Upon a successful login, we redirect the user to a protected route (e.g., /app). If an error occurs, we capture and display the error message to the user.

This structure ensures that the authentication state is shared globally and consistently across the app, while allowing components like `Login` to handle form submission and error handling locally. By centralizing login within `AuthContext`, any component in the app can access the current authentication state or trigger login actions, promoting reusability and cleaner code.

#### Example: Accessing the `AuthContext` in a Custom Hook:

Finally, let's see how the `AuthContext` can be used in a custom hook, `useAdminSettings`, which in turn will be used by our `AdminSettings` page. Remember from our AppRouter that the `AdminSettings` component is an element of the protected route `app/settings` and that, from our API code this route uses both the `verifyToken` and `verifyRole` middleware, thus ensuring that the user is both authenticated and has the necessary authorization to access this route. 

```Javascript
import { useState, useEffect } from "react";
import useAxiosWithRefresh from "./apiService";
import { useAuth } from "../context/AuthContext";

export function useAdminSettings() {
  const { user }                            = useAuth(); 
  const axiosInstance                       = useAxiosWithRefresh();
  const [adminSettings, setAdminSettings]   = useState(null);
  const [error, setError]                   = useState(null);
  const [loading, setLoading]               = useState(true);  

  useEffect(() => {
    const fetchAdminSettings = async () => {
      try {
        setLoading(true);  
        const response = await axiosInstance.get('/app/settings', { params: { id: user.id } });
        setAdminSettings(response.data);
        setError(null);  
      } catch (err) {
        setError(err);  
      } finally {
        setLoading(false); 
      }
    };

    fetchAdminSettings();
  }, [axiosInstance, user.id]);

  return { adminSettings, error, loading };  
}

```

The `useAdminSettings` custom hook leverages `AuthContext` and an Axios instance with token refresh capabilities (`useAxiosWithRefresh`) to fetch and manage admin settings data, securing the retrieval process with authentication checks. Here’s how it works in detail:

1. **Using AuthContext for User Data**: 
   The hook imports `useAuth` to access the `user` object from `AuthContext`. This allows `useAdminSettings` to access properties like `user.id`, ensuring that the API request is specific to the logged-in user. Without the `user.id`, the API request could not personalize or secure the fetched settings data based on the authenticated user.

2. **Secure Axios Instance**: 
   The hook calls `useAxiosWithRefresh`, which provides an Axios instance configured with interceptors for token management. This instance will automatically refresh the access token if it expires, enhancing security and session continuity. As a result, `useAdminSettings` can focus on data fetching logic without worrying about token expiration, and any invalid token scenarios are handled seamlessly.

3. **Fetching Admin Settings Data**:
   Within the `useEffect` hook, the `fetchAdminSettings` function is defined and executed on component mount. It initiates a GET request to the `/app/settings` endpoint, sending the user’s ID as a parameter for personalized data retrieval. Since the hook is dependent on `user.id`, any change in the authenticated user’s ID will trigger a re-fetch, keeping data in sync with the current user.

4. **State Management**:
   - `adminSettings`: Stores the fetched settings data.
   - `error`: Catches any errors that occur during the request.
   - `loading`: Manages the loading state, providing feedback to the UI.

5. **Return Values**:
   The hook returns `adminSettings`, `error`, and `loading` to the component that invokes it, allowing that component to conditionally render content based on the loading state, show error messages if necessary, and display the admin settings once available.

In summary, `useAdminSettings` effectively uses `AuthContext` for user-specific data, combines it with a secure Axios instance, and manages the asynchronous fetching of admin settings, providing a straightforward, reusable hook for authenticated data retrieval in the application. This approach encapsulates logic for handling secure API requests, maintaining session integrity, and updating the UI accordingly.



---------------------------------------------
