<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b5bf5-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="b5bf5-102">Cette opération est obligatoire pour obtenir le jeton d’accès OAuth nécessaire pour appeler l’API Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="b5bf5-103">Dans cette étape, vous allez intégrer l’intermédiaire OWIN et la bibliothèque bibliothèque d’authentification [Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client/) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

1. <span data-ttu-id="b5bf5-104">Cliquez avec le bouton droit sur le projet de didacticiel **graphique** dans l’Explorateur de solutions et **sélectionnez Ajouter > nouvel élément...**. Choose **Web Configuration File**, name the file and select `PrivateSettings.config` **Add**.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-104">Right-click the **graph-tutorial** project in Solution Explorer and select **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and select **Add**.</span></span> <span data-ttu-id="b5bf5-105">Remplacez l'ensemble de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-105">Replace its entire contents with the following code.</span></span>

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    <span data-ttu-id="b5bf5-106">Remplacez-le par l’ID de l’application à partir du portail d’inscription des applications et par la `YOUR_APP_ID_HERE` secret client que vous avez `YOUR_APP_PASSWORD_HERE` générée.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="b5bf5-107">Si votre secret client contient des signes esperluette (`&`), n’oubliez pas de les remplacer par des `&amp;` dans `PrivateSettings.config`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="b5bf5-108">Assurez-vous également de modifier la valeur `PORT` en `ida:RedirectUri` afin qu’elle corresponde à l’URL de votre application.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="b5bf5-109">Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier du contrôle source afin d’éviter toute fuite accidentelle de votre ID d’application et de votre mot de `PrivateSettings.config` passe.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="b5bf5-110">Mise `Web.config` à jour pour charger ce nouveau fichier.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="b5bf5-111">Remplacez `<appSettings>` (ligne 7) par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-111">Replace the `<appSettings>` (line 7) with the following</span></span>

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="b5bf5-112">Implémentation de la connexion</span><span class="sxs-lookup"><span data-stu-id="b5bf5-112">Implement sign-in</span></span>

<span data-ttu-id="b5bf5-113">Commencez par initialiser l’intergiciel OWIN pour utiliser l’authentification Azure AD pour l’application.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span>

