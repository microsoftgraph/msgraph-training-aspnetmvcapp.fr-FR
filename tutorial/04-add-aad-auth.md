<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cela est nécessaire pour obtenir le jeton d’accès OAuth nécessaire pour appeler l’API Microsoft Graph. Dans cette étape, vous allez intégrer le middleware OWIN et la bibliothèque de [bibliothèque d’authentification Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client/) dans l’application.

1. Cliquez avec le bouton droit sur le projet du **didacticiel Graph** dans l’Explorateur de solutions et sélectionnez **Ajouter > nouvel élément...**. Choisissez **fichier de configuration Web**, nommez `PrivateSettings.config` le fichier, puis sélectionnez **Ajouter**. Remplacez l'ensemble de son contenu par le code suivant.

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="http://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    Remplacez `YOUR_APP_ID_HERE` par l’ID de l’application dans le portail d’inscription de `YOUR_APP_PASSWORD_HERE` l’application et remplacez par la clé secrète client que vous avez générée. Si votre clé secrète client contient des esperluettes (`&`), veillez à les remplacer par `&amp;` dans `PrivateSettings.config`. Veillez également à modifier la `PORT` valeur de `ida:RedirectUri` pour qu’elle corresponde à l’URL de votre application.

    > [!IMPORTANT]
    > Si vous utilisez le contrôle de code source tel que git, il est maintenant recommandé d’exclure le `PrivateSettings.config` fichier du contrôle de code source afin d’éviter une fuite accidentelle de votre ID d’application et de votre mot de passe.

1. Mise `Web.config` à jour pour charger ce nouveau fichier. Remplacez `<appSettings>` (ligne 7) par ce qui suit:

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a>Mettre en œuvre la connexion

Commencez par initialiser l’intergiciel OWIN pour utiliser l’authentification Azure AD pour l’application.

1. Cliquez avec le bouton droit sur le dossier **App_Start** dans l’Explorateur de solutions et sélectionnez **Ajouter une > classe...**. Nommez le `Startup.Auth.cs` fichier, puis sélectionnez **Ajouter**. Remplacez tout le contenu par le code suivant.

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

    Ce code configure le middleware OWIN avec les valeurs de et définit `PrivateSettings.config` deux méthodes de rappel, `OnAuthenticationFailedAsync` et `OnAuthorizationCodeReceivedAsync`. Ces méthodes de rappel seront appelées lorsque le processus de connexion est renvoyé à partir d’Azure.

1. À présent, `Startup.cs` mettez à jour le `ConfigureAuth` fichier pour appeler la méthode. Remplacez tout le contenu de `Startup.cs` par le code suivant.

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

1. Ajoutez une `Error` action à la `HomeController` classe pour transformer les `message` paramètres `debug` de requête et en `Alert` un objet. Ouvrez `Controllers/HomeController.cs` et ajoutez la fonction suivante.

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. Ajoutez un contrôleur pour gérer la connexion. Cliquez avec le bouton droit sur le dossier **Controllers** dans l’Explorateur de solutions et sélectionnez **Ajouter un contrôleur de >...**. Sélectionnez **contrôleur MVC 5-vide** et sélectionnez **Ajouter**. Nommez le `AccountController` contrôleur et sélectionnez **Ajouter**. Remplacez tout le contenu du fichier par le code suivant.

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

    Définit une `SignIn` action and `SignOut` . L' `SignIn` action vérifie si la demande est déjà authentifiée. Si ce n’est pas le cas, il appelle le middleware OWIN pour authentifier l’utilisateur. L' `SignOut` action appelle l’intergiciel OWIN pour se déconnecter.

1. Enregistrez vos modifications et démarrez le projet. Cliquez sur le bouton de connexion et vous devez être redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et acceptez les autorisations demandées. Le navigateur redirige vers l’application en affichant le jeton.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Une fois que l’utilisateur est connecté, vous pouvez obtenir ses informations à partir de Microsoft Graph.

1. Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > nouveau dossier**. Nommez le `Helpers`dossier.

1. Cliquez avec le bouton droit sur ce nouveau dossier, puis sélectionnez **Ajouter une classe de >.**... Nommez le `GraphHelper.cs` fichier, puis sélectionnez **Ajouter**. Remplacez le contenu de ce fichier par le code suivant.

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

    Cette fonctionnalité implémente `GetUserDetails` la fonction, qui utilise le kit de développement logiciel ( `/me` SDK) Microsoft Graph pour appeler le point de terminaison et renvoyer le résultat.

