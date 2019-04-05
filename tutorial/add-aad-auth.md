<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="18daa-101">Dans cet exercice, vous allez étendre l'application de l'exercice précédent pour prendre en charge l'authentification avec Azure AD.</span><span class="sxs-lookup"><span data-stu-id="18daa-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="18daa-102">Cela est nécessaire pour obtenir le jeton d'accès OAuth nécessaire pour appeler Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="18daa-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="18daa-103">Dans cette étape, vous allez intégrer le middleware OWIN et la bibliothèque de [bibliothèque d'authentification Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client/) dans l'application.</span><span class="sxs-lookup"><span data-stu-id="18daa-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

<span data-ttu-id="18daa-104">Cliquez avec le bouton droit sur le projet du **didacticiel Graph** dans l'Explorateur de solutions et choisissez **Ajouter un nouvel élément >...**. Choisissez **fichier de configuration Web**, nommez `PrivateSettings.config` le fichier, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-104">Right-click the **graph-tutorial** project in Solution Explorer and choose **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and choose **Add**.</span></span> <span data-ttu-id="18daa-105">Remplacez l'ensemble de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-105">Replace its entire contents with the following code.</span></span>

```xml
<appSettings>
    <add key="ida:AppID" value="YOUR APP ID" />
    <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
    <add key="ida:RedirectUri" value="http://localhost:PORT/" />
    <add key="ida:AppScopes" value="User.Read Calendars.Read" />
</appSettings>
```

<span data-ttu-id="18daa-106">Remplacez `YOUR_APP_ID_HERE` par l'ID de l'application dans le portail d'inscription de `YOUR_APP_PASSWORD_HERE` l'application et remplacez par la clé secrète client que vous avez générée.</span><span class="sxs-lookup"><span data-stu-id="18daa-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="18daa-107">Si votre clé secrète client contient des esperluettes (`&`), veillez à les remplacer par `&amp;` dans `PrivateSettings.config`.</span><span class="sxs-lookup"><span data-stu-id="18daa-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="18daa-108">Veillez également à modifier la `PORT` valeur de `ida:RedirectUri` pour qu'elle corresponde à l'URL de votre application.</span><span class="sxs-lookup"><span data-stu-id="18daa-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="18daa-109">Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d'exclure le `PrivateSettings.config` fichier du contrôle de code source afin d'éviter une fuite accidentelle de votre ID d'application et de votre mot de passe.</span><span class="sxs-lookup"><span data-stu-id="18daa-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="18daa-110">Mise `Web.config` à jour pour charger ce nouveau fichier.</span><span class="sxs-lookup"><span data-stu-id="18daa-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="18daa-111">Remplacez `<appSettings>` (ligne 7) par ce qui suit:</span><span class="sxs-lookup"><span data-stu-id="18daa-111">Replace the `<appSettings>` (line 7) with the following</span></span>

```xml
<appSettings file="PrivateSettings.config">
```

## <a name="implement-sign-in"></a><span data-ttu-id="18daa-112">Mettre en œuvre la connexion</span><span class="sxs-lookup"><span data-stu-id="18daa-112">Implement sign-in</span></span>

