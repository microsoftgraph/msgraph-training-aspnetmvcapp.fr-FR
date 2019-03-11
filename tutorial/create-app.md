<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="00915-101">Ouvrez Visual Studio, puis sélectionnez **fichier _GT_ nouveau projet >**.</span><span class="sxs-lookup"><span data-stu-id="00915-101">Open Visual Studio, and select **File > New > Project**.</span></span> <span data-ttu-id="00915-102">Dans la boîte de dialogue **nouveau projet** , procédez comme suit:</span><span class="sxs-lookup"><span data-stu-id="00915-102">In the **New Project** dialog, do the following:</span></span>

1. <span data-ttu-id="00915-103">Sélectionnez **modèles _GT_ Visual C# _GT_ Web**.</span><span class="sxs-lookup"><span data-stu-id="00915-103">Select **Templates > Visual C# > Web**.</span></span>
1. <span data-ttu-id="00915-104">Sélectionnez **application Web ASP.net (.NET Framework)**.</span><span class="sxs-lookup"><span data-stu-id="00915-104">Select **ASP.NET Web Application (.NET Framework)**.</span></span>
1. <span data-ttu-id="00915-105">Entrez **un didacticiel** pour le nom du projet.</span><span class="sxs-lookup"><span data-stu-id="00915-105">Enter **graph-tutorial** for the Name of the project.</span></span>

![Boîte de dialogue créer un nouveau projet dans Visual Studio 2017](./images/vs-new-project-01.png)

> [!NOTE]
> <span data-ttu-id="00915-107">Assurez-vous d'entrer exactement le même nom pour le projet Visual Studio spécifié dans ces instructions d'atelier.</span><span class="sxs-lookup"><span data-stu-id="00915-107">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="00915-108">Le nom du projet Visual Studio devient partie intégrante de l'espace de noms dans le code.</span><span class="sxs-lookup"><span data-stu-id="00915-108">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="00915-109">Le code à l'intérieur de ces instructions dépend de l'espace de noms correspondant au nom de projet Visual Studio spécifié dans ces instructions.</span><span class="sxs-lookup"><span data-stu-id="00915-109">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="00915-110">Si vous utilisez un nom de projet différent, le code n'est pas compilé, sauf si vous ajustez tous les espaces de noms pour qu'ils correspondent au nom de projet Visual Studio que vous entrez lors de la création du projet.</span><span class="sxs-lookup"><span data-stu-id="00915-110">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

<span data-ttu-id="00915-111">Sélectionnez **OK**.</span><span class="sxs-lookup"><span data-stu-id="00915-111">Select **OK**.</span></span> <span data-ttu-id="00915-112">Dans la boîte de dialogue **nouveau projet d'application Web ASP.net** , sélectionnez **MVC** (sous **ASP.net modèles de 4.7.2**), puis cliquez sur **OK**.</span><span class="sxs-lookup"><span data-stu-id="00915-112">In the **New ASP.NET Web Application Project** dialog, select **MVC** (under **ASP.NET 4.7.2 Templates**) and select **OK**.</span></span>

<span data-ttu-id="00915-113">Appuyez sur **F5** ou sélectionnez **débogage > démarrer**le débogage.</span><span class="sxs-lookup"><span data-stu-id="00915-113">Press **F5** or select **Debug > Start Debugging**.</span></span> <span data-ttu-id="00915-114">Si tout fonctionne, votre navigateur par défaut doit ouvrir et afficher une page ASP.NET par défaut.</span><span class="sxs-lookup"><span data-stu-id="00915-114">If everything is working, your default browser should open and display a default ASP.NET page.</span></span>