1. <span data-ttu-id="b5bf5-114">Cliquez avec le bouton **droit sur App_Start** dossier dans l’Explorateur de solutions et **sélectionnez Ajouter > classe...**. Nommez le fichier `Startup.Auth.cs` et sélectionnez **Ajouter.**</span><span class="sxs-lookup"><span data-stu-id="b5bf5-114">Right-click the **App_Start** folder in Solution Explorer and select **Add > Class...**. Name the file `Startup.Auth.cs` and select **Add**.</span></span> <span data-ttu-id="b5bf5-115">Remplacez tout le contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-115">Replace the entire contents with the following code.</span></span>

    ```cs
    using Microsoft.Identity.Client;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.IdentityModel.Tokens;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.Notifications;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Owin;
    using System.Configuration;
    using System.Threading.Tasks;
    using System.Web;

    namespace graph_tutorial
    {
        public partial class Startup
        {
            // Load configuration settings from PrivateSettings.config
            private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
            private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
            private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
            private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

            public void ConfigureAuth(IAppBuilder app)
            {
                app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

                app.UseCookieAuthentication(new CookieAuthenticationOptions());

                app.UseOpenIdConnectAuthentication(
                    new OpenIdConnectAuthenticationOptions
                    {
                        ClientId = appId,
                        Authority = "https://login.microsoftonline.com/common/v2.0",
                        Scope = $"openid email profile offline_access {graphScopes}",
                        RedirectUri = redirectUri,
                        PostLogoutRedirectUri = redirectUri,
                        TokenValidationParameters = new TokenValidationParameters
                        {
                            // For demo purposes only, see below
                            ValidateIssuer = false

                            // In a real multi-tenant app, you would add logic to determine whether the
                            // issuer was from an authorized tenant
                            //ValidateIssuer = true,
                            //IssuerValidator = (issuer, token, tvp) =>
                            //{
                            //  if (MyCustomTenantValidation(issuer))
                            //  {
                            //    return issuer;
                            //  }
                            //  else
                            //  {
                            //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                            //  }
                            //}
                        },
                        Notifications = new OpenIdConnectAuthenticationNotifications
                        {
                            AuthenticationFailed = OnAuthenticationFailedAsync,
                            AuthorizationCodeReceived = OnAuthorizationCodeReceivedAsync
                        }
                    }
                );
            }

            private static Task OnAuthenticationFailedAsync(AuthenticationFailedNotification<OpenIdConnectMessage,
              OpenIdConnectAuthenticationOptions> notification)
            {
                notification.HandleResponse();
                string redirect = $"/Home/Error?message={notification.Exception.Message}";
                if (notification.ProtocolMessage != null && !string.IsNullOrEmpty(notification.ProtocolMessage.ErrorDescription))
                {
                    redirect += $"&debug={notification.ProtocolMessage.ErrorDescription}";
                }
                notification.Response.Redirect(redirect);
                return Task.FromResult(0);
            }

            private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
            {
                var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                    .WithRedirectUri(redirectUri)
                    .WithClientSecret(appSecret)
                    .Build();

                string message;
                string debug;

                try
                {
                    string[] scopes = graphScopes.Split(' ');

                    var result = await idClient.AcquireTokenByAuthorizationCode(
                        scopes, notification.Code).ExecuteAsync();

                    message = "Access token retrieved.";
                    debug = result.AccessToken;
                }
                catch (MsalException ex)
                {
                    message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
                    debug = ex.Message;
                }

                notification.HandleResponse();
                notification.Response.Redirect($"/Home/Error?message={message}&debug={debug}");
            }
        }
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="b5bf5-116">Ce code configure l’middleware OWIN avec les valeurs de et définit deux méthodes de `PrivateSettings.config` rappel, `OnAuthenticationFailedAsync` et `OnAuthorizationCodeReceivedAsync` .</span><span class="sxs-lookup"><span data-stu-id="b5bf5-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="b5bf5-117">Ces méthodes de rappel sont appelées lorsque le processus de connexion renvoie à partir d’Azure.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

1. <span data-ttu-id="b5bf5-118">Maintenant, mettez à `Startup.cs` jour le fichier pour appeler la `ConfigureAuth` méthode.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="b5bf5-119">Remplacez l’intégralité `Startup.cs` du contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

    ```cs
    using Microsoft.Owin;
    using Owin;

    [assembly: OwinStartup(typeof(graph_tutorial.Startup))]

    namespace graph_tutorial
    {
        public partial class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                ConfigureAuth(app);
            }
        }
    }
    ```

1. <span data-ttu-id="b5bf5-120">Ajoutez une action `Error` à la classe `HomeController` pour transformer le `message` et `debug` les paramètres de requête en un objet `Alert`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="b5bf5-121">Ouvrez `Controllers/HomeController.cs` et ajoutez la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. <span data-ttu-id="b5bf5-122">Ajouter un contrôleur pour gérer la connexion.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="b5bf5-123">Faites un clic droit sur le dossier **Contrôleurs** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Contrôleur...**. Sélectionnez **Contrôleur MVC 5 - Vide**, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-123">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="b5bf5-124">Nommez le contrôleur `AccountController`, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-124">Name the controller `AccountController` and select **Add**.</span></span> <span data-ttu-id="b5bf5-125">Remplacez tout le contenu du fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-125">Replace the entire contents of the file with the following code.</span></span>

    ```cs
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using System.Security.Claims;
    using System.Web;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public class AccountController : Controller
        {
            public void SignIn()
            {
                if (!Request.IsAuthenticated)
                {
                    // Signal OWIN to send an authorization request to Azure
                    Request.GetOwinContext().Authentication.Challenge(
                        new AuthenticationProperties { RedirectUri = "/" },
                        OpenIdConnectAuthenticationDefaults.AuthenticationType);
                }
            }

            public ActionResult SignOut()
            {
                if (Request.IsAuthenticated)
                {
                    Request.GetOwinContext().Authentication.SignOut(
                        CookieAuthenticationDefaults.AuthenticationType);
                }

                return RedirectToAction("Index", "Home");
            }
        }
    }
    ```

    <span data-ttu-id="b5bf5-126">Cela définit une action `SignIn` et une action `SignOut`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="b5bf5-127">L’action `SignIn` vérifie si la demande est déjà authentifiée.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="b5bf5-128">Si ce n’est pas le cas, il appelle l’intergiciel OWIN pour authentifier l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="b5bf5-129">L’action `SignOut` appelle l’intergiciel OWIN pour se déconnecter.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

1. <span data-ttu-id="b5bf5-130">Enregistrez vos modifications et démarrez le projet.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-130">Save your changes and start the project.</span></span> <span data-ttu-id="b5bf5-131">Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="b5bf5-132">Connectez-vous avec votre compte Microsoft et consentez aux autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="b5bf5-133">Le navigateur vous redirige vers l’application, affichant le jeton.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="b5bf5-134">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="b5bf5-134">Get user details</span></span>

<span data-ttu-id="b5bf5-135">Une fois que l’utilisateur a ouvert une session, vous pouvez obtenir ses informations à partir de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-135">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="b5bf5-136">Faites un clic droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-136">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="b5bf5-137">Nommer le dossier `Helpers`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-137">Name the folder `Helpers`.</span></span>

1. <span data-ttu-id="b5bf5-138">Cliquez avec le bouton droit sur ce nouveau dossier et **sélectionnez > classe...**. Nommez le fichier `GraphHelper.cs` et sélectionnez **Ajouter.**</span><span class="sxs-lookup"><span data-stu-id="b5bf5-138">Right click this new folder and select **Add > Class...**. Name the file `GraphHelper.cs` and select **Add**.</span></span> <span data-ttu-id="b5bf5-139">Remplacez le contenu du fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-139">Replace the contents of this file with the following code.</span></span>

    ```cs
    using Microsoft.Graph;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace graph_tutorial.Helpers
    {
        public static class GraphHelper
        {
            public static async Task<User> GetUserDetailsAsync(string accessToken)
            {
                var graphClient = new GraphServiceClient(
                    new DelegateAuthenticationProvider(
                        (requestMessage) =>
                        {
                            requestMessage.Headers.Authorization =
                                new AuthenticationHeaderValue("Bearer", accessToken);
                            return Task.FromResult(0);
                        }));

                return await graphClient.Me.Request().GetAsync();
            }
        }
    }
    ```

    <span data-ttu-id="b5bf5-140">Cette opération implémente la fonction `GetUserDetails`, qui utilise le kit de développement logiciel Microsoft Graph pour appeler le point de terminaison `/me` et renvoyer le résultat.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="b5bf5-141">Mettez à `OnAuthorizationCodeReceivedAsync` jour la méthode pour appeler cette `App_Start/Startup.Auth.cs` fonction.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="b5bf5-142">Ajoutez l’instruction `using` suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-142">Add the following `using` statement to the top of the file.</span></span>

    ```cs
    using graph_tutorial.Helpers;
    ```

1. <span data-ttu-id="b5bf5-143">Remplacez le bloc `try` existant dans `OnAuthorizationCodeReceivedAsync` par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

    ```cs
    try
    {
        string[] scopes = graphScopes.Split(' ');

        var result = await idClient.AcquireTokenByAuthorizationCode(
            scopes, notification.Code).ExecuteAsync();

        var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

        string email = string.IsNullOrEmpty(userDetails.Mail) ?
            userDetails.UserPrincipalName : userDetails.Mail;

        message = "User info retrieved.";
        debug = $"User: {userDetails.DisplayName}, Email: {email}";
    }
    ```

1. <span data-ttu-id="b5bf5-144">Enregistrer vos modifications et démarrer l’application. Après vous être connecté, vous devriez voir le nom et l’adresse de messagerie de l’utilisateur au lieu du jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-144">Save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="b5bf5-145">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="b5bf5-145">Storing the tokens</span></span>

<span data-ttu-id="b5bf5-146">Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="b5bf5-147">Étant donné qu’il s’agit d’un exemple d’application, vous utiliserez la session pour stocker les jetons.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-147">Since this is a sample app, you will use the session to store the tokens.</span></span> <span data-ttu-id="b5bf5-148">Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span> <span data-ttu-id="b5bf5-149">Dans cette section, vous allez :</span><span class="sxs-lookup"><span data-stu-id="b5bf5-149">In this section, you will:</span></span>

- <span data-ttu-id="b5bf5-150">Implémenter une classe stockage de jetons pour sérialiser et stocker le cache de jetons MSAL et les détails de l’utilisateur dans la session utilisateur.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-150">Implement a token store class to serialize and store the MSAL token cache and the user's details in the user session.</span></span>
- <span data-ttu-id="b5bf5-151">Mettre à jour le code d’authentification pour utiliser la classe stockage de jetons.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-151">Update the authentication code to use the token store class.</span></span>
- <span data-ttu-id="b5bf5-152">Mettre à jour la classe contrôleur de base pour exposer les détails stockés de l’utilisateur, à tous les affichages dans l’application.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-152">Update the base controller class to expose the stored user details to all views in the application.</span></span>

1. <span data-ttu-id="b5bf5-153">Faites un clic droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-153">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="b5bf5-154">Nommer le dossier `TokenStorage`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-154">Name the folder `TokenStorage`.</span></span>

1. <span data-ttu-id="b5bf5-155">Cliquez avec le bouton droit sur ce nouveau dossier et **sélectionnez > classe...**. Nommez le fichier `SessionTokenStore.cs` et sélectionnez **Ajouter.**</span><span class="sxs-lookup"><span data-stu-id="b5bf5-155">Right click this new folder and select **Add > Class...**. Name the file `SessionTokenStore.cs` and select **Add**.</span></span> <span data-ttu-id="b5bf5-156">Remplacez le contenu du fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-156">Replace the contents of this file with the following code.</span></span>

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT license.

    using Microsoft.Identity.Client;
    using Newtonsoft.Json;
    using System.Security.Claims;
    using System.Threading;
    using System.Web;

    namespace graph_tutorial.TokenStorage
    {
        // Simple class to serialize into the session
        public class CachedUser
        {
            public string DisplayName { get; set; }
            public string Email { get; set; }
            public string Avatar { get; set; }
        }

        public class SessionTokenStore
        {
            private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

            private HttpContext httpContext = null;
            private string tokenCacheKey = string.Empty;
            private string userCacheKey = string.Empty;

            public SessionTokenStore(ITokenCache tokenCache, HttpContext context, ClaimsPrincipal user)
            {
                httpContext = context;

                if (tokenCache != null)
                {
                    tokenCache.SetBeforeAccess(BeforeAccessNotification);
                    tokenCache.SetAfterAccess(AfterAccessNotification);
                }

                var userId = GetUsersUniqueId(user);
                tokenCacheKey = $"{userId}_TokenCache";
                userCacheKey = $"{userId}_UserCache";
            }

            public bool HasData()
            {
                return (httpContext.Session[tokenCacheKey] != null &&
                    ((byte[])httpContext.Session[tokenCacheKey]).Length > 0);
            }

            public void Clear()
            {
                sessionLock.EnterWriteLock();

                try
                {
                    httpContext.Session.Remove(tokenCacheKey);
                }
                finally
                {
                    sessionLock.ExitWriteLock();
                }
            }

            private void BeforeAccessNotification(TokenCacheNotificationArgs args)
            {
                sessionLock.EnterReadLock();

                try
                {
                    // Load the cache from the session
                    args.TokenCache.DeserializeMsalV3((byte[])httpContext.Session[tokenCacheKey]);
                }
                finally
                {
                    sessionLock.ExitReadLock();
                }
            }

            private void AfterAccessNotification(TokenCacheNotificationArgs args)
            {
                if (args.HasStateChanged)
                {
                    sessionLock.EnterWriteLock();

                    try
                    {
                        // Store the serialized cache in the session
                        httpContext.Session[tokenCacheKey] = args.TokenCache.SerializeMsalV3();
                    }
                    finally
                    {
                        sessionLock.ExitWriteLock();
                    }
                }
            }

            public void SaveUserDetails(CachedUser user)
            {

                sessionLock.EnterWriteLock();
                httpContext.Session[userCacheKey] = JsonConvert.SerializeObject(user);
                sessionLock.ExitWriteLock();
            }

            public CachedUser GetUserDetails()
            {
                sessionLock.EnterReadLock();
                var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[userCacheKey]);
                sessionLock.ExitReadLock();
                return cachedUser;
            }

            private string GetUsersUniqueId(ClaimsPrincipal user)
            {
                // Combine the user's object ID with their tenant ID

                if (user != null)
                {
                    var userObjectId = user.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value ??
                        user.FindFirst("oid").Value;

                    var userTenantId = user.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value ??
                        user.FindFirst("tid").Value;

                    if (!string.IsNullOrEmpty(userObjectId) && !string.IsNullOrEmpty(userTenantId))
                    {
                        return $"{userObjectId}.{userTenantId}";
                    }
                }

                return null;
            }
        }
    }
    ```

