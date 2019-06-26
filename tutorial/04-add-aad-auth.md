<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="68db3-101">Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="68db3-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="68db3-102">Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler l’API Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="68db3-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="68db3-103">Dans cette étape, vous allez intégrer le middleware OWIN et la bibliothèque de [bibliothèque d’authentification Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client/) dans l’application.</span><span class="sxs-lookup"><span data-stu-id="68db3-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

1. <span data-ttu-id="68db3-104">Cliquez avec le bouton droit sur le projet du **didacticiel Graph** dans l’Explorateur de solutions et sélectionnez **Ajouter > nouvel élément...**. Choisissez **fichier de configuration Web**, nommez `PrivateSettings.config` le fichier, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-104">Right-click the **graph-tutorial** project in Solution Explorer and select **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and select **Add**.</span></span> <span data-ttu-id="68db3-105">Remplacez l'ensemble de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-105">Replace its entire contents with the following code.</span></span>

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="http://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    <span data-ttu-id="68db3-106">Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_PASSWORD_HERE` l’application et remplacez par la clé secrète client que vous avez générée.</span><span class="sxs-lookup"><span data-stu-id="68db3-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="68db3-107">Si votre clé secrète client contient des esperluettes (`&`), veillez à les remplacer par `&amp;` dans `PrivateSettings.config`.</span><span class="sxs-lookup"><span data-stu-id="68db3-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="68db3-108">Veillez également à modifier la `PORT` valeur de `ida:RedirectUri` pour qu’elle corresponde à l’URL de votre application.</span><span class="sxs-lookup"><span data-stu-id="68db3-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="68db3-109">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `PrivateSettings.config` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="68db3-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="68db3-110">Mise `Web.config` à jour pour charger ce nouveau fichier.</span><span class="sxs-lookup"><span data-stu-id="68db3-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="68db3-111">Remplacez `<appSettings>` (ligne 7) par ce qui suit:</span><span class="sxs-lookup"><span data-stu-id="68db3-111">Replace the `<appSettings>` (line 7) with the following</span></span>

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="68db3-112">Mettre en œuvre la connexion</span><span class="sxs-lookup"><span data-stu-id="68db3-112">Implement sign-in</span></span>

<span data-ttu-id="68db3-113">Commencez par initialiser l’intergiciel OWIN pour utiliser l’authentification Azure AD pour l’application.</span><span class="sxs-lookup"><span data-stu-id="68db3-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span>

1. <span data-ttu-id="68db3-114">Cliquez avec le bouton droit sur le dossier **App_Start** dans l’Explorateur de solutions et sélectionnez **Ajouter une > classe...**. Nommez le `Startup.Auth.cs` fichier, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-114">Right-click the **App_Start** folder in Solution Explorer and select **Add > Class...**. Name the file `Startup.Auth.cs` and select **Add**.</span></span> <span data-ttu-id="68db3-115">Remplacez tout le contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-115">Replace the entire contents with the following code.</span></span>

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

    <span data-ttu-id="68db3-116">Ce code configure le middleware OWIN avec les valeurs de et définit `PrivateSettings.config` deux méthodes de rappel, `OnAuthenticationFailedAsync` et `OnAuthorizationCodeReceivedAsync`.</span><span class="sxs-lookup"><span data-stu-id="68db3-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="68db3-117">Ces méthodes de rappel seront appelées lorsque le processus de connexion est renvoyé à partir d’Azure.</span><span class="sxs-lookup"><span data-stu-id="68db3-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

1. <span data-ttu-id="68db3-118">À présent, `Startup.cs` mettez à jour le `ConfigureAuth` fichier pour appeler la méthode.</span><span class="sxs-lookup"><span data-stu-id="68db3-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="68db3-119">Remplacez tout le contenu de `Startup.cs` par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

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

1. <span data-ttu-id="68db3-120">Ajoutez une `Error` action à la `HomeController` classe pour transformer les `message` paramètres `debug` de requête et en `Alert` un objet.</span><span class="sxs-lookup"><span data-stu-id="68db3-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="68db3-121">Ouvrez `Controllers/HomeController.cs` et ajoutez la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="68db3-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. <span data-ttu-id="68db3-122">Ajoutez un contrôleur pour gérer la connexion.</span><span class="sxs-lookup"><span data-stu-id="68db3-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="68db3-123">Cliquez avec le bouton droit sur le dossier **Controllers** dans l’Explorateur de solutions et sélectionnez **Ajouter un contrôleur de >...**. Sélectionnez **contrôleur MVC 5-vide** et sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-123">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="68db3-124">Nommez le `AccountController` contrôleur et sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-124">Name the controller `AccountController` and select **Add**.</span></span> <span data-ttu-id="68db3-125">Remplacez tout le contenu du fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-125">Replace the entire contents of the file with the following code.</span></span>

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

    <span data-ttu-id="68db3-126">Définit une `SignIn` action and `SignOut` .</span><span class="sxs-lookup"><span data-stu-id="68db3-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="68db3-127">L' `SignIn` action vérifie si la demande est déjà authentifiée.</span><span class="sxs-lookup"><span data-stu-id="68db3-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="68db3-128">Si ce n’est pas le cas, il appelle le middleware OWIN pour authentifier l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="68db3-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="68db3-129">L' `SignOut` action appelle l’intergiciel OWIN pour se déconnecter.</span><span class="sxs-lookup"><span data-stu-id="68db3-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

