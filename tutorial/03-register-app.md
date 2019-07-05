<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ee2b4-101">Dans cet exercice, vous allez créer une inscription de l’application Web Azure AD à l’aide du centre d’administration Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="ee2b4-102">Déterminez l’URL SSL de votre application ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-102">Determine your ASP.NET app's SSL URL.</span></span> <span data-ttu-id="ee2b4-103">Dans l’Explorateur de solutions de Visual Studio, sélectionnez le projet du **didacticiel Graph** .</span><span class="sxs-lookup"><span data-stu-id="ee2b4-103">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="ee2b4-104">Dans la fenêtre **Propriétés** , recherchez la valeur de l' **URL SSL**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-104">In the **Properties** window, find the value of **SSL URL**.</span></span> <span data-ttu-id="ee2b4-105">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-105">Copy this value.</span></span>

    ![Capture d’écran de la fenêtre Propriétés de Visual Studio](./images/vs-project-url.png)

1. <span data-ttu-id="ee2b4-107">Ouvrez un navigateur et accédez au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="ee2b4-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="ee2b4-108">Connectez-vous à l’aide d’un **compte personnel** (compte Microsoft) ou d’un **compte professionnel ou scolaire**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-108">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="ee2b4-109">Sélectionnez **Azure Active Directory** dans le volet de navigation de gauche, puis sélectionnez **inscriptions des applications** sous **gérer**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-109">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="ee2b4-110">Capture d’écran des inscriptions d’application</span><span class="sxs-lookup"><span data-stu-id="ee2b4-110">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="ee2b4-111">Sélectionnez **Nouvelle inscription**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-111">Select **New registration**.</span></span> <span data-ttu-id="ee2b4-112">Sur la page **Inscrire une application**, définissez les valeurs comme suit.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="ee2b4-113">Définissez le **Nom** sur `ASP.NET Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-113">Set **Name** to `ASP.NET Graph Tutorial`.</span></span>
    - <span data-ttu-id="ee2b4-114">Définissez les **Types de comptes pris en charge** sur **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-114">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="ee2b4-115">Sous **URI de redirection**, définissez la première flèche déroulante sur `Web`, et la valeur sur l’URL d’application ASP.NET vous avez copiée à l’étape 1.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-115">Under **Redirect URI**, set the first drop-down to `Web` and set the value to the ASP.NET app URL you copied in step 1.</span></span>

    ![Capture d’écran de la page inscrire une application](./images/aad-register-an-app.png)

1. <span data-ttu-id="ee2b4-117">Sélectionnez **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-117">Select **Register**.</span></span> <span data-ttu-id="ee2b4-118">Sur la page **ASP.net Graph Tutorial** , copiez la valeur de l' **ID d’application (client)** et enregistrez-la, vous en aurez besoin à l’étape suivante.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-118">On the **ASP.NET Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Capture d’écran de l’ID d’application de la nouvelle inscription de l’application](./images/aad-application-id.png)

1. <span data-ttu-id="ee2b4-120">Sous **Gérer**, sélectionnez **Authentification**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-120">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="ee2b4-121">Recherchez la section **Octroi implicite**, puis activez **Jetons d’ID**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-121">Locate the **Implicit grant** section and enable **ID tokens**.</span></span> <span data-ttu-id="ee2b4-122">Cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-122">Select **Save**.</span></span>

    ![Capture d’écran de la section Grant implicite](./images/aad-implicit-grant.png)

1. <span data-ttu-id="ee2b4-124">Sélectionnez **Certificats et secrets** sous **Gérer**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-124">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="ee2b4-125">Sélectionnez le bouton **Nouveau secret client**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-125">Select the **New client secret** button.</span></span> <span data-ttu-id="ee2b4-126">Entrez une valeur dans **Description** et sélectionnez l’une des options pour \*\*\*\* expirer, puis sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-126">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    ![Capture d’écran de la boîte de dialogue Ajouter une clé secrète client](./images/aad-new-client-secret.png)

1. <span data-ttu-id="ee2b4-128">Copiez la valeur du secret client avant de quitter cette page.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-128">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="ee2b4-129">Vous en aurez besoin à l’étape suivante.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-129">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="ee2b4-130">Ce secret client n’apparaîtra plus jamais, aussi veillez à le copier maintenant.</span><span class="sxs-lookup"><span data-stu-id="ee2b4-130">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Capture d’écran de la clé secrète client récemment ajoutée](./images/aad-copy-client-secret.png)
