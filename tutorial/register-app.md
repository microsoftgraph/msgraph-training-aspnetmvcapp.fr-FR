<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a1347-101">Dans cet exercice, vous allez créer une inscription de l'application Web Azure AD à l'aide du centre d'administration Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="a1347-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="a1347-102">Déterminez l'URL de votre application ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="a1347-102">Determine your ASP.NET app's URL.</span></span> <span data-ttu-id="a1347-103">Dans l'Explorateur de solutions de Visual Studio, sélectionnez le projet du **didacticiel Graph** .</span><span class="sxs-lookup"><span data-stu-id="a1347-103">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="a1347-104">Dans la fenêtre **Propriétés** , recherchez la valeur de **URL**.</span><span class="sxs-lookup"><span data-stu-id="a1347-104">In the **Properties** window, find the value of **URL**.</span></span> <span data-ttu-id="a1347-105">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="a1347-105">Copy this value.</span></span>

    ![Capture d'écran de la fenêtre Propriétés de Visual Studio](./images/vs-project-url.png)

1. <span data-ttu-id="a1347-107">Ouvrez un navigateur et accédez au [Centre d'administration Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="a1347-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="a1347-108">Connectez-vous à l'aide d'un compte **personnel** (alias Microsoft) ou **compte professionnel ou scolaire**.</span><span class="sxs-lookup"><span data-stu-id="a1347-108">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="a1347-109">Sélectionnez **Azure Active Directory** dans le volet de navigation de gauche, puis sélectionnez **inscriptions des applications (aperçu)** sous **gérer**.</span><span class="sxs-lookup"><span data-stu-id="a1347-109">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations (Preview)** under **Manage**.</span></span>

    ![<span data-ttu-id="a1347-110">Capture d'écran des inscriptions d'application</span><span class="sxs-lookup"><span data-stu-id="a1347-110">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="a1347-111">Sélectionnez **nouvelle inscription**.</span><span class="sxs-lookup"><span data-stu-id="a1347-111">Select **New registration**.</span></span> <span data-ttu-id="a1347-112">Sur la page **inscrire une application** , définissez les valeurs comme suit.</span><span class="sxs-lookup"><span data-stu-id="a1347-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="a1347-113">Définissez **nom** sur `ASP.NET Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="a1347-113">Set **Name** to `ASP.NET Graph Tutorial`.</span></span>
    - <span data-ttu-id="a1347-114">Définissez les types de comptes **pris en charge** sur **les comptes de tous les comptes d'annuaire et de Microsoft personnels**.</span><span class="sxs-lookup"><span data-stu-id="a1347-114">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="a1347-115">Sous **URI**de redirection, définissez la première liste déroulante sur `Web` et définissez la valeur sur l'URL de l'application ASP.net que vous avez copiée à l'étape 1.</span><span class="sxs-lookup"><span data-stu-id="a1347-115">Under **Redirect URI**, set the first drop-down to `Web` and set the value to the ASP.NET app URL you copied in step 1.</span></span>

    ![Capture d'écran de la page inscrire une application](./images/aad-register-an-app.png)

1. <span data-ttu-id="a1347-117">Sélectionnez **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="a1347-117">Choose **Register**.</span></span> <span data-ttu-id="a1347-118">Sur la page **ASP.net Graph Tutorial** , copiez la valeur de l' **ID d'application (client)** et enregistrez-la, vous en aurez besoin à l'étape suivante.</span><span class="sxs-lookup"><span data-stu-id="a1347-118">On the **ASP.NET Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Capture d'écran de l'ID d'application de la nouvelle inscription de l'application](./images/aad-application-id.png)

1. <span data-ttu-id="a1347-120">Sélectionnez **authentification** sous **gérer**.</span><span class="sxs-lookup"><span data-stu-id="a1347-120">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="a1347-121">Recherchez la section **Grant implicite** et activez les **jetons d'ID**.</span><span class="sxs-lookup"><span data-stu-id="a1347-121">Locate the **Implicit grant** section and enable **ID tokens**.</span></span> <span data-ttu-id="a1347-122">Cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="a1347-122">Choose **Save**.</span></span>

    ![Capture d'écran de la section Grant implicite](./images/aad-implicit-grant.png)

1. <span data-ttu-id="a1347-124">Sélectionnez **certificats & secrets** sous **gérer**.</span><span class="sxs-lookup"><span data-stu-id="a1347-124">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="a1347-125">Sélectionnez le bouton **nouveau client secrète** .</span><span class="sxs-lookup"><span data-stu-id="a1347-125">Select the **New client secret** button.</span></span> <span data-ttu-id="a1347-126">Entrez une valeur dans **Description** , puis sélectionnez l'une des options \*\*\*\* pour expirer, puis choisissez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="a1347-126">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Capture d'écran de la boîte de dialogue Ajouter une clé secrète client](./images/aad-new-client-secret.png)

1. <span data-ttu-id="a1347-128">Copiez la valeur de la clé secrète client avant de quitter cette page.</span><span class="sxs-lookup"><span data-stu-id="a1347-128">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="a1347-129">Vous en aurez besoin à l'étape suivante.</span><span class="sxs-lookup"><span data-stu-id="a1347-129">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="a1347-130">Cette clé secrète client ne s'affiche plus, vérifiez que vous la copiez maintenant.</span><span class="sxs-lookup"><span data-stu-id="a1347-130">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Capture d'écran de la clé secrète client récemment ajoutée](./images/aad-copy-client-secret.png)