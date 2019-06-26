<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2f258-101">Dans cette démonstration, vous allez incorporer Microsoft Graph dans l’application.</span><span class="sxs-lookup"><span data-stu-id="2f258-101">In this demo you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="2f258-102">Pour cette application, vous allez utiliser la [bibliothèque cliente Microsoft Graph pour .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) pour effectuer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="2f258-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="2f258-103">Obtenir des événements de calendrier à partir d’Outlook</span><span class="sxs-lookup"><span data-stu-id="2f258-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="2f258-104">Commencez par étendre la `GraphHelper` classe que vous avez créée dans le dernier module.</span><span class="sxs-lookup"><span data-stu-id="2f258-104">Start by extending the `GraphHelper` class you created in the last module.</span></span>

1. <span data-ttu-id="2f258-105">Ajoutez les instructions `using` suivantes en haut du `Helpers/GraphHelper.cs` fichier.</span><span class="sxs-lookup"><span data-stu-id="2f258-105">Add the following `using` statements to the top of the `Helpers/GraphHelper.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using Microsoft.Identity.Client;
    using System.Collections.Generic;
    using System.Configuration;
    using System.Linq;
    using System.Security.Claims;
    using System.Web;
    ```

1. <span data-ttu-id="2f258-106">Ajoutez le code suivant à la `GraphHelper` classe.</span><span class="sxs-lookup"><span data-stu-id="2f258-106">Add the following code to the `GraphHelper` class.</span></span>

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
                    var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                        .WithRedirectUri(redirectUri)
                        .WithClientSecret(appSecret)
                        .Build();

                    string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
                    var tokenStore = new SessionTokenStore(signedInUserId, HttpContext.Current);
                    tokenStore.Initialize(idClient.UserTokenCache);

                    var accounts = await idClient.GetAccountsAsync();

                    // By calling this here, the token can be refreshed
                    // if it's expired right before the Graph call is made
                    var scopes = graphScopes.Split(' ');
                    var result = await idClient.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                        .ExecuteAsync();

                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                }));
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="2f258-107">Examinez ce que fait ce code.</span><span class="sxs-lookup"><span data-stu-id="2f258-107">Consider what this code is doing.</span></span>
    >
    > - <span data-ttu-id="2f258-108">La `GetAuthenticatedClient` fonction Initialise un `GraphServiceClient` avec un fournisseur d’authentification qui appelle `AcquireTokenSilent`.</span><span class="sxs-lookup"><span data-stu-id="2f258-108">The `GetAuthenticatedClient` function initializes a `GraphServiceClient` with an authentication provider that calls `AcquireTokenSilent`.</span></span>
    > - <span data-ttu-id="2f258-109">Dans la `GetEventsAsync` fonction:</span><span class="sxs-lookup"><span data-stu-id="2f258-109">In the `GetEventsAsync` function:</span></span>
    >   - <span data-ttu-id="2f258-110">L’URL qui sera appelée est `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="2f258-110">The URL that will be called is `/v1.0/me/events`.</span></span>
    >   - <span data-ttu-id="2f258-111">La `Select` fonction limite les champs renvoyés pour chaque événement à ceux que l’affichage utilise réellement.</span><span class="sxs-lookup"><span data-stu-id="2f258-111">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="2f258-112">La `OrderBy` fonction trie les résultats en fonction de la date et de l’heure de leur création, avec l’élément le plus récent en premier.</span><span class="sxs-lookup"><span data-stu-id="2f258-112">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="2f258-113">Créez un contrôleur pour les affichages de calendrier.</span><span class="sxs-lookup"><span data-stu-id="2f258-113">Create a controller for the calendar views.</span></span> <span data-ttu-id="2f258-114">Cliquez avec le bouton droit sur le dossier **Controllers** dans l’Explorateur de solutions et sélectionnez **Ajouter un contrôleur de >...**. Sélectionnez **contrôleur MVC 5-vide** et sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="2f258-114">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="2f258-115">Nommez le `CalendarController` contrôleur et sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="2f258-115">Name the controller `CalendarController` and select **Add**.</span></span> <span data-ttu-id="2f258-116">Remplacez tout le contenu du nouveau fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="2f258-116">Replace the entire contents of the new file with the following code.</span></span>

    ```cs
    using System;
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

                // Change start and end dates from UTC to local time
                foreach (var ev in events)
                {
                    ev.Start.DateTime = DateTime.Parse(ev.Start.DateTime).ToLocalTime().ToString();
                    ev.Start.TimeZone = TimeZoneInfo.Local.Id;
                    ev.End.DateTime = DateTime.Parse(ev.End.DateTime).ToLocalTime().ToString();
                    ev.End.TimeZone = TimeZoneInfo.Local.Id;
                }

                return Json(events, JsonRequestBehavior.AllowGet);
            }
        }
    }
    ```

1. <span data-ttu-id="2f258-117">Démarrez l’application, connectez-vous, puis cliquez sur le lien **calendrier** dans la barre de navigation.</span><span class="sxs-lookup"><span data-stu-id="2f258-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="2f258-118">Si tout fonctionne, vous devez voir un vidage JSON des événements sur le calendrier de l’utilisateur.</span><span class="sxs-lookup"><span data-stu-id="2f258-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="2f258-119">Afficher les résultats</span><span class="sxs-lookup"><span data-stu-id="2f258-119">Display the results</span></span>

<span data-ttu-id="2f258-120">À présent, vous pouvez ajouter une vue pour afficher les résultats de manière plus conviviale.</span><span class="sxs-lookup"><span data-stu-id="2f258-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="2f258-121">Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le dossier **views/Calendar** et sélectionnez **Add > View...**. Nommez l' `Index` affichage et sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="2f258-121">In Solution Explorer, right-click the **Views/Calendar** folder and select **Add > View...**. Name the view `Index` and select **Add**.</span></span> <span data-ttu-id="2f258-122">Remplacez tout le contenu du nouveau fichier par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="2f258-122">Replace the entire contents of the new file with the following code.</span></span>

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

    <span data-ttu-id="2f258-123">Cela permet d’exécuter une boucle dans une collection d’événements et d’ajouter une ligne de tableau pour chacun d’eux.</span><span class="sxs-lookup"><span data-stu-id="2f258-123">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="2f258-124">Supprimez `return Json(events, JsonRequestBehavior.AllowGet);` la ligne de `Index` la fonction `Controllers/CalendarController.cs`dans et remplacez-la par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="2f258-124">Remove the `return Json(events, JsonRequestBehavior.AllowGet);` line from the `Index` function in `Controllers/CalendarController.cs`, and replace it with the following code.</span></span>

    ```cs
    return View(events);
    ```

1. <span data-ttu-id="2f258-125">Démarrez l’application, connectez-vous, puis cliquez sur le lien **calendrier** .</span><span class="sxs-lookup"><span data-stu-id="2f258-125">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="2f258-126">L’application doit maintenant afficher un tableau d’événements.</span><span class="sxs-lookup"><span data-stu-id="2f258-126">The app should now render a table of events.</span></span>

    ![Capture d’écran du tableau des événements](./images/add-msgraph-01.png)
