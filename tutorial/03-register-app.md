<!-- markdownlint-disable MD002 MD041 -->

Dans cet exercice, vous allez créer une inscription de l’application Web Azure AD à l’aide du centre d’administration Azure Active Directory.

1. Déterminez l’URL SSL de votre application ASP.NET. Dans l’Explorateur de solutions de Visual Studio, sélectionnez le projet du **didacticiel Graph** . Dans la fenêtre **Propriétés** , recherchez la valeur de l' **URL SSL**. Copiez cette valeur.

    ![Capture d’écran de la fenêtre Propriétés de Visual Studio](./images/vs-project-url.png)

1. Ouvrez un navigateur et accédez au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com). Connectez-vous à l’aide d’un **compte personnel** (compte Microsoft) ou d’un **compte professionnel ou scolaire**.

1. Sélectionnez **Azure Active Directory** dans le volet de navigation de gauche, puis sélectionnez **inscriptions des applications** sous **gérer**.

    ![Capture d’écran des inscriptions d’application ](./images/aad-portal-app-registrations.png)

1. Sélectionnez **Nouvelle inscription**. Sur la page **Inscrire une application**, définissez les valeurs comme suit.

    - Définissez le **Nom** sur `ASP.NET Graph Tutorial`.
    - Définissez les **Types de comptes pris en charge** sur **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
    - Sous **URI de redirection**, définissez la première flèche déroulante sur `Web`, et la valeur sur l’URL d’application ASP.NET vous avez copiée à l’étape 1.

    ![Capture d’écran de la page inscrire une application](./images/aad-register-an-app.png)

1. Sélectionnez **Enregistrer**. Sur la page **ASP.net Graph Tutorial** , copiez la valeur de l' **ID d’application (client)** et enregistrez-la, vous en aurez besoin à l’étape suivante.

    ![Capture d’écran de l’ID d’application de la nouvelle inscription de l’application](./images/aad-application-id.png)

1. Sous **Gérer**, sélectionnez **Authentification**. Recherchez la section **Octroi implicite**, puis activez **Jetons d’ID**. Cliquez sur **Enregistrer**.

    ![Capture d’écran de la section Grant implicite](./images/aad-implicit-grant.png)

1. Sélectionnez **Certificats et secrets** sous **Gérer**. Sélectionnez le bouton **Nouveau secret client**. Entrez une valeur dans **Description** et sélectionnez l’une des options pour **** expirer, puis sélectionnez **Ajouter**.

    ![Capture d’écran de la boîte de dialogue Ajouter une clé secrète client](./images/aad-new-client-secret.png)

1. Copiez la valeur du secret client avant de quitter cette page. Vous en aurez besoin à l’étape suivante.

    > [!IMPORTANT]
    > Ce secret client n’apparaîtra plus jamais, aussi veillez à le copier maintenant.

    ![Capture d’écran de la clé secrète client récemment ajoutée](./images/aad-copy-client-secret.png)