1. <span data-ttu-id="68db3-130">Enregistrez vos modifications et démarrez le projet.</span><span class="sxs-lookup"><span data-stu-id="68db3-130">Save your changes and start the project.</span></span> <span data-ttu-id="68db3-131">Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="68db3-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="68db3-132">Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="68db3-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="68db3-133">Le navigateur redirige vers l’application en affichant le jeton.</span><span class="sxs-lookup"><span data-stu-id="68db3-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="68db3-134">Obtenir les détails de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="68db3-134">Get user details</span></span>

<span data-ttu-id="68db3-135">Une fois que l’utilisateur est connecté, vous pouvez obtenir ses informations à partir de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="68db3-135">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="68db3-136">Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="68db3-136">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="68db3-137">Nommez le `Helpers`dossier.</span><span class="sxs-lookup"><span data-stu-id="68db3-137">Name the folder `Helpers`.</span></span>

1. <span data-ttu-id="68db3-138">Cliquez avec le bouton droit sur ce nouveau dossier, puis sélectionnez **Ajouter une classe de >.**... Nommez le `GraphHelper.cs` fichier, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-138">Right click this new folder and select **Add > Class...**. Name the file `GraphHelper.cs` and select **Add**.</span></span> <span data-ttu-id="68db3-139">Remplacez le contenu de ce fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-139">Replace the contents of this file with the following code.</span></span>

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
                        async (requestMessage) =>
                        {
                            requestMessage.Headers.Authorization =
                                new AuthenticationHeaderValue("Bearer", accessToken);
                        }));

                return await graphClient.Me.Request().GetAsync();
            }
        }
    }
    ```

    <span data-ttu-id="68db3-140">Cette fonctionnalité implémente `GetUserDetails` la fonction, qui utilise le kit de développement logiciel ( `/me` SDK) Microsoft Graph pour appeler le point de terminaison et renvoyer le résultat.</span><span class="sxs-lookup"><span data-stu-id="68db3-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="68db3-141">Mettez à `OnAuthorizationCodeReceivedAsync` jour la `App_Start/Startup.Auth.cs` méthode dans pour appeler cette fonction.</span><span class="sxs-lookup"><span data-stu-id="68db3-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="68db3-142">Ajoutez l’instruction `using` suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="68db3-142">Add the following `using` statement to the top of the file.</span></span>

    ```cs
    using graph_tutorial.Helpers;
    ```

1. <span data-ttu-id="68db3-143">Remplacez le bloc `try` `OnAuthorizationCodeReceivedAsync` existant par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

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

1. <span data-ttu-id="68db3-144">Enregistrez vos modifications et démarrez l’application, après vous être connecté, vous devriez voir le nom et l’adresse de messagerie de l’utilisateur au lieu du jeton d’accès.</span><span class="sxs-lookup"><span data-stu-id="68db3-144">Save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="68db3-145">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="68db3-145">Storing the tokens</span></span>

<span data-ttu-id="68db3-146">Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l’application.</span><span class="sxs-lookup"><span data-stu-id="68db3-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="68db3-147">Étant donné qu’il s’agit d’un exemple d’application, vous allez utiliser la session pour stocker les jetons.</span><span class="sxs-lookup"><span data-stu-id="68db3-147">Since this is a sample app, you will use the session to store the tokens.</span></span> <span data-ttu-id="68db3-148">Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="68db3-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span> <span data-ttu-id="68db3-149">Dans cette section, vous allez:</span><span class="sxs-lookup"><span data-stu-id="68db3-149">In this section, you will:</span></span>

- <span data-ttu-id="68db3-150">Implémentez une classe de magasin de jetons pour sérialiser et stocker le cache de jetons MSAL, ainsi que les détails de l’utilisateur dans la session utilisateur.</span><span class="sxs-lookup"><span data-stu-id="68db3-150">Implement a token store class to serialize and store the MSAL token cache and the user's details in the user session.</span></span>
- <span data-ttu-id="68db3-151">Mettez à jour le code d’authentification pour utiliser la classe de magasin de jetons.</span><span class="sxs-lookup"><span data-stu-id="68db3-151">Update the authentication code to use the token store class.</span></span>
- <span data-ttu-id="68db3-152">Mettez à jour la classe de contrôleur de base pour exposer les détails de l’utilisateur stocké à tous les affichages de l’application.</span><span class="sxs-lookup"><span data-stu-id="68db3-152">Update the base controller class to expose the stored user details to all views in the application.</span></span>

1. <span data-ttu-id="68db3-153">Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="68db3-153">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="68db3-154">Nommez le `TokenStorage`dossier.</span><span class="sxs-lookup"><span data-stu-id="68db3-154">Name the folder `TokenStorage`.</span></span>

1. <span data-ttu-id="68db3-155">Cliquez avec le bouton droit sur ce nouveau dossier, puis sélectionnez **Ajouter une classe de >.**... Nommez le `SessionTokenStore.cs` fichier, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="68db3-155">Right click this new folder and select **Add > Class...**. Name the file `SessionTokenStore.cs` and select **Add**.</span></span> <span data-ttu-id="68db3-156">Remplacez le contenu de ce fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-156">Replace the contents of this file with the following code.</span></span>

    ```cs
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

        // Adapted from https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2
        public class SessionTokenStore
        {
            private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

            private readonly string userId = string.Empty;
            private readonly string cacheId = string.Empty;
            private readonly string cachedUserId = string.Empty;
            private HttpContext httpContext = null;
            private ITokenCache tokenCache;

            public SessionTokenStore(string userId, HttpContext httpcontext)
            {
                this.userId = userId;
                this.cacheId = $"{userId}_TokenCache";
                this.cachedUserId = $"{userId}_UserCache";
                this.httpContext = httpcontext;
            }

            public void Initialize(ITokenCache tokenCache)
            {
                this.tokenCache = tokenCache;
                this.tokenCache.SetBeforeAccess(BeforeAccessNotification);
                this.tokenCache.SetAfterAccess(AfterAccessNotification);
                Load();
            }

            public bool HasData()
            {
                return (httpContext.Session[cacheId] != null && ((byte[])httpContext.Session[cacheId]).Length > 0);
            }

            public void Clear()
            {
                httpContext.Session.Remove(cacheId);
            }

            private void Load()
            {
                sessionLock.EnterReadLock();
                tokenCache.DeserializeMsalV3((byte[])httpContext.Session[cacheId]);
                sessionLock.ExitReadLock();
            }

            private void Persist()
            {
                sessionLock.EnterReadLock();
                httpContext.Session[cacheId] = tokenCache.SerializeMsalV3();
                sessionLock.ExitReadLock();
            }

            private void BeforeAccessNotification(TokenCacheNotificationArgs args)
            {
                Load();
            }

            private void AfterAccessNotification(TokenCacheNotificationArgs args)
            {
                if (args.HasStateChanged)
                {
                    Persist();
                }
            }

            public void SaveUserDetails(CachedUser user)
            {
                sessionLock.EnterReadLock();
                httpContext.Session[cachedUserId] = JsonConvert.SerializeObject(user);
                sessionLock.ExitReadLock();
            }

            public CachedUser GetUserDetails()
            {
                sessionLock.EnterReadLock();
                var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[cachedUserId]);
                sessionLock.ExitReadLock();
                return cachedUser;
            }
        }
    }
    ```

1. <span data-ttu-id="68db3-157">Ajoutez l’instruction `using` suivante en haut du `App_Start/Startup.Auth.cs` fichier.</span><span class="sxs-lookup"><span data-stu-id="68db3-157">Add the following `using` statement to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.IdentityModel.Claims;
    ```

