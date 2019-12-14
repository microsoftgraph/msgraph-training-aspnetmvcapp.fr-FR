<!-- markdownlint-disable MD002 MD041 -->

Commencez par créer un projet MVC ASP.NET.

1. Ouvrez Visual Studio, puis sélectionnez **créer un nouveau projet**.

1. Dans la boîte de dialogue **créer un nouveau projet** , choisissez l’option **application Web ASP.net (.NET Framework)** qui utilise C#, puis sélectionnez **suivant**.

    ![Boîte de dialogue créer un nouveau projet dans Visual Studio 2019](./images/vs-create-new-project.png)

1. Entrez `graph-tutorial` dans le champ **nom du projet** , puis sélectionnez **créer**.

    ![Boîte de dialogue Configurer un nouveau projet de Visual Studio 2019](./images/vs-configure-new-project.png)

    > [!NOTE]
    > Assurez-vous d’entrer exactement le même nom pour le projet Visual Studio spécifié dans ces instructions d’atelier. Le nom du projet Visual Studio devient partie intégrante de l’espace de noms dans le code. Le code à l’intérieur de ces instructions dépend de l’espace de noms correspondant au nom de projet Visual Studio spécifié dans ces instructions. Si vous utilisez un nom de projet différent, le code n’est pas compilé, sauf si vous ajustez tous les espaces de noms pour qu’ils correspondent au nom de projet Visual Studio que vous entrez lors de la création du projet.

1. Choisissez **MVC** et sélectionnez **créer**.

    ![Visual Studio 2019 créer une nouvelle boîte de dialogue d’application Web ASP.NET](./images/vs-create-new-asp-app.png)

1. Appuyez sur **F5** ou sélectionnez **déboguer > démarrer le débogage**. Si tout fonctionne, votre navigateur par défaut doit ouvrir et afficher une page ASP.NET par défaut.

## <a name="add-nuget-packages"></a>Ajouter des packages NuGet

Avant de poursuivre, mettez à `bootstrap` jour le package NuGet et installez des packages NuGet supplémentaires que vous utiliserez plus tard.