1. <span data-ttu-id="b5bf5-157">Ajoutez les `using` instructions suivantes en haut du `App_Start/Startup.Auth.cs` fichier.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-157">Add the following `using` statements to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. <span data-ttu-id="b5bf5-158">Remplacer la fonction `OnAuthorizationCodeReceivedAsync` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-158">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

    ```cs
    private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
    {
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
            .WithRedirectUri(redirectUri)
            .WithClientSecret(appSecret)
            .Build();

        var signedInUser = new ClaimsPrincipal(notification.AuthenticationTicket.Identity);
        var tokenStore = new SessionTokenStore(idClient.UserTokenCache, HttpContext.Current, signedInUser);

        try
        {
            string[] scopes = graphScopes.Split(' ');

            var result = await idClient.AcquireTokenByAuthorizationCode(
                scopes, notification.Code).ExecuteAsync();

            var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

            var cachedUser = new CachedUser()
            {
                DisplayName = userDetails.DisplayName,
                Email = string.IsNullOrEmpty(userDetails.Mail) ?
                userDetails.UserPrincipalName : userDetails.Mail,
                Avatar = string.Empty
            };

            tokenStore.SaveUserDetails(cachedUser);
        }
        catch (MsalException ex)
        {
            string message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
            notification.HandleResponse();
            notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
        }
        catch (Microsoft.Graph.ServiceException ex)
        {
            string message = "GetUserDetailsAsync threw an exception";
            notification.HandleResponse();
            notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
        }
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="b5bf5-159">Les modifications apportées à cette nouvelle version de `OnAuthorizationCodeReceivedAsync` apportent ces changements :</span><span class="sxs-lookup"><span data-stu-id="b5bf5-159">The changes in this new version of `OnAuthorizationCodeReceivedAsync` do the following:</span></span>
    >
    > - <span data-ttu-id="b5bf5-160">Le code encapsule désormais le cache de jetons utilisateur par défaut de `ConfidentialClientApplication`avec la classe `SessionTokenStore`.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-160">The code now wraps the `ConfidentialClientApplication`'s default user token cache with the `SessionTokenStore` class.</span></span> <span data-ttu-id="b5bf5-161">La bibliothèque MSAL gère la logique de stockage des jetons et les actualise en cas de besoin.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-161">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
    > - <span data-ttu-id="b5bf5-162">Le code transmet désormais les informations utilisateur obtenues à partir de Microsoft Graph à l’objet `SessionTokenStore` à stocker dans la session.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-162">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
    > - <span data-ttu-id="b5bf5-163">En cas de réussite, le code ne redirige plus, il renvoie simplement.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-163">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="b5bf5-164">Cela permet à l’intergiciel OWIN de terminer le processus d’authentification.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-164">This allows the OWIN middleware to complete the authentication process.</span></span>

1. <span data-ttu-id="b5bf5-165">Mettez à jour `SignOut` l’action pour effacer le magasin de jetons avant la signature. Ajoutez `using` l’instruction suivante en haut de `Controllers/AccountController.cs` .</span><span class="sxs-lookup"><span data-stu-id="b5bf5-165">Update the `SignOut` action to clear the token store before signing out. Add the following `using` statement to the top of `Controllers/AccountController.cs`.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. <span data-ttu-id="b5bf5-166">Remplacez la fonction `SignOut` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-166">Replace the existing `SignOut` function with the following.</span></span>

    ```cs
    public ActionResult SignOut()
    {
        if (Request.IsAuthenticated)
        {
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

            tokenStore.Clear();

            Request.GetOwinContext().Authentication.SignOut(
                CookieAuthenticationDefaults.AuthenticationType);
        }

        return RedirectToAction("Index", "Home");
    }
    ```

1. <span data-ttu-id="b5bf5-167">Ouvrez `Controllers/BaseController.cs` et ajoutez les `using` instructions suivantes en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-167">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. <span data-ttu-id="b5bf5-168">Ajoutez la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-168">Add the following function.</span></span>

    ```cs
    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (Request.IsAuthenticated)
        {
            // Get the user's token cache
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

            if (tokenStore.HasData())
            {
                // Add the user to the view bag
                ViewBag.User = tokenStore.GetUserDetails();
            }
            else
            {
                // The session has lost data. This happens often
                // when debugging. Log out so the user can log back in
                Request.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
                filterContext.Result = RedirectToAction("Index", "Home");
            }
        }

        base.OnActionExecuting(filterContext);
    }
    ```

1. <span data-ttu-id="b5bf5-169">Démarrez le serveur et suivez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-169">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="b5bf5-170">Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes en cours de signature.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-170">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. <span data-ttu-id="b5bf5-172">Cliquez sur l’avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **de** connexion.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-172">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="b5bf5-173">Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-173">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="b5bf5-175">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="b5bf5-175">Refreshing tokens</span></span>

<span data-ttu-id="b5bf5-176">À ce stade, votre application dispose d’un jeton d’accès, qui est envoyé dans l’en-tête des `Authorization` appels d’API.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-176">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="b5bf5-177">Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-177">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="b5bf5-178">Cependant, ce jeton est de courte durée.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-178">However, this token is short-lived.</span></span> <span data-ttu-id="b5bf5-179">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-179">The token expires an hour after it is issued.</span></span> <span data-ttu-id="b5bf5-180">C’est là que le jeton d’actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-180">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="b5bf5-181">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-181">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="b5bf5-182">Étant donné que l’application utilise la bibliothèque MSAL et sérialise l’objet, vous n’avez pas besoin d’implémenter de logique d’actualisation `TokenCache` de jeton.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-182">Because the app is using the MSAL library and serializing the `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="b5bf5-183">La méthode `ConfidentialClientApplication.AcquireTokenSilentAsync` fait la logique pour vous.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-183">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="b5bf5-184">Il vérifie d’abord le jeton mis en cache et, s’il n’a pas expiré, il le renvoie.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-184">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="b5bf5-185">S’il a expiré, il utilise le jeton d’actualisation mis en cache pour en obtenir un nouveau.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-185">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="b5bf5-186">Vous utiliserez cette méthode dans le module suivant.</span><span class="sxs-lookup"><span data-stu-id="b5bf5-186">You'll use this method in the following module.</span></span>
