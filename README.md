export const msalConfig = {
  auth: {
    clientId: "d7b1c114-0856-474a-b1f2-cf517c4b1071",
    authority:
      "https://login.microsoftonline.com/e0e80fc4-4c12-4245-8bfd-c06791ad303a",
    redirectUri: "https://d3icudfrgx44qe.cloudfront.net/index.html", // Ensure this is the correct URL
  },
  cache: {
    cacheLocation: "sessionStorage", // This will ensure the session is maintained during the browser session
    storeAuthStateInCookie: false,
  },
};
export const loginRequest = {
  scopes: ["User.Read"],
};





this is index.js

// import React from "react";
// import ReactDOM from "react-dom/client";
// import "./index.css";
// import App from "./App";
// import reportWebVitals from "./reportWebVitals";

// const root = ReactDOM.createRoot(document.getElementById("root"));
// root.render(
//   <React.StrictMode>
//     <App />
//   </React.StrictMode>
// );

import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { PublicClientApplication, EventType } from "@azure/msal-browser";
import { msalConfig } from "./authConfig";
console.log("index.js is running");
const msalInstance = new PublicClientApplication(msalConfig);
console.log("Initial active account:", msalInstance.getActiveAccount());
if (
  !msalInstance.getActiveAccount() &&
  msalInstance.getAllAccounts().length > 0
) {
  console.log("Setting active account to the first account in all accounts");
  msalInstance.setActiveAccount(msalInstance.getAllAccounts()[0]);
}
msalInstance.addEventCallback((event) => {
  if (event.eventType === EventType.LOGIN_SUCCESS && event.payload.account) {
    console.log("Login success event received. Setting active account.");
    msalInstance.setActiveAccount(event.payload.account);
  } else if (event.eventType === EventType.LOGIN_FAILURE) {
    console.error("Login failure event received:", event.error);
  } else {
    console.log("Event received:", event);
  }
});
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App instance={msalInstance} />);


this is app.js

// import React from "react";
// import "./App.css";
// import Dashboard from "./Dashboard";
// import Header from "./Header";
// import Footer from "./Footer";
// function App() {
//   return (
//     <div className="App">
//       <header>{/* <h1>QuickSight Dashboard</h1> */}</header>
//       <main>
//         <Header />
//         <Dashboard />
//         <Footer />
//       </main>
//     </div>
//   );
// }
// export default App;

import React, { useEffect, useState } from "react";
import "./App.css";
import Dashboard from "./Dashboard";
import Header from "./Header";
import Footer from "./Footer";
import {
  AuthenticatedTemplate,
  UnauthenticatedTemplate,
  useMsal,
  MsalProvider,
} from "@azure/msal-react";
import { loginRequest } from "./authConfig";
const WrappedView = () => {
  const { instance, accounts } = useMsal();
  const [accessToken, setAccessToken] = useState(null);
  useEffect(() => {
    const account = accounts[0];
    if (account) {
      instance
        .acquireTokenSilent({
          ...loginRequest,
          account: account,
        })
        .then((response) => {
          setAccessToken(response.accessToken);
        })
        .catch((error) => {
          console.error("Error acquiring token silently:", error);
        });
    }
  }, [instance, accounts]);
  const handleLoginPopup = () => {
    instance
      .loginPopup(loginRequest)
      .then((response) => {
        setAccessToken(response.accessToken);
      })
      .catch((error) => {
        console.error("Login Popup Error:", error);
      });
  };
  const handleLogout = () => {
    instance.logoutPopup().catch((error) => {
      console.error("Logout Error:", error);
    });
  };
  return (
    <div className="App">
      <AuthenticatedTemplate>
        {accessToken ? (
          <div>
            <Header onLogout={handleLogout} />
            <Dashboard />
            <Footer />
          </div>
        ) : (
          <p>Loading...</p>
        )}
      </AuthenticatedTemplate>
      <UnauthenticatedTemplate>
        <div className="welcome-container">
          <h1>Welcome to the Wallboard</h1>
          <p>Please click on the button below to sign in:</p>
          <button className="sign-in-button" onClick={handleLoginPopup}>
            Sign in
          </button>
        </div>
      </UnauthenticatedTemplate>
    </div>
  );
};
const App = ({ instance }) => {
  return (
    <MsalProvider instance={instance}>
      <WrappedView />
    </MsalProvider>
  );
};
export default App;

