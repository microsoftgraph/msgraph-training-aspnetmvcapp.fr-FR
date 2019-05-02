<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="03a0e-101">Ce didacticiel vous apprend à créer une application Web ASP.NET MVC qui utilise l’API Microsoft Graph pour récupérer des informations de calendrier pour un utilisateur.</span><span class="sxs-lookup"><span data-stu-id="03a0e-101">This tutorial teaches you how to build an ASP.NET MVC web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="03a0e-102">Si vous préférez télécharger simplement le didacticiel terminé, vous pouvez le télécharger de deux manières.</span><span class="sxs-lookup"><span data-stu-id="03a0e-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="03a0e-103">Téléchargez le [démarrage rapide de ASP.net](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet) pour obtenir du code de travail en quelques minutes.</span><span class="sxs-lookup"><span data-stu-id="03a0e-103">Download the [ASP.NET quick start](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet) to get working code in minutes.</span></span>
> - <span data-ttu-id="03a0e-104">Téléchargez ou clonez le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span><span class="sxs-lookup"><span data-stu-id="03a0e-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="03a0e-105">Conditions préalables</span><span class="sxs-lookup"><span data-stu-id="03a0e-105">Prerequisites</span></span>

<span data-ttu-id="03a0e-106">Avant de commencer ce didacticiel, [Visual Studio](https://visualstudio.microsoft.com/vs/) doit être installé sur votre ordinateur de développement.</span><span class="sxs-lookup"><span data-stu-id="03a0e-106">Before you start this tutorial, you should have [Visual Studio](https://visualstudio.microsoft.com/vs/) installed on your development machine.</span></span> <span data-ttu-id="03a0e-107">Si vous ne disposez pas de Visual Studio, reportez-vous au lien précédent pour obtenir les options de téléchargement.</span><span class="sxs-lookup"><span data-stu-id="03a0e-107">If you do not have Visual Studio, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="03a0e-108">Ce didacticiel est écrit avec Visual Studio 2017 version 15,81.</span><span class="sxs-lookup"><span data-stu-id="03a0e-108">This tutorial was written with Visual Studio 2017 version 15.81.</span></span> <span data-ttu-id="03a0e-109">Les étapes de ce guide peuvent fonctionner avec d’autres versions, mais cela n’a pas été testé.</span><span class="sxs-lookup"><span data-stu-id="03a0e-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="03a0e-110">Commentaires</span><span class="sxs-lookup"><span data-stu-id="03a0e-110">Feedback</span></span>

<span data-ttu-id="03a0e-111">Veuillez fournir des commentaires sur ce didacticiel dans le [référentiel GitHub](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span><span class="sxs-lookup"><span data-stu-id="03a0e-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>