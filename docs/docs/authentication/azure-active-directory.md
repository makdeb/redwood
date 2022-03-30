### Azure Active Directory

+++ View Installation and Setup

#### Installation

The following CLI command will install required packages and generate boilerplate code and files for Redwood Projects:

```terminal
yarn rw setup auth azureActiveDirectory
```

_If you prefer to manually install the package and add code_, run the following command and then add the required code provided in the next section.

```bash
cd web
yarn add @azure/msal-browser
```

#### Setup

To get your application credentials, create an [App Registration](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-app-registration) using in your Azure Active Directory tenant and make sure you configure as a [MSAL.js 2.0 with auth code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-spa-app-registration#redirect-uri-msaljs-20-with-auth-code-flow) registration. Take a note of your generated _Application ID_ (client), and the _Directory ID_ (tenant).

[Learn more about authorization code flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/reference-third-party-cookies-spas).

##### Redirect URIs

Enter allowed redirect urls for the integrations, e.g. `http://localhost:8910/login`. This will be the `AZURE_ACTIVE_DIRECTORY_REDIRECT_URI` environment variable, and suggestively `AZURE_ACTIVE_DIRECTORY_LOGOUT_REDIRECT_URI`.

#### Authority

The Authority is a URL that indicates a directory that MSAL can request tokens from which you can read about [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/msal-client-application-configuration#authority). However, you most likely want to have e.g. `https://login.microsoftonline.com/<tenant>` as Authority URL, where `<tenant>` is the Azure Active Directory tenant id. This will be the `AZURE_ACTIVE_DIRECTORY_AUTHORITY` environment variable.

```js
// web/src/App.js
import { AuthProvider } from '@redwoodjs/auth'
import { PublicClientApplication } from '@azure/msal-browser'
import { FatalErrorBoundary } from '@redwoodjs/web'
import { RedwoodApolloProvider } from '@redwoodjs/web/apollo'

import FatalErrorPage from 'src/pages/FatalErrorPage'
import Routes from 'src/Routes'

import './index.css'

const azureActiveDirectoryClient = new PublicClientApplication({
  auth: {
    clientId: process.env.AZURE_ACTIVE_DIRECTORY_CLIENT_ID,
    authority: process.env.AZURE_ACTIVE_DIRECTORY_AUTHORITY,
    redirectUri: process.env.AZURE_ACTIVE_DIRECTORY_REDIRECT_URI,
    postLogoutRedirectUri: process.env.AZURE_ACTIVE_DIRECTORY_LOGOUT_REDIRECT_URI,
  },
})

const App = () => (
  <FatalErrorBoundary page={FatalErrorPage}>
    <AuthProvider client={azureActiveDirectoryClient} type="azureActiveDirectory">
      <RedwoodApolloProvider>
        <Routes />
      </RedwoodApolloProvider>
    </AuthProvider>
  </FatalErrorBoundary>
)

export default App
```

#### Roles

To setup your App Registration with custom roles and have them exposed via the `roles` claim, follow [this documentation](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps).

#### Login Options

Options in method `logIn(options?)` is of type [RedirectRequest](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_browser.html#redirectrequest) and is a good place to pass in optional [scopes](https://docs.microsoft.com/en-us/graph/permissions-reference#user-permissions) to be authorized. By default, MSAL sets `scopes` to [/.default](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent#the-default-scope) which is built in for every application that refers to the static list of permissions configured on the application registration. Furthermore, MSAL will add `openid` and `profile` to all requests. In example below we explicit include `User.Read.All` to the login scope.

```js
await logIn({
  scopes: ['User.Read.All'], // becomes ['openid', 'profile', 'User.Read.All']
})
```

See [loginRedirect](https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html#loginredirect), [PublicClientApplication](https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html) class and [Scopes Behavior](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/docs/scopes.md#scopes-behavior) for more documentation.

#### getToken Options

Options in method `getToken(options?)` is of type [RedirectRequest](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_browser.html#redirectrequest). By default, `getToken` will be called with scope `['openid', 'profile']`. As Azure Active Directory apply [incremental consent](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/resources-and-scopes.md#dynamic-scopes-and-incremental-consent), we can extend the permissions from the login example by including another scope, for example `Mail.Read`.

```js
await getToken({
  scopes: ['Mail.Read'], // becomes ['openid', 'profile', 'User.Read.All', 'Mail.Read']
})
```

See [acquireTokenSilent](https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html#acquiretokensilent), [Resources and Scopes](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/resources-and-scopes.md#resources-and-scopes) or [full class documentation](https://pub.dev/documentation/msal_js/latest/msal_js/PublicClientApplication-class.html#constructors) for more documentation.

+++