<span data-ttu-id="00915-115">Avant de poursuivre, mettez à `bootstrap` jour le package NuGet et installez des packages NuGet supplémentaires que vous utiliserez plus tard.</span><span class="sxs-lookup"><span data-stu-id="00915-115">Before moving on, update the `bootstrap` NuGet package, and install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="00915-116">[Microsoft. Owin. Host. SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb/) pour activer les interfaces [Owin](http://owin.org/) dans l'application ASP.net.</span><span class="sxs-lookup"><span data-stu-id="00915-116">[Microsoft.Owin.Host.SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb/) to enable the [OWIN](http://owin.org/) interfaces in the ASP.NET application.</span></span>
- <span data-ttu-id="00915-117">[Microsoft. Owin. Security. OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) pour effectuer l'authentification OpenID Connect avec Azure.</span><span class="sxs-lookup"><span data-stu-id="00915-117">[Microsoft.Owin.Security.OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/) for doing OpenID Connect authentication with Azure.</span></span>
- <span data-ttu-id="00915-118">[Microsoft. Owin. Security. Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies/) pour activer l'authentification basée sur les cookies.</span><span class="sxs-lookup"><span data-stu-id="00915-118">[Microsoft.Owin.Security.Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies/) to enable cookie-based authentication.</span></span>
- <span data-ttu-id="00915-119">[Microsoft. Identity. client](https://www.nuget.org/packages/Microsoft.Identity.Client/) pour la demande et la gestion des jetons d'accès.</span><span class="sxs-lookup"><span data-stu-id="00915-119">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="00915-120">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) pour effectuer des appels à Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="00915-120">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>

<span data-ttu-id="00915-121">Sélectionnez **Outils _GT_ gestionnaire de package NuGet _GT_ console Gestionnaire de package**.</span><span class="sxs-lookup"><span data-stu-id="00915-121">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span> <span data-ttu-id="00915-122">Dans la console Gestionnaire de package, entrez les commandes suivantes.</span><span class="sxs-lookup"><span data-stu-id="00915-122">In the Package Manager Console, enter the following commands.</span></span>

```Powershell
Update-Package bootstrap
Install-Package Microsoft.Owin.Host.SystemWeb
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Identity.Client -Version 2.1.0-preview
Install-Package Microsoft.Graph -Version 1.11.0
```

<span data-ttu-id="00915-123">Créer une classe de démarrage OWIN de base.</span><span class="sxs-lookup"><span data-stu-id="00915-123">Create a basic OWIN startup class.</span></span> <span data-ttu-id="00915-124">Cliquez avec le bouton `graph-tutorial` droit sur le dossier dans l'Explorateur de solutions et choisissez **Ajouter un nouvel élément >**.</span><span class="sxs-lookup"><span data-stu-id="00915-124">Right-click the `graph-tutorial` folder in Solution Explorer and choose **Add > New Item**.</span></span> <span data-ttu-id="00915-125">Choisissez le modèle de **classe de démarrage OWIN** , nommez le fichier `Startup.cs`, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="00915-125">Choose the **OWIN Startup Class** template, name the file `Startup.cs`, and choose **Add**.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="00915-126">Concevoir l'application</span><span class="sxs-lookup"><span data-stu-id="00915-126">Design the app</span></span>

<span data-ttu-id="00915-127">Commencez par créer un modèle simple pour un message d'erreur.</span><span class="sxs-lookup"><span data-stu-id="00915-127">Start by creating a simple model for an error message.</span></span> <span data-ttu-id="00915-128">Vous utiliserez ce modèle pour flasher les messages d'erreur dans les affichages de l'application.</span><span class="sxs-lookup"><span data-stu-id="00915-128">You'll use this model to flash error messages in the app's views.</span></span>

<span data-ttu-id="00915-129">Cliquez avec le bouton droit sur le dossier **modèles** dans l'Explorateur de solutions et choisissez **Ajouter une classe >...**. Nommez la `Alert` classe, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="00915-129">Right-click the **Models** folder in Solution Explorer and choose **Add > Class...**. Name the class `Alert` and choose **Add**.</span></span> <span data-ttu-id="00915-130">Ajoutez le code suivant dans `Alert.cs`.</span><span class="sxs-lookup"><span data-stu-id="00915-130">Add the following code in `Alert.cs`.</span></span>

```cs
namespace graph_tutorial.Models
{
    public class Alert
    {
        public const string AlertKey = "TempDataAlerts";
        public string Message { get; set; }
        public string Debug { get; set; }
    }
}
```

<span data-ttu-id="00915-131">À présent, mettez à jour la disposition globale de l'application.</span><span class="sxs-lookup"><span data-stu-id="00915-131">Now update the global layout of the app.</span></span> <span data-ttu-id="00915-132">Ouvrez le `./Views/Shared/_Layout.cshtml` fichier et remplacez l'intégralité de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="00915-132">Open the `./Views/Shared/_Layout.cshtml` file, and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="00915-133">Ce code ajoute [bootstrap](https://getbootstrap.com/) pour les styles simples et [font Isard](https://fontawesome.com/) pour certaines icônes simples.</span><span class="sxs-lookup"><span data-stu-id="00915-133">This code adds [Bootstrap](https://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="00915-134">Il définit également une disposition globale avec une barre de navigation et utilise la `Alert` classe pour afficher les alertes.</span><span class="sxs-lookup"><span data-stu-id="00915-134">It also defines a global layout with a nav bar, and uses the `Alert` class to display any alerts.</span></span>

<span data-ttu-id="00915-135">Maintenant, `Content/Site.css` ouvrez et remplacez l'intégralité de son contenu par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="00915-135">Now open `Content/Site.css` and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="00915-136">À présent, mettez à jour la page par défaut.</span><span class="sxs-lookup"><span data-stu-id="00915-136">Now update the default page.</span></span> <span data-ttu-id="00915-137">Ouvrez le `Views/Home/index.cshtml` fichier et remplacez son contenu par ce qui suit.</span><span class="sxs-lookup"><span data-stu-id="00915-137">Open the `Views/Home/index.cshtml` file and replace its contents with the following.</span></span>

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

<span data-ttu-id="00915-138">À présent, ajoutez une fonction d'assistance pour `Alert` créer un et le transmettre à la vue.</span><span class="sxs-lookup"><span data-stu-id="00915-138">Now add a helper function to create an `Alert` and pass it to the view.</span></span> <span data-ttu-id="00915-139">Pour faciliter la disponibilité de tout contrôleur que nous créons, définissez une classe de contrôleur de base.</span><span class="sxs-lookup"><span data-stu-id="00915-139">In order to make it easily available to any controller we create, define a base controller class.</span></span>

<span data-ttu-id="00915-140">Cliquez avec le bouton droit sur le dossier **Controllers** dans l'Explorateur de solutions, puis choisissez **Ajouter un contrôleur >..**.. Sélectionnez **contrôleur MVC 5-vide** et choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="00915-140">Right-click the **Controllers** folder in Solution Explorer and choose **Add > Controller...**. Choose **MVC 5 Controller - Empty** and choose **Add**.</span></span> <span data-ttu-id="00915-141">Nommez le `BaseController` contrôleur, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="00915-141">Name the controller `BaseController` and choose **Add**.</span></span> <span data-ttu-id="00915-142">Remplacez le contenu de `BaseController.cs` par le code suivant.</span><span class="sxs-lookup"><span data-stu-id="00915-142">Replace the contents of `BaseController.cs` with the following code.</span></span>

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

<span data-ttu-id="00915-143">Tout contrôleur peut hériter de cette classe de contrôleur de base pour accéder `Flash` à la fonction.</span><span class="sxs-lookup"><span data-stu-id="00915-143">Any controller can inherit from this base controller class to gain access to the `Flash` function.</span></span> <span data-ttu-id="00915-144">Mettez à `HomeController` jour la classe à `BaseController`partir de laquelle hériter.</span><span class="sxs-lookup"><span data-stu-id="00915-144">Update the `HomeController` class to inherit from `BaseController`.</span></span> <span data-ttu-id="00915-145">Ouvrez `Controllers/HomeController.cs` et modifiez la `public class HomeController : Controller` ligne comme suit:</span><span class="sxs-lookup"><span data-stu-id="00915-145">Open `Controllers/HomeController.cs` and change the `public class HomeController : Controller` line to:</span></span>

```cs
public class HomeController : BaseController
```

<span data-ttu-id="00915-146">Enregistrez toutes vos modifications et redémarrez le serveur.</span><span class="sxs-lookup"><span data-stu-id="00915-146">Save all of your changes and restart the server.</span></span> <span data-ttu-id="00915-147">À présent, l'application doit être très différente.</span><span class="sxs-lookup"><span data-stu-id="00915-147">Now, the app should look very different.</span></span>

![Capture d'écran de la page d'accueil repensée](./images/create-app-01.png)