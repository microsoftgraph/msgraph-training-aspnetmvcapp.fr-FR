<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez étendre l’application de l’exercice précédent pour prendre en charge l’authentification avec Azure AD. Cette opération est obligatoire pour obtenir le jeton d’accès OAuth nécessaire pour appeler l’API Microsoft Graph. Dans cette étape, vous allez intégrer l’intermédiaire OWIN et la bibliothèque bibliothèque d’authentification [Microsoft](https://www.nuget.org/packages/Microsoft.Identity.Client/) dans l’application.

1. Cliquez avec le bouton droit sur le projet de didacticiel **graphique** dans l’Explorateur de solutions et **sélectionnez Ajouter > nouvel élément...**. Choose **Web Configuration File**, name the file and select `PrivateSettings.config` **Add**. Remplacez l'ensemble de son contenu par le code suivant.

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    Remplacez-le par l’ID de l’application à partir du portail d’inscription des applications et par la `YOUR_APP_ID_HERE` secret client que vous avez `YOUR_APP_PASSWORD_HERE` générée. Si votre secret client contient des signes esperluette (`&`), n’oubliez pas de les remplacer par des `&amp;` dans `PrivateSettings.config`. Assurez-vous également de modifier la valeur `PORT` en `ida:RedirectUri` afin qu’elle corresponde à l’URL de votre application.

    > [!IMPORTANT]
    > Si vous utilisez un contrôle source tel que Git, il est temps d’exclure le fichier du contrôle source afin d’éviter toute fuite accidentelle de votre ID d’application et de votre mot de `PrivateSettings.config` passe.

1. Mise `Web.config` à jour pour charger ce nouveau fichier. Remplacez `<appSettings>` (ligne 7) par ce qui suit.

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a>Implémentation de la connexion

Commencez par initialiser l’intergiciel OWIN pour utiliser l’authentification Azure AD pour l’application.

1. Cliquez avec le bouton **droit sur App_Start** dossier dans l’Explorateur de solutions et **sélectionnez Ajouter > classe...**. Nommez le fichier `Startup.Auth.cs` et sélectionnez **Ajouter.** Remplacez tout le contenu par le code suivant.

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
    > Ce code configure l’middleware OWIN avec les valeurs de et définit deux méthodes de `PrivateSettings.config` rappel, `OnAuthenticationFailedAsync` et `OnAuthorizationCodeReceivedAsync` . Ces méthodes de rappel sont appelées lorsque le processus de connexion renvoie à partir d’Azure.

1. Maintenant, mettez à `Startup.cs` jour le fichier pour appeler la `ConfigureAuth` méthode. Remplacez l’intégralité `Startup.cs` du contenu par le code suivant.

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

1. Ajoutez une action `Error` à la classe `HomeController` pour transformer le `message` et `debug` les paramètres de requête en un objet `Alert`. Ouvrez `Controllers/HomeController.cs` et ajoutez la fonction suivante.

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. Ajouter un contrôleur pour gérer la connexion. Faites un clic droit sur le dossier **Contrôleurs** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Contrôleur...**. Sélectionnez **Contrôleur MVC 5 - Vide**, puis sélectionnez **Ajouter**. Nommez le contrôleur `AccountController`, puis sélectionnez **Ajouter**. Remplacez tout le contenu du fichier par le code suivant.

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

    Cela définit une action `SignIn` et une action `SignOut`. L’action `SignIn` vérifie si la demande est déjà authentifiée. Si ce n’est pas le cas, il appelle l’intergiciel OWIN pour authentifier l’utilisateur. L’action `SignOut` appelle l’intergiciel OWIN pour se déconnecter.

1. Enregistrez vos modifications et démarrez le projet. Cliquez sur le bouton de connexion. vous serez redirigé vers `https://login.microsoftonline.com`. Connectez-vous avec votre compte Microsoft et consentez aux autorisations demandées. Le navigateur vous redirige vers l’application, affichant le jeton.

### <a name="get-user-details"></a>Obtenir les détails de l’utilisateur

Une fois que l’utilisateur a ouvert une session, vous pouvez obtenir ses informations à partir de Microsoft Graph.

1. Faites un clic droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Nouveau dossier**. Nommer le dossier `Helpers`.

1. Cliquez avec le bouton droit sur ce nouveau dossier et **sélectionnez > classe...**. Nommez le fichier `GraphHelper.cs` et sélectionnez **Ajouter.** Remplacez le contenu du fichier par le code suivant.

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

    Cette opération implémente la fonction `GetUserDetails`, qui utilise le kit de développement logiciel Microsoft Graph pour appeler le point de terminaison `/me` et renvoyer le résultat.

1. Mettez à `OnAuthorizationCodeReceivedAsync` jour la méthode pour appeler cette `App_Start/Startup.Auth.cs` fonction. Ajoutez l’instruction `using` suivante en haut du fichier.

    ```cs
    using graph_tutorial.Helpers;
    ```

1. Remplacez le bloc `try` existant dans `OnAuthorizationCodeReceivedAsync` par le code suivant.

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

1. Enregistrer vos modifications et démarrer l’application. Après vous être connecté, vous devriez voir le nom et l’adresse de messagerie de l’utilisateur au lieu du jeton d’accès.

## <a name="storing-the-tokens"></a>Stockage des jetons

Maintenant que vous pouvez obtenir des jetons, nous vous conseillons d’implémenter un moyen de les stocker dans l’application. Étant donné qu’il s’agit d’un exemple d’application, vous utiliserez la session pour stocker les jetons. Une application réelle utilise une solution de stockage sécurisé plus fiable, comme une base de données. Dans cette section, vous allez :

- Implémenter une classe stockage de jetons pour sérialiser et stocker le cache de jetons MSAL et les détails de l’utilisateur dans la session utilisateur.
- Mettre à jour le code d’authentification pour utiliser la classe stockage de jetons.
- Mettre à jour la classe contrôleur de base pour exposer les détails stockés de l’utilisateur, à tous les affichages dans l’application.

1. Faites un clic droit sur le dossier **Graph-Tutorial** dans l’Explorateur de solutions, puis sélectionnez **Ajouter > Nouveau dossier**. Nommer le dossier `TokenStorage`.

1. Cliquez avec le bouton droit sur ce nouveau dossier et **sélectionnez > classe...**. Nommez le fichier `SessionTokenStore.cs` et sélectionnez **Ajouter.** Remplacez le contenu du fichier par le code suivant.

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

1. Ajoutez les `using` instructions suivantes en haut du `App_Start/Startup.Auth.cs` fichier.

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. Remplacer la fonction `OnAuthorizationCodeReceivedAsync` existante par ce qui suit.

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
    > Les modifications apportées à cette nouvelle version de `OnAuthorizationCodeReceivedAsync` apportent ces changements :
    >
    > - Le code encapsule désormais le cache de jetons utilisateur par défaut de `ConfidentialClientApplication`avec la classe `SessionTokenStore`. La bibliothèque MSAL gère la logique de stockage des jetons et les actualise en cas de besoin.
    > - Le code transmet désormais les informations utilisateur obtenues à partir de Microsoft Graph à l’objet `SessionTokenStore` à stocker dans la session.
    > - En cas de réussite, le code ne redirige plus, il renvoie simplement. Cela permet à l’intergiciel OWIN de terminer le processus d’authentification.

1. Mettez à jour `SignOut` l’action pour effacer le magasin de jetons avant la signature. Ajoutez `using` l’instruction suivante en haut de `Controllers/AccountController.cs` .

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. Remplacez la fonction `SignOut` existante par ce qui suit.

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

1. Ouvrez `Controllers/BaseController.cs` et ajoutez les `using` instructions suivantes en haut du fichier.

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

1. Démarrez le serveur et suivez le processus de connexion. Vous devez revenir sur la page d’accueil, mais l’interface utilisateur doit changer pour indiquer que vous êtes en cours de signature.

    ![Capture d’écran de la page d’accueil après la connexion](./images/add-aad-auth-01.png)

1. Cliquez sur l’avatar de l’utilisateur dans le coin supérieur droit pour accéder au lien **de** connexion. Le fait de cliquer sur **Se déconnecter** réinitialise la session et vous ramène à la page d’accueil.

    ![Capture d’écran du menu déroulant avec le lien de déconnexion](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualisation des jetons

À ce stade, votre application dispose d’un jeton d’accès, qui est envoyé dans l’en-tête des `Authorization` appels d’API. Il s’agit du jeton qui permet à l’application d’accéder à Microsoft Graph pour le compte de l’utilisateur.

Cependant, ce jeton est de courte durée. Le jeton expire une heure après son émission. C’est là que le jeton d’actualisation devient utile. Le jeton d’actualisation permet à l’application de demander un nouveau jeton d’accès sans obliger l’utilisateur à se reconnecter.

Étant donné que l’application utilise la bibliothèque MSAL et sérialise l’objet, vous n’avez pas besoin d’implémenter de logique d’actualisation `TokenCache` de jeton. La méthode `ConfidentialClientApplication.AcquireTokenSilentAsync` fait la logique pour vous. Il vérifie d’abord le jeton mis en cache et, s’il n’a pas expiré, il le renvoie. S’il a expiré, il utilise le jeton d’actualisation mis en cache pour en obtenir un nouveau. Vous utiliserez cette méthode dans le module suivant.