1. Mettez à `OnAuthorizationCodeReceivedAsync` jour la `App_Start/Startup.Auth.cs` méthode dans pour appeler cette fonction. Ajoutez l’instruction `using` suivante en haut du fichier.

    ```cs
    using graph_tutorial.Helpers;
    ```

1. Remplacez le bloc `try` `OnAuthorizationCodeReceivedAsync` existant par le code suivant.

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

1. Enregistrez vos modifications et démarrez l’application, après vous être connecté, vous devriez voir le nom et l’adresse de messagerie de l’utilisateur au lieu du jeton d’accès.

## <a name="storing-the-tokens"></a>Stockage des jetons

Maintenant que vous pouvez obtenir des jetons, il est temps de mettre en œuvre un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, vous allez utiliser la session pour stocker les jetons. Une application réelle utiliserait une solution de stockage sécurisé plus fiable, comme une base de données. Dans cette section, vous allez:

- Implémentez une classe de magasin de jetons pour sérialiser et stocker le cache de jetons MSAL, ainsi que les détails de l’utilisateur dans la session utilisateur.
- Mettez à jour le code d’authentification pour utiliser la classe de magasin de jetons.
- Mettez à jour la classe de contrôleur de base pour exposer les détails de l’utilisateur stocké à tous les affichages de l’application.

1. Cliquez avec le bouton droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > nouveau dossier**. Nommez le `TokenStorage`dossier.

1. Cliquez avec le bouton droit sur ce nouveau dossier, puis sélectionnez **Ajouter une classe de >.**... Nommez le `SessionTokenStore.cs` fichier, puis sélectionnez **Ajouter**. Remplacez le contenu de ce fichier par le code suivant.

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

1. Ajoutez l’instruction `using` suivante en haut du `App_Start/Startup.Auth.cs` fichier.

    ```cs
    using graph_tutorial.TokenStorage;
    using System.IdentityModel.Claims;
    ```

1. Remplacez la fonction `OnAuthorizationCodeReceivedAsync` existante par ce qui suit.

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
    > Les modifications apportées à cette `OnAuthorizationCodeReceivedAsync` nouvelle version de effectuent les opérations suivantes:
    >
    > - Le code intègre désormais le `ConfidentialClientApplication`cache de jetons de l’utilisateur par défaut `SessionTokenStore` à la classe. La bibliothèque MSAL gérera la logique de stockage des jetons et de l’actualiser si nécessaire.
    > - Le code transmet maintenant les détails de l’utilisateur obtenus de Microsoft Graph `SessionTokenStore` à l’objet à stocker dans la session.
    > - En cas de réussite, le code ne redirige plus, il se contente de le renvoyer. Cela permet au middleware OWIN de terminer le processus d’authentification.

1. Mettre à `SignOut` jour l’action pour effacer le magasin de jetons avant de se déconnecter. Ajoutez l’instruction `using` suivante en haut de `Controllers/AccountController.cs`.

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. Remplacez la fonction `SignOut` existante par ce qui suit.

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

1. Ouvrez `Controllers/BaseController.cs` et ajoutez les instructions `using` suivantes en haut du fichier.

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. Ajoutez la fonction suivante.

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

1. Démarrez le serveur et parcourez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes connecté.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur Avatar de l’utilisateur dans le coin supérieur droit pour **** accéder au lien Déconnexion. Cliquez **** sur Déconnexion pour réinitialiser la session et revenir à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien Déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application a un jeton d’accès, qui est envoyé `Authorization` dans l’en-tête des appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Toutefois, ce jeton est éphémère. Le jeton expire une heure après son émission. C’est ici que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans demander à l’utilisateur de se reconnecter.

Étant donné que l’application utilise la bibliothèque MSAL et `TokenCache` un objet, il n’est pas nécessaire d’implémenter une logique d’actualisation de jeton. La `ConfidentialClientApplication.AcquireTokenSilentAsync` méthode effectue l’ensemble de la logique pour vous. Il vérifie d’abord le jeton mis en cache et, s’il n’a pas expiré, il le renvoie. Si elle a expiré, elle utilise le jeton d’actualisation mis en cache pour en obtenir une nouvelle. Vous utiliserez cette méthode dans le module suivant.