- [Microsoft. Owin. Host. SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb/) pour activer les interfaces [Owin](http://owin.org/) dans l’application ASP.net.
- [Microsoft. Owin. Security. OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) pour effectuer l’authentification OpenID Connect avec Azure.
- [Microsoft. Owin. Security. Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies/) pour activer l’authentification basée sur les cookies.
- [Microsoft. Identity. client](https://www.nuget.org/packages/Microsoft.Identity.Client/) pour la demande et la gestion des jetons d’accès.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels à Microsoft Graph.

1. Sélectionnez **outils > gestionnaire de package NuGet > console Gestionnaire de package**.
1. Dans la console Gestionnaire de package, entrez les commandes suivantes.

    ```Powershell
    Update-Package bootstrap
    Install-Package Microsoft.Owin.Host.SystemWeb
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Identity.Client -Version 4.7.1
    Install-Package Microsoft.Graph -Version 1.20.0
    ```

## <a name="design-the-app"></a>Concevoir l’application

Dans cette section, vous allez créer la structure de base de l’application.

1. Créer une classe de démarrage OWIN de base. Cliquez avec le bouton `graph-tutorial` droit sur le dossier dans l’Explorateur de solutions et sélectionnez **Ajouter > nouvel élément**. Choisissez le modèle de **classe de démarrage OWIN** , nommez le fichier `Startup.cs`, puis sélectionnez **Ajouter**.

1. Cliquez avec le bouton droit sur le dossier **Models** dans l’Explorateur de solutions et sélectionnez **Ajouter une > classe...**. Nommez la `Alert` classe et sélectionnez **Ajouter**. Remplacez tout le contenu de `Alert.cs` par le code suivant.

    ```cs
    namespace graph_tutorial.Models
    {
        // Used to flash error messages in the app's views.
        public class Alert
        {
            public const string AlertKey = "TempDataAlerts";
            public string Message { get; set; }
            public string Debug { get; set; }
        }
    }
    ```

1. Ouvrez le `./Views/Shared/_Layout.cshtml` fichier et remplacez l’intégralité de son contenu par le code suivant pour mettre à jour la disposition globale de l’application.

    ```html
    @{
        var alerts = TempData.ContainsKey(graph_tutorial.Models.Alert.AlertKey) ?
            (List<graph_tutorial.Models.Alert>)TempData[graph_tutorial.Models.Alert.AlertKey] :
            new List<graph_tutorial.Models.Alert>();
    }

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>ASP.NET Graph Tutorial</title>
        @Styles.Render("~/Content/css")
        @Scripts.Render("~/bundles/modernizr")

        <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
              integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt"
              crossorigin="anonymous">
    </head>

    <body>
        <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
            <div class="container">
                @Html.ActionLink("ASP.NET Graph Tutorial", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
                        aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse" id="navbarCollapse">
                    <ul class="navbar-nav mr-auto">
                        <li class="nav-item">
                            @Html.ActionLink("Home", "Index", "Home", new { area = "" },
                                new { @class = ViewBag.Current == "Home" ? "nav-link active" : "nav-link" })
                        </li>
                        @if (Request.IsAuthenticated)
                        {
                            <li class="nav-item" data-turbolinks="false">
                                @Html.ActionLink("Calendar", "Index", "Calendar", new { area = "" },
                                    new { @class = ViewBag.Current == "Calendar" ? "nav-link active" : "nav-link" })
                            </li>
                        }
                    </ul>
                    <ul class="navbar-nav justify-content-end">
                        <li class="nav-item">
                            <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                                <i class="fas fa-external-link-alt mr-1"></i>Docs
                            </a>
                        </li>
                        @if (Request.IsAuthenticated)
                        {
                            <li class="nav-item dropdown">
                                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                                    @if (!string.IsNullOrEmpty(ViewBag.User.Avatar))
                                    {
                                        <img src="@ViewBag.User.Avatar" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                                    }
                                    else
                                    {
                                        <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                                    }
                                </a>
                                <div class="dropdown-menu dropdown-menu-right">
                                    <h5 class="dropdown-item-text mb-0">@ViewBag.User.DisplayName</h5>
                                    <p class="dropdown-item-text text-muted mb-0">@ViewBag.User.Email</p>
                                    <div class="dropdown-divider"></div>
                                    @Html.ActionLink("Sign Out", "SignOut", "Account", new { area = "" }, new { @class = "dropdown-item" })
                                </div>
                            </li>
                        }
                        else
                        {
                            <li class="nav-item">
                                @Html.ActionLink("Sign In", "SignIn", "Account", new { area = "" }, new { @class = "nav-link" })
                            </li>
                        }
                    </ul>
                </div>
            </div>
        </nav>
        <main role="main" class="container">
            @foreach (var alert in alerts)
            {
                <div class="alert alert-danger" role="alert">
                    <p class="mb-3">@alert.Message</p>
                    @if (!string.IsNullOrEmpty(alert.Debug))
                    {
                        <pre class="alert-pre border bg-light p-2"><code>@alert.Debug</code></pre>
                    }
                </div>
            }

            @RenderBody()
        </main>
        @Scripts.Render("~/bundles/jquery")
        @Scripts.Render("~/bundles/bootstrap")
        @RenderSection("scripts", required: false)
    </body>
    </html>
    ```

    > [!NOTE]
    > Ce code ajoute [bootstrap](https://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples. Il définit également une disposition globale avec une barre de navigation et utilise la `Alert` classe pour afficher les alertes.

1. Ouvrez `Content/Site.css` et remplacez l’intégralité de son contenu par le code suivant.

    ```css
    body {
      padding-top: 4.5rem;
    }

    .alert-pre {
      word-wrap: break-word;
      word-break: break-all;
      white-space: pre-wrap;
    }
    ```

1. Ouvrez le `Views/Home/index.cshtml` fichier et remplacez son contenu par ce qui suit.

    ```html
    @{
        ViewBag.Current = "Home";
    }

    <div class="jumbotron">
        <h1>ASP.NET Graph Tutorial</h1>
        <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from ASP.NET</p>
        @if (Request.IsAuthenticated)
        {
            <h4>Welcome @ViewBag.User.DisplayName!</h4>
            <p>Use the navigation bar at the top of the page to get started.</p>
        }
        else
        {
            @Html.ActionLink("Click here to sign in", "SignIn", "Account", new { area = "" }, new { @class = "btn btn-primary btn-large" })
        }
    </div>
    ```

1. Cliquez avec le bouton droit sur le dossier **Controllers** dans l’Explorateur de solutions et sélectionnez **Ajouter un contrôleur de >...**. Sélectionnez **contrôleur MVC 5-vide** et sélectionnez **Ajouter**. Nommez le `BaseController` contrôleur et sélectionnez **Ajouter**. Remplacez le contenu de `BaseController.cs` par le code suivant.

    ```cs
    using graph_tutorial.Models;
    using System.Collections.Generic;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public abstract class BaseController : Controller
        {
            protected void Flash(string message, string debug=null)
            {
                var alerts = TempData.ContainsKey(Alert.AlertKey) ?
                    (List<Alert>)TempData[Alert.AlertKey] :
                    new List<Alert>();

                alerts.Add(new Alert
                {
                    Message = message,
                    Debug = debug
                });

                TempData[Alert.AlertKey] = alerts;
            }
        }
    }
    ```

1. Ouvrez `Controllers/HomeController.cs` et modifiez la `public class HomeController : Controller` ligne comme suit :

    ```cs
    public class HomeController : BaseController
    ```

1. Enregistrez toutes vos modifications et redémarrez le serveur. À présent, l’application doit être très différente.

    ![Capture d’écran de la page d’accueil repensée](./images/create-app-01.png)
