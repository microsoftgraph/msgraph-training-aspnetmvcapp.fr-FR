<!-- markdownlint-disable MD002 MD041 -->

Dans cette démonstration, vous allez incorporer Microsoft Graph dans l'application. Pour cette application, vous allez utiliser la [bibliothèque cliente Microsoft Graph pour .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) pour effectuer des appels à Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtenir des événements de calendrier à partir d'Outlook

Commencez par étendre la `GraphHelper` classe que vous avez créée dans le dernier module. Tout d'abord, ajoutez `using` les instructions suivantes en haut du `Helpers/GraphHelper.cs` fichier.

```cs
using graph_tutorial.TokenStorage;
using Microsoft.Identity.Client;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Security.Claims;
using System.Web;
```

Ajoutez ensuite le code suivant à la `GraphHelper` classe.

```cs
// Load configuration settings from PrivateSettings.config
private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

public static async Task<IEnumerable<Event>> GetEventsAsync()
{
    var graphClient = GetAuthenticatedClient();

    var events = await graphClient.Me.Events.Request()
        .Select("subject,organizer,start,end")
        .OrderBy("createdDateTime DESC")
        .GetAsync();

    return events.CurrentPage;
}

private static GraphServiceClient GetAuthenticatedClient()
{
    return new GraphServiceClient(
        new DelegateAuthenticationProvider(
            async (requestMessage) =>
            {
                // Get the signed in user's id and create a token cache
                string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
                SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId,
                    new HttpContextWrapper(HttpContext.Current));

                var idClient = new ConfidentialClientApplication(
                    appId, redirectUri, new ClientCredential(appSecret),
                    tokenStore.GetMsalCacheInstance(), null);

                var accounts = await idClient.GetAccountsAsync();

                // By calling this here, the token can be refreshed
                // if it's expired right before the Graph call is made
                var result = await idClient.AcquireTokenSilentAsync(
                    graphScopes.Split(' '), accounts.FirstOrDefault());

                requestMessage.Headers.Authorization =
                    new AuthenticationHeaderValue("Bearer", result.AccessToken);
            }));
}
```

Examinez ce que fait ce code.

- La `GetAuthenticatedClient` fonction Initialise un `GraphServiceClient` avec un fournisseur d'authentification qui appelle `AcquireTokenSilentAsync`.
- Dans la `GetEventsAsync` fonction:
  - L'URL qui sera appelée est `/v1.0/me/events`.
  - La `Select` fonction limite les champs renvoyés pour chaque événement à ceux que l'affichage utilise réellement.
  - La `OrderBy` fonction trie les résultats en fonction de la date et de l'heure de leur création, avec l'élément le plus récent en premier.

Créez maintenant un contrôleur pour les affichages de calendrier. Cliquez avec le bouton droit sur le dossier **Controllers** dans l'Explorateur de solutions, puis choisissez **Ajouter un contrôleur >..**.. Sélectionnez **contrôleur MVC 5-vide** et choisissez **Ajouter**. Nommez le `CalendarController` contrôleur, puis choisissez **Ajouter**. Remplacez tout le contenu du nouveau fichier par le code suivant.

```cs
using graph_tutorial.Helpers;
using System.Threading.Tasks;
using System.Web.Mvc;

namespace graph_tutorial.Controllers
{
    public class CalendarController : BaseController
    {
        // GET: Calendar
        [Authorize]
        public async Task<ActionResult> Index()
        {
            var events = await GraphHelper.GetEventsAsync();
            return Json(events, JsonRequestBehavior.AllowGet);
        }
    }
}
```

À présent, vous pouvez le tester. Démarrez l'application, connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation. Si tout fonctionne, vous devez voir un vidage JSON des événements sur le calendrier de l'utilisateur.

## <a name="display-the-results"></a>Afficher les résultats

À présent, vous pouvez ajouter une vue pour afficher les résultats de manière plus conviviale. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur le dossier **views/Calendar** , puis choisissez **Add > View..**.. Nommez l' `Index` affichage, puis choisissez **Ajouter**. Remplacez tout le contenu du nouveau fichier par le code suivant.

```html
@model IEnumerable<Microsoft.Graph.Event>

@{
    ViewBag.Current = "Calendar";
}

<h1>Calendar</h1>
<table class="table">
    <thead>
        <tr>
            <th scope="col">Organizer</th>
            <th scope="col">Subject</th>
            <th scope="col">Start</th>
            <th scope="col">End</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.Organizer.EmailAddress.Name</td>
                <td>@item.Subject</td>
                <td>@Convert.ToDateTime(item.Start.DateTime).ToString("M/d/yy h:mm tt")</td>
                <td>@Convert.ToDateTime(item.End.DateTime).ToString("M/d/yy h:mm tt")</td>
            </tr>
        }
    </tbody>
</table>
```

Cela permet d'exécuter une boucle dans une collection d'événements et d'ajouter une ligne de tableau pour chacun d'eux. Supprimez `return Json(events, JsonRequestBehavior.AllowGet);` la ligne de `Index` la fonction `Controllers/CalendarController.cs`dans et remplacez-la par le code suivant.

```cs
return View(events);
```

Démarrez l'application, connectez-vous, puis cliquez sur le lien **calendrier** . L'application doit maintenant afficher un tableau d'événements.

![Capture d'écran du tableau des événements](./images/add-msgraph-01.png)