1. <span data-ttu-id="68db3-158">Remplacez la fonction `OnAuthorizationCodeReceivedAsync` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="68db3-158">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

    ```cs
    private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
    {
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
            .WithRedirectUri(redirectUri)
            .WithClientSecret(appSecret)
            .Build();

        var signedInUserId = notification.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
        var tokenStore = new SessionTokenStore(signedInUserId, HttpContext.Current);
        tokenStore.Initialize(idClient.UserTokenCache);

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
    > <span data-ttu-id="68db3-159">Les modifications apportées à cette `OnAuthorizationCodeReceivedAsync` nouvelle version de effectuent les opérations suivantes:</span><span class="sxs-lookup"><span data-stu-id="68db3-159">The changes in this new version of `OnAuthorizationCodeReceivedAsync` do the following:</span></span>
    >
    > - <span data-ttu-id="68db3-160">Le code intègre désormais le `ConfidentialClientApplication`cache de jetons de l’utilisateur par défaut `SessionTokenStore` à la classe.</span><span class="sxs-lookup"><span data-stu-id="68db3-160">The code now wraps the `ConfidentialClientApplication`'s default user token cache with the `SessionTokenStore` class.</span></span> <span data-ttu-id="68db3-161">La bibliothèque MSAL gérera la logique de stockage des jetons et de l’actualiser si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="68db3-161">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
    > - <span data-ttu-id="68db3-162">Le code transmet maintenant les détails de l’utilisateur obtenus de Microsoft Graph `SessionTokenStore` à l’objet à stocker dans la session.</span><span class="sxs-lookup"><span data-stu-id="68db3-162">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
    > - <span data-ttu-id="68db3-163">En cas de réussite, le code ne redirige plus, il se contente de le renvoyer.</span><span class="sxs-lookup"><span data-stu-id="68db3-163">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="68db3-164">Cela permet au middleware OWIN de terminer le processus d’authentification.</span><span class="sxs-lookup"><span data-stu-id="68db3-164">This allows the OWIN middleware to complete the authentication process.</span></span>

1. <span data-ttu-id="68db3-165">Mettre à `SignOut` jour l’action pour effacer le magasin de jetons avant de se déconnecter. Ajoutez l’instruction `using` suivante en haut de `Controllers/AccountController.cs`.</span><span class="sxs-lookup"><span data-stu-id="68db3-165">Update the `SignOut` action to clear the token store before signing out. Add the following `using` statement to the top of `Controllers/AccountController.cs`.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. <span data-ttu-id="68db3-166">Remplacez la fonction `SignOut` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="68db3-166">Replace the existing `SignOut` function with the following.</span></span>

    ```cs
    public ActionResult SignOut()
    {
        if (Request.IsAuthenticated)
        {
            string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
            SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

            tokenStore.Clear();

            Request.GetOwinContext().Authentication.SignOut(
                CookieAuthenticationDefaults.AuthenticationType);
        }

        return RedirectToAction("Index", "Home");
    }
    ```

1. <span data-ttu-id="68db3-167">Ouvrez `Controllers/BaseController.cs` et ajoutez les instructions `using` suivantes en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="68db3-167">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. <span data-ttu-id="68db3-168">Ajoutez la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="68db3-168">Add the following function.</span></span>

    ```cs
    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (Request.IsAuthenticated)
        {
            // Get the signed in user's id and create a token cache
            string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
            SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

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

1. <span data-ttu-id="68db3-169">Démarrez le serveur et parcourez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="68db3-169">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="68db3-170">Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.</span><span class="sxs-lookup"><span data-stu-id="68db3-170">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. <span data-ttu-id="68db3-172">Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour \*\*\*\* accéder au lien Déconnexion.</span><span class="sxs-lookup"><span data-stu-id="68db3-172">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="68db3-173">Cliquez \*\*\*\* sur Déconnexion pour réinitialiser la session et revenir à la page d’accueil.</span><span class="sxs-lookup"><span data-stu-id="68db3-173">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Capture d’écran du menu déroulant avec le lien Déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="68db3-175">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="68db3-175">Refreshing tokens</span></span>

<span data-ttu-id="68db3-176">À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API.</span><span class="sxs-lookup"><span data-stu-id="68db3-176">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="68db3-177">Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="68db3-177">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="68db3-178">Toutefois, ce jeton est éphémère.</span><span class="sxs-lookup"><span data-stu-id="68db3-178">However, this token is short-lived.</span></span> <span data-ttu-id="68db3-179">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="68db3-179">The token expires an hour after it is issued.</span></span> <span data-ttu-id="68db3-180">C’est ici que le jeton d’actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="68db3-180">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="68db3-181">Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans demander à l’utilisateur de se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="68db3-181">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="68db3-182">Étant donné que l’application utilise la bibliothèque MSAL et `TokenCache` un objet, il n’est pas nécessaire d’implémenter une logique d’actualisation de jeton.</span><span class="sxs-lookup"><span data-stu-id="68db3-182">Because the app is using the MSAL library and a `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="68db3-183">La `ConfidentialClientApplication.AcquireTokenSilentAsync` méthode effectue l’ensemble de la logique pour vous.</span><span class="sxs-lookup"><span data-stu-id="68db3-183">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="68db3-184">Il vérifie d’abord le jeton mis en cache et, s’il n’a pas expiré, il le renvoie.</span><span class="sxs-lookup"><span data-stu-id="68db3-184">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="68db3-185">Si elle a expiré, elle utilise le jeton d’actualisation mis en cache pour en obtenir une nouvelle.</span><span class="sxs-lookup"><span data-stu-id="68db3-185">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="68db3-186">Vous utiliserez cette méthode dans le module suivant.</span><span class="sxs-lookup"><span data-stu-id="68db3-186">You'll use this method in the following module.</span></span>