<span data-ttu-id="18daa-113">Commencez par initialiser l'intergiciel OWIN pour utiliser l'authentification Azure AD pour l'application.</span><span class="sxs-lookup"><span data-stu-id="18daa-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span> <span data-ttu-id="18daa-114">Cliquez avec le bouton droit sur le dossier **App_Start** dans l'Explorateur de solutions et choisissez **Ajouter une classe >.**... Nommez le `Startup.Auth.cs` fichier, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-114">Right-click the **App_Start** folder in Solution Explorer and choose **Add > Class...**. Name the file `Startup.Auth.cs` and choose **Add**.</span></span> <span data-ttu-id="18daa-115">Remplacez tout le contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-115">Replace the entire contents with the following code.</span></span>

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
            var idClient = new ConfidentialClientApplication(
                appId, redirectUri, new ClientCredential(appSecret), null, null);

            string message;
            string debug;

            try
            {
                string[] scopes = graphScopes.Split(' ');

                var result = await idClient.AcquireTokenByAuthorizationCodeAsync(
                    notification.Code, scopes);

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

<span data-ttu-id="18daa-116">Ce code configure le middleware OWIN avec les valeurs de et définit `PrivateSettings.config` deux méthodes de rappel, `OnAuthenticationFailedAsync` et `OnAuthorizationCodeReceivedAsync`.</span><span class="sxs-lookup"><span data-stu-id="18daa-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="18daa-117">Ces méthodes de rappel seront appelées lorsque le processus de connexion est renvoyé à partir d'Azure.</span><span class="sxs-lookup"><span data-stu-id="18daa-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

<span data-ttu-id="18daa-118">À présent, `Startup.cs` mettez à jour le `ConfigureAuth` fichier pour appeler la méthode.</span><span class="sxs-lookup"><span data-stu-id="18daa-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="18daa-119">Remplacez tout le contenu de `Startup.cs` par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

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

<span data-ttu-id="18daa-120">Ajoutez une `Error` action à la `HomeController` classe pour transformer les `message` paramètres `debug` de requête et en `Alert` un objet.</span><span class="sxs-lookup"><span data-stu-id="18daa-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="18daa-121">Ouvrez `Controllers/HomeController.cs` et ajoutez la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="18daa-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

```cs
public ActionResult Error(string message, string debug)
{
    Flash(message, debug);
    return RedirectToAction("Index");
}
```

<span data-ttu-id="18daa-122">Ajoutez un contrôleur pour gérer la connexion.</span><span class="sxs-lookup"><span data-stu-id="18daa-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="18daa-123">Cliquez avec le bouton droit sur le dossier **Controllers** dans l'Explorateur de solutions, puis choisissez **Ajouter un contrôleur >..**.. Sélectionnez **contrôleur MVC 5-vide** et choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-123">Right-click the **Controllers** folder in Solution Explorer and choose **Add > Controller...**. Choose **MVC 5 Controller - Empty** and choose **Add**.</span></span> <span data-ttu-id="18daa-124">Nommez le `AccountController` contrôleur, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-124">Name the controller `AccountController` and choose **Add**.</span></span> <span data-ttu-id="18daa-125">Remplacez tout le contenu du fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-125">Replace the entire contents of the file with the following code.</span></span>

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

<span data-ttu-id="18daa-126">Définit une `SignIn` action and `SignOut` .</span><span class="sxs-lookup"><span data-stu-id="18daa-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="18daa-127">L' `SignIn` action vérifie si la demande est déjà authentifiée.</span><span class="sxs-lookup"><span data-stu-id="18daa-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="18daa-128">Si ce n'est pas le cas, il appelle le middleware OWIN pour authentifier l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="18daa-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="18daa-129">L' `SignOut` action appelle l'intergiciel OWIN pour se déconnecter.</span><span class="sxs-lookup"><span data-stu-id="18daa-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

<span data-ttu-id="18daa-130">Enregistrez vos modifications et démarrez le projet.</span><span class="sxs-lookup"><span data-stu-id="18daa-130">Save your changes and start the project.</span></span> <span data-ttu-id="18daa-131">Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="18daa-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="18daa-132">Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées.</span><span class="sxs-lookup"><span data-stu-id="18daa-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="18daa-133">Le navigateur redirige vers l'application en affichant le jeton.</span><span class="sxs-lookup"><span data-stu-id="18daa-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="18daa-134">Obtenir les détails de l'utilisateur</span><span class="sxs-lookup"><span data-stu-id="18daa-134">Get user details</span></span>

<span data-ttu-id="18daa-135">Commencez par créer un fichier qui contiendra tous vos appels Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="18daa-135">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="18daa-136">Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l'Explorateur de solutions, puis choisissez **Ajouter > nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="18daa-136">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="18daa-137">Nommez le `Helpers`dossier.</span><span class="sxs-lookup"><span data-stu-id="18daa-137">Name the folder `Helpers`.</span></span> <span data-ttu-id="18daa-138">Cliquez avec le bouton droit sur ce nouveau dossier, puis choisissez **Ajouter une classe >.**... Nommez le `GraphHelper.cs` fichier, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-138">Right click this new folder and choose **Add > Class...**. Name the file `GraphHelper.cs` and choose **Add**.</span></span> <span data-ttu-id="18daa-139">Remplacez le contenu de ce fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-139">Replace the contents of this file with the following code.</span></span>

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

<span data-ttu-id="18daa-140">Cette fonctionnalité implémente `GetUserDetails` la fonction, qui utilise le kit de développement logiciel ( `/me` SDK) Microsoft Graph pour appeler le point de terminaison et renvoyer le résultat.</span><span class="sxs-lookup"><span data-stu-id="18daa-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="18daa-141">Mettez à `OnAuthorizationCodeReceivedAsync` jour la `App_Start/Startup.Auth.cs` méthode dans pour appeler cette fonction.</span><span class="sxs-lookup"><span data-stu-id="18daa-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="18daa-142">Tout d'abord, ajoutez `using` l'instruction suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="18daa-142">First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.Helpers;
```

<span data-ttu-id="18daa-143">Remplacez le bloc `try` `OnAuthorizationCodeReceivedAsync` existant par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

```cs
try
{
    string[] scopes = graphScopes.Split(' ');

    var result = await idClient.AcquireTokenByAuthorizationCodeAsync(
        notification.Code, scopes);

    var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

    string email = string.IsNullOrEmpty(userDetails.Mail) ?
        userDetails.UserPrincipalName : userDetails.Mail;

    message = "User info retrieved.";
    debug = $"User: {userDetails.DisplayName}, Email: {email}";
}
```

<span data-ttu-id="18daa-144">Maintenant, si vous enregistrez vos modifications et démarrez l'application, après vous être connecté, vous devriez voir le nom et l'adresse de messagerie de l'utilisateur au lieu du jeton d'accès.</span><span class="sxs-lookup"><span data-stu-id="18daa-144">Now if you save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="18daa-145">Stockage des jetons</span><span class="sxs-lookup"><span data-stu-id="18daa-145">Storing the tokens</span></span>

<span data-ttu-id="18daa-146">Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l'application.</span><span class="sxs-lookup"><span data-stu-id="18daa-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="18daa-147">Étant donné qu'il s'agit d'un exemple d'application, nous allons utiliser la session pour stocker les jetons.</span><span class="sxs-lookup"><span data-stu-id="18daa-147">Since this is a sample app, we'll use the session to store the tokens.</span></span> <span data-ttu-id="18daa-148">Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données.</span><span class="sxs-lookup"><span data-stu-id="18daa-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="18daa-149">Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l'Explorateur de solutions, puis choisissez **Ajouter > nouveau dossier**.</span><span class="sxs-lookup"><span data-stu-id="18daa-149">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="18daa-150">Nommez le `TokenStorage`dossier.</span><span class="sxs-lookup"><span data-stu-id="18daa-150">Name the folder `TokenStorage`.</span></span> <span data-ttu-id="18daa-151">Cliquez avec le bouton droit sur ce nouveau dossier, puis choisissez **Ajouter une classe >.**... Nommez le `SessionTokenStore.cs` fichier, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="18daa-151">Right click this new folder and choose **Add > Class...**. Name the file `SessionTokenStore.cs` and choose **Add**.</span></span> <span data-ttu-id="18daa-152">Remplacez le contenu de ce fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-152">Replace the contents of this file with the following code.</span></span>

```cs
using Microsoft.Identity.Client;
using Newtonsoft.Json;
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
        private static ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);
        private readonly string userId = string.Empty;
        private readonly string cacheId = string.Empty;
        private readonly string cachedUserId = string.Empty;
        private HttpContextBase httpContext = null;

        TokenCache tokenCache = new TokenCache();

        public SessionTokenStore(string userId, HttpContextBase httpContext)
        {
            this.userId = userId;
            cacheId = $"{userId}_TokenCache";
            cachedUserId = $"{userId}_UserCache";
            this.httpContext = httpContext;
            Load();
        }

        public TokenCache GetMsalCacheInstance()
        {
            tokenCache.SetBeforeAccess(BeforeAccessNotification);
            tokenCache.SetAfterAccess(AfterAccessNotification);
            Load();
            return tokenCache;
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
            tokenCache.Deserialize((byte[])httpContext.Session[cacheId]);
            sessionLock.ExitReadLock();
        }

        private void Persist()
        {
            sessionLock.EnterReadLock();
            httpContext.Session[cacheId] = tokenCache.Serialize();
            sessionLock.ExitReadLock();
        }

        // Triggered right before MSAL needs to access the cache.
        private void BeforeAccessNotification(TokenCacheNotificationArgs args)
        {
            // Reload the cache from the persistent store in case it changed since the last access.
            Load();
        }

        // Triggered right after MSAL accessed the cache.
        private void AfterAccessNotification(TokenCacheNotificationArgs args)
        {
            // if the access operation resulted in a cache update
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

<span data-ttu-id="18daa-153">Ce code crée une `SessionTokenStore` classe qui fonctionne avec la classe de `TokenCache` la bibliothèque MSAL.</span><span class="sxs-lookup"><span data-stu-id="18daa-153">This code creates a `SessionTokenStore` class that works with the MSAL library's `TokenCache` class.</span></span> <span data-ttu-id="18daa-154">La plus grande partie du code implique la sérialisation et la désérialisation `TokenCache` du vers la session.</span><span class="sxs-lookup"><span data-stu-id="18daa-154">Most of the code here involves serializing and deserializing the `TokenCache` to the session.</span></span> <span data-ttu-id="18daa-155">Elle fournit également une classe et des méthodes pour sérialiser et désérialiser les détails de l'utilisateur dans la session.</span><span class="sxs-lookup"><span data-stu-id="18daa-155">It also provides a class and methods to serialize and deserialize the user's details to the session.</span></span>

<span data-ttu-id="18daa-156">À présent, ajoutez l' `using` instruction suivante en haut du `App_Start/Startup.Auth.cs` fichier.</span><span class="sxs-lookup"><span data-stu-id="18daa-156">Now, add the following `using` statement to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.IdentityModel.Claims;
```

<span data-ttu-id="18daa-157">À présent, `OnAuthorizationCodeReceivedAsync` mettez à jour la fonction pour créer `SessionTokenStore` une instance de la classe et fournissez- `ConfidentialClientApplication` lui le constructeur pour l'objet.</span><span class="sxs-lookup"><span data-stu-id="18daa-157">Now update the `OnAuthorizationCodeReceivedAsync` function to create an instance of the `SessionTokenStore` class and provide that to the constructor for the `ConfidentialClientApplication` object.</span></span> <span data-ttu-id="18daa-158">Cela entraînera l'utilisation de votre implémentation de cache par MSAL pour le stockage des jetons.</span><span class="sxs-lookup"><span data-stu-id="18daa-158">That will cause MSAL to use your cache implementation for storing tokens.</span></span> <span data-ttu-id="18daa-159">Remplacez la fonction `OnAuthorizationCodeReceivedAsync` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="18daa-159">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

```cs
private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
{
    // Get the signed in user's id and create a token cache
    string signedInUserId = notification.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
    SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId,
        notification.OwinContext.Environment["System.Web.HttpContextBase"] as HttpContextBase);

    var idClient = new ConfidentialClientApplication(
        appId, redirectUri, new ClientCredential(appSecret), tokenStore.GetMsalCacheInstance(), null);

    try
    {
        string[] scopes = graphScopes.Split(' ');

        var result = await idClient.AcquireTokenByAuthorizationCodeAsync(
            notification.Code, scopes);

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
    catch(Microsoft.Graph.ServiceException ex)
    {
        string message = "GetUserDetailsAsync threw an exception";
        notification.HandleResponse();
        notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
    }
}
```

<span data-ttu-id="18daa-160">Pour résumer les modifications:</span><span class="sxs-lookup"><span data-stu-id="18daa-160">To summarize the changes:</span></span>

- <span data-ttu-id="18daa-161">Le code transmet à présent `TokenCache` un objet au constructeur pour `ConfidentialClientApplication`.</span><span class="sxs-lookup"><span data-stu-id="18daa-161">The code now passes a `TokenCache` object to the constructor for `ConfidentialClientApplication`.</span></span> <span data-ttu-id="18daa-162">La bibliothèque MSAL gérera la logique de stockage des jetons et de l'actualiser si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="18daa-162">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
- <span data-ttu-id="18daa-163">Le code transmet maintenant les détails de l'utilisateur obtenus de Microsoft Graph `SessionTokenStore` à l'objet à stocker dans la session.</span><span class="sxs-lookup"><span data-stu-id="18daa-163">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
- <span data-ttu-id="18daa-164">En cas de réussite, le code ne redirige plus, il se contente de le renvoyer.</span><span class="sxs-lookup"><span data-stu-id="18daa-164">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="18daa-165">Cela permet au middleware OWIN de terminer le processus d'authentification.</span><span class="sxs-lookup"><span data-stu-id="18daa-165">This allows the OWIN middleware to complete the authentication process.</span></span>

<span data-ttu-id="18daa-166">Étant donné que le cache de jetons est stocké dans la `SignOut` session, `Controllers/AccountController.cs` mettez à jour l'action dans pour effacer le magasin de jetons avant de vous déconnecter. Tout d'abord, ajoutez `using` l'instruction suivante en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="18daa-166">Since the token cache is stored in the session, update the `SignOut` action in `Controllers/AccountController.cs` to clear the token store before signing out. First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
```

<span data-ttu-id="18daa-167">Ensuite, remplacez la fonction `SignOut` existante par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="18daa-167">Then, replace the existing `SignOut` function with the following.</span></span>

```cs
public ActionResult SignOut()
{
    if (Request.IsAuthenticated)
    {
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, HttpContext);

        tokenStore.Clear();

        Request.GetOwinContext().Authentication.SignOut(
            CookieAuthenticationDefaults.AuthenticationType);
    }

    return RedirectToAction("Index", "Home");
}
```

<span data-ttu-id="18daa-168">Les détails de l'utilisateur mis en cache sont ceux que chaque vue de l'application aura besoin, `BaseController` donc mettez à jour la classe pour charger ces informations à partir de la session.</span><span class="sxs-lookup"><span data-stu-id="18daa-168">The cached user details are something that every view in the application will need, so update the `BaseController` class to load this information from the session.</span></span> <span data-ttu-id="18daa-169">Ouvrez `Controllers/BaseController.cs` et ajoutez les instructions `using` suivantes en haut du fichier.</span><span class="sxs-lookup"><span data-stu-id="18daa-169">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.Security.Claims;
using System.Web;
using Microsoft.Owin.Security.Cookies;
```

<span data-ttu-id="18daa-170">Ajoutez ensuite la fonction suivante.</span><span class="sxs-lookup"><span data-stu-id="18daa-170">Then add the following function.</span></span>

```cs
protected override void OnActionExecuting(ActionExecutingContext filterContext)
{
    if (Request.IsAuthenticated)
    {
        // Get the signed in user's id and create a token cache
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, HttpContext);

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

<span data-ttu-id="18daa-171">Démarrez le serveur et parcourez le processus de connexion.</span><span class="sxs-lookup"><span data-stu-id="18daa-171">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="18daa-172">Vous devez revenir sur la page d'accueil, mais l'interface utilisateur doit changer pour indiquer que vous êtes connecté.</span><span class="sxs-lookup"><span data-stu-id="18daa-172">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Capture d'écran de la page d'accueil après la connexion](./images/add-aad-auth-01.png)

<span data-ttu-id="18daa-174">Cliquez sur Avatar de l'utilisateur dans le coin supérieur droit pour \*\*\*\* accéder au lien Déconnexion.</span><span class="sxs-lookup"><span data-stu-id="18daa-174">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="18daa-175">Cliquez \*\*\*\* sur Déconnexion pour réinitialiser la session et revenir à la page d'accueil.</span><span class="sxs-lookup"><span data-stu-id="18daa-175">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Capture d'écran du menu déroulant avec le lien déConnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="18daa-177">Actualisation des jetons</span><span class="sxs-lookup"><span data-stu-id="18daa-177">Refreshing tokens</span></span>

<span data-ttu-id="18daa-178">À ce stade, votre application a un jeton d'accès, qui est envoyé `Authorization` dans l'en-tête des appels d'API.</span><span class="sxs-lookup"><span data-stu-id="18daa-178">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="18daa-179">Il s'agit du jeton qui permet à l'application d'accéder à Microsoft Graph pour le compte de l'utilisateur.</span><span class="sxs-lookup"><span data-stu-id="18daa-179">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="18daa-180">Toutefois, ce jeton est éphémère.</span><span class="sxs-lookup"><span data-stu-id="18daa-180">However, this token is short-lived.</span></span> <span data-ttu-id="18daa-181">Le jeton expire une heure après son émission.</span><span class="sxs-lookup"><span data-stu-id="18daa-181">The token expires an hour after it is issued.</span></span> <span data-ttu-id="18daa-182">C'est ici que le jeton d'actualisation devient utile.</span><span class="sxs-lookup"><span data-stu-id="18daa-182">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="18daa-183">Le jeton d'actualisation permet à l'application de demander un nouveau jeton d'accès sans demander à l'utilisateur de se reconnecter.</span><span class="sxs-lookup"><span data-stu-id="18daa-183">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="18daa-184">Étant donné que l'application utilise la bibliothèque MSAL et `TokenCache` un objet, il n'est pas nécessaire d'implémenter une logique d'actualisation de jeton.</span><span class="sxs-lookup"><span data-stu-id="18daa-184">Because the app is using the MSAL library and a `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="18daa-185">La `ConfidentialClientApplication.AcquireTokenSilentAsync` méthode effectue l'ensemble de la logique pour vous.</span><span class="sxs-lookup"><span data-stu-id="18daa-185">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="18daa-186">Il vérifie d'abord le jeton mis en cache et, s'il n'a pas expiré, il le renvoie.</span><span class="sxs-lookup"><span data-stu-id="18daa-186">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="18daa-187">Si elle a expiré, elle utilise le jeton d'actualisation mis en cache pour en obtenir une nouvelle.</span><span class="sxs-lookup"><span data-stu-id="18daa-187">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="18daa-188">Vous utiliserez cette méthode dans le module suivant.</span><span class="sxs-lookup"><span data-stu-id="18daa-188">You'll use this method in the following module.</span></span>