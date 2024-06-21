# Authentication workshop

This workshop is for EPITA SIGL 2025's students.

**Objective of the workshop**: Handle users by:

- adding a logging page feature to Sotracteur's frontend
- securing access to Sotracteur's web services (meaning a user needs to be logged in to request Sotracteur's web service)

## Step 1: Setup Login to Sotracteur

This step was made in class.

To sum up, you should have:

- use [auth0](https://auth0.com) to manage authentication on stotracteur
- created a tenant in auth0 like `groupXX-sotracteur.eu.auth0.com`
- created an application in Auth0 and configured the `<Auth0Context.Provder />` react components
- Made use of an `<Authenticated />` custom react component to redirect the user in case it's not logged-in (using `useAuth0` with its method `loginWithRedirect()`)

## Step 2: Secure your web service

**Objective**: protect your web service so that **only** logged-in users can query your web service.

### **How?**

The idea is to get a token (called a `Bearer` token) provided by the React hook `useAuth0()` and add it in the headers of your `fetch` calls from your frontend.

### Example: Search tractors (with auth)

#### Secure your web service

- Go to your Auth0 dashboard on [auth0.com](https://auth0.com) and create a new `API` under application
- name don't matter, you can chose your own
- Follow the quickstart in `NodeJS express` and adapt your web service to add security on it
- Make sure your `Auth0Context.Provider`'s component props contains the `audience` of the newly created `API` in auth0.

#### Make authenticated call from frontend

Ex: Let's consider the `SearchTractor` React component that queries tractors from your backend.

It could look like the following:

> Note: your web service should be running on localhost:3000

```jsx
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";

export function SearchTractor() {
  const [data, setData] = React.useState(null);
  const [error, setError] = React.useState(null);
  const {getAccessTokenSilently} = useAuth0();
  React.useEffect(() => {
    async function getTractors() {
        try {
            const position = { latitude: 48.8583145, longitude: 2.292334 };
            const token = await getAccessTokenSilently();
            const response = await fetch(
                `http://localhost:3000/v1/tractors??latitude=${latitude}&longitude=${longitude}&radius=50`,
                headers: {
                    "Content-Type": "application/json",
                    "Authorization": `Bearer ${token}` // Here is the protection, w/o this token, you can't call the protected web service
                }
            );
            const tractors = await response.json();
            setData(tractors);
        } catch (err) {
            setError(err)
        }
    }
    getTractors(); // we  need this weird call, because a use effect callback can't be async...

  }, []) // [] this means that the useEffect's callback will be called only once when SearchTractor will be mounted
  if (error) {
    return <span>Erreur: {error}</span>
  }
  return data && Array.isArray(data) && data.length > 0 ? (data.map({modele} => (
    <li>Modèle du tracteur: {modele}</li>
  ))) : <span>Pas de tracteurs</span>;
}
```

Adapt your own code to fetch your web service correctly in your components.

## Challenge: add RBAC to your users and web service

**Objective**: Depending on a user with role `agriculteur`:

- you will have a new `Créer une location` menu item
- when navigating to the menu item `Créer une location`, the user will send a `GET` to `/v1/renting` and retreive a message like `{"userRole": "agriculteur"}`
  - this `/v1/renting` route will return a `403` (Unauthorized) if the user **do not have `agriculteur` role**.
