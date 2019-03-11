<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7c3f2-101">Dans cet exercice, vous allez créer une inscription de l'application Web Azure AD à l'aide du portail de registre d'applications (ARP).</span><span class="sxs-lookup"><span data-stu-id="7c3f2-101">In this exercise, you will create a new Azure AD web application registration using the Application Registry Portal (ARP).</span></span>

1. <span data-ttu-id="7c3f2-102">Ouvrez un navigateur et accédez au [portail d'inscription des applications](https://apps.dev.microsoft.com).</span><span class="sxs-lookup"><span data-stu-id="7c3f2-102">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="7c3f2-103">Connectez-vous à l'aide d'un compte **personnel** (alias Microsoft) ou **compte professionnel ou scolaire**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="7c3f2-104">Sélectionnez **Ajouter une application** en haut de la page.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-104">Select **Add an app** at the top of the page.</span></span>

    > [!NOTE]
    > <span data-ttu-id="7c3f2-105">Si vous voyez plus d'un bouton **Ajouter une application** sur la page, sélectionnez celui qui correspond à la liste **applications** convergées.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-105">If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="7c3f2-106">Sur la page **inscrire votre application** , définissez le **nom** de l'application sur **ASP.net Graph Tutorial** et sélectionnez **créer**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-106">On the **Register your application** page, set the **Application Name** to **ASP.NET Graph Tutorial** and select **Create**.</span></span>

    ![Capture d'écran de la création d'une nouvelle application dans le site Web du portail d'inscription des applications](./images/arp-create-app-01.png)

1. <span data-ttu-id="7c3f2-108">Sur la page **d'inscription du didacticiel de Graph ASP.net** , dans la section **Propriétés** , copiez l'ID de l' **application** , car vous en aurez besoin plus tard.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-108">On the **ASP.NET Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![Capture d'écran de l'ID de l'application nouvellement créée](./images/arp-create-app-02.png)

1. <span data-ttu-id="7c3f2-110">Faites déFiler jusqu'à la section secrets de l' **application** .</span><span class="sxs-lookup"><span data-stu-id="7c3f2-110">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="7c3f2-111">Sélectionnez **générer un nouveau mot de passe**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-111">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="7c3f2-112">Dans la boîte de dialogue **nouveau mot de passe généré** , copiez le contenu de la zone, car vous en aurez besoin plus tard.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-112">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="7c3f2-113">**Important:** Ce mot de passe ne s'affiche plus, vérifiez que vous le copiez maintenant.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-113">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![Capture d'écran du mot de passe d'une application nouvellement créée](./images/arp-create-app-03.png)

1. <span data-ttu-id="7c3f2-115">Déterminez l'URL de votre application ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-115">Determine your ASP.NET app's URL.</span></span> <span data-ttu-id="7c3f2-116">Dans l'Explorateur de solutions de Visual Studio, sélectionnez le projet du **didacticiel Graph** .</span><span class="sxs-lookup"><span data-stu-id="7c3f2-116">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="7c3f2-117">Dans la fenêtre **Propriétés** , recherchez la valeur de **URL**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-117">In the **Properties** window, find the value of **URL**.</span></span> <span data-ttu-id="7c3f2-118">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-118">Copy this value.</span></span>

    ![Capture d'écran de la fenêtre Propriétés de Visual Studio](./images/vs-project-url.png)

1. <span data-ttu-id="7c3f2-120">Faites déFiler \*\*\*\* vers le bas jusqu'à la section plateformes.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-120">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="7c3f2-121">Sélectionnez **Ajouter une plateforme**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-121">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="7c3f2-122">Dans la boîte de dialogue **Ajouter une plateforme** , sélectionnez **Web**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-122">In the **Add Platform** dialog, select **Web**.</span></span>

        ![Capture d'écran création d'une plateforme pour l'application](./images/arp-create-app-04.png)

    1. <span data-ttu-id="7c3f2-124">Dans la zone plateforme **Web** , entrez l'URL que vous avez copiée à partir des propriétés du projet Visual Studio pour les **URL**de redirection.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-124">In the **Web** platform box, enter the URL you copied from the Visual Studio project's properties for the **Redirect URLs**.</span></span>

        ![Capture d'écran de la plateforme Web récemment ajoutée pour l'application](./images/arp-create-app-05.png)

1. <span data-ttu-id="7c3f2-126">Faites déFiler la page jusqu'en bas et sélectionnez **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="7c3f2-126">Scroll to the bottom of the page and select **Save**.</span></span>