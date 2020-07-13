# Iniciar sesión con Azure Active Directory (para WordPress)

Un complemento de WordPress que permite a las organizaciones usar sus cuentas de usuario de Azure Active Directory para iniciar sesión en WordPress. Las organizaciones con Office 365 ya tienen Azure Active Directory (Azure AD) y pueden usar este complemento para todos sus usuarios.

- La pertenencia a grupos de Azure AD se puede usar para determinar el acceso y el rol.
- Los nuevos usuarios pueden registrarse sobre la marcha en función de su perfil de Azure AD.
- Siempre puede recurrir a un nombre de usuario y contraseña normales.
*Este es un trabajo en progreso, no dude en ponerse en contacto conmigo para obtener ayuda. Este complemento se proporciona tal cual, sin garantías ni garantías.*

En el flujo típico:

1. El usuario intenta iniciar sesión en el blog (wp-admin). En la página de inicio de sesión, se les proporciona un enlace para iniciar sesión con su cuenta profesional o educativa de Azure Active Directory (por ejemplo, una cuenta de Office 365).
2. Después de iniciar sesión, el usuario es redirigido de nuevo al blog con un código de autorización, que el complemento intercambia por un token de ID, que contiene un conjunto mínimo de reclamos sobre el usuario que inició sesión, y un token de acceso, que puede usarse para consultar Azure AD para detalles adicionales sobre el usuario.
3. El complemento utiliza los reclamos en el token de ID para intentar encontrar un usuario de WordPress con una dirección de correo electrónico o nombre de inicio de sesión que coincida con el usuario de Azure AD.
4. Si se encuentra uno, el usuario se autentica en WordPress como esa cuenta de usuario. Si no se encuentra uno, el usuario de WordPress (opcionalmente) se aprovisionará automáticamente sobre la marcha.


## Para empezar

Las siguientes instrucciones lo ayudarán a comenzar. En este caso, configuraremos el complemento para usar los roles de usuario configurados en WordPress.

### 1. Descargue y active el complemento

Este complemento aún no está registrado en el directorio de complementos de WordPress (¡próximamente!), Pero aún puede instalarlo manualmente:

1. Descargue el complemento usando git o con el enlace 'Descargar ZIP' a la derecha.
2. Ingresar al administrador del sitio con wordpress y darle agregar nuevo plugin, subir.
3. Elegir el archivo zip, subirlo y activarlo.

### 2. Registre una aplicación de Azure Active Directory

Con estos pasos, creará un registro de la aplicación Azure AD. Esto proporcionará a su sitio de WordPress una identidad de aplicación en el inquilino de Azure AD de su organización.

1. Sign in to the [**Azure portal**](https://portal.azure.com), and ensure you are signed in to the directory which has the users you'd like to allow to sign in. (This will typically be your organization's directory.) You can view which directory you're signed in to (and switch directories if needed) by clicking on your username in the upper right-hand corner.

2. Navigate to the [**Azure Active Directory**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade) blade, and enter the [**App registrations**](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) section.

    ![Clicking Azure Active Directory](https://user-images.githubusercontent.com/231140/29496874-6cf6f722-85dc-11e7-8898-89db80593ffc.png) <br />
    ![Clicking App registrations](https://user-images.githubusercontent.com/231140/29496884-9b3693ae-85dc-11e7-89a0-77e80979af23.png)

3. Choose **New registration**.

    ![Clicking New registration](https://user-images.githubusercontent.com/231140/66044424-cf882b80-e521-11e9-9f76-1e0d83ff8467.png)<br />

4. Fill out the initial form as follows:

    * **Name**: Enter your site's name. This will be displayed to users at the Azure AD sign-in page, in the sign-in logs, and in any consent prompt users may come across.

    * **Supported account types**: Choose "Accounts in this organizational directory" if you only expect users from one organization to sign in to your app. Otherwise, choose "Accounts in any organizational directory" to allow users from _any_ Azure AD tenant to sign in.

      > **Note**: This plugin does not yet support the third option, "Accounts in any organizational directory and personal Microsoft accounts".

    * **Redirect URI**: Leave the redirect URI type set to "Web", and provide a URL matching the format `https://<your blog url>/wp-login.php`, or whichever page your blog uses to sign in users.

      > **Note**: If you're not sure what to enter here, you can leave it empty for now and come back and update this (under Azure AD > App registrations > Authentication) later. The plugin itself will tell you what URL to use.

      > **Note**: The page must invoke the `authenticate` action. (By default, this will be `wp-login.php`.)

4. After clicking **Register**, enter the **API permissions** section.

    ![API permissions](https://user-images.githubusercontent.com/231140/66045425-03fce700-e524-11e9-82ae-8772fa4e9724.png)

5. Verify that the delegated permission *User.Read* for Microsoft Graph is already be selected. This permission is all you need if you do not require mapping Azure AD group membership to WordPress roles.

    ![User.Read delegated permission for Microsoft Graph](https://user-images.githubusercontent.com/231140/66046005-23484400-e525-11e9-9712-fed4c5273040.png)

   > **Note**: If you do wish to map Azure AD groups to WordPress roles, you must also select the delegated permission *Directory.Read.All* (click "Add a permission" > Microsoft Graph > Delegated > *Directory.Read.All*).

   > **Important**: Some permissions *require* administrator consent before it can be used, and in some organizations, administrator consent is required for *any* permission. A tenant administrator can use the **Grant admin consent** option to grant there permissions (i.e. consent) on behalf of all users in the organization.

6. Under **Certificates & secrets**, create a new client secret. Provide a description and choose a duration (I recommend no longer than two years). After clicking **Add**, the secret value will appear. Copy it, as this is the only time it will be available.

    ![Creating a new secret key](https://user-images.githubusercontent.com/231140/66046096-52f74c00-e525-11e9-93ce-62581e097aaa.png)

8. Switch to the **Overview** section and keep the tab open, as you will need to copy some fields when configuring the plugin.

    ![App overview page](https://user-images.githubusercontent.com/231140/66046578-5b9c5200-e526-11e9-810f-027d31d99148.png)

### 3. Configure the plugin

Once the plugin is activated in WordPress (step 1), update your settings from the WordPress admin console under **Settings** > **Azure AD**. Basic settings to include are:

<dl>
  <dt>Display name</dt>
  <dd>
    The display name of the organization, used in the link on the WordPress login page which will start the Azure AD sign-in process.
  </dd>

  <dt>Client ID</dt>
  <dd>
    The Application ID. (Copy this from the Azure AD app registration's **Overview** page.)
  </dd>

  <dt>Client Secret</dt>
  <dd>
    The client secret. (You copy this from the Azure AD app registration's **Certificates & secrets** page.)
  </dd>

  <dt>Reply URL</dt>
  <dd>
    The URL that Azure AD will send the user to after authenticating. This is usually the blog's sign-in page, which is the default value. Ensure that the reply URL configured in Azure AD matches this value.
  </dd>
</dl>

### 4. (Optional) Set WordPress roles based on Azure AD group membership

The Single Sign-on with Azure AD plugin can be configured to set different WordPress roles based on the user's membership to a set of user-defined groups. This is a great way to control who has access to the site, and under what role.

This is also configured **Settings** > **Azure AD** (from the WordPress admin console). The following fields should be included:

<dl>
  <dt>Enable Azure AD group to WP role association</dt>
  <dd>
    Check this to enable Azure AD group-based WordPress roles.
  </dd>

  <dt>Default WordPress role if not in Azure AD group</dt>
  <dd>
    This is the default role that users will be assigned to if matching Azure AD group to WordPress roles is enabled. If this is not set, and the user authenticating does not belong to any of the groups defined, they will be denied access.
  </dd>

  <dt>WordPress role to Azure AD group map</dt>
  <dd>
    For each of the blog's WordPress roles, there is a field for the ObjectId of the Azure AD group that will be associated with that role.
  </dd>
</dl>

> **Note**: For the Azure AD group to WordPress role mapping to work, the app in Azure AD needs the delegated permission *Directory.Read.All* for Microsoft Graph. See step 5 of *Register an Azure Active Directory application*, above, for more details.

## Example settings

The different fields that can be defined in the settings JSON in **Settings** > **Azure AD** are documented in [Settings.php](Settings.php). The following may give you an idea of the typical scenarios that may be encountered.

### Minimal

Users are matched by their email address in WordPress, and whichever role they have in WordPress is maintained.

| Setting | Example value
| --- | ---
| Display name | Contoso
| Client ID | 9054eff5-bfef-4cc5-82fd-8c35534e48f9
| Client Secret | NTY5MmE5YjMwMGY2MWQ0NjU5MzYxNjdjNzE1OGNiZmY=
| Reply URL | https://www.example.com/blog/wp-login.php
| Field to match to UPN | Email Address

### Match on username alias

Users are matched by their login names in WordPress and the alias portion of their Azure AD UserPrincipalName. Whichever role they have in WordPress is maintained.

| Setting | Example value
| --- | ---
| Display name | Contoso
| Client ID | 9054eff5-bfef-4cc5-82fd-8c35534e48f9
| Client Secret | NTY5MmE5YjMwMGY2MWQ0NjU5MzYxNjdjNzE1OGNiZmY=
| Reply URL | https://www.example.com/blog/wp-login.php
| Field to match to UPN | Login Name
| Match on alias of the UPN | Yes

### Group membership-based roles, no default role

Users are matched by their login names in WordPress, and WordPress roles are dictated by membership to a given Azure AD group. Access is denied if they are not members of any of these groups.

| Setting | Example value
| --- | ---
| Display name | Contoso
| Client ID | 9054eff5-bfef-4cc5-82fd-8c35534e48f9
| Client Secret | NTY5MmE5YjMwMGY2MWQ0NjU5MzYxNjdjNzE1OGNiZmY=
| Reply URL | https://www.example.com/blog/wp-login.php
| Field to match to UPN | Login Name
| Enable Azure AD group to WP role association | Yes
| Default WordPress role if not in Azure AD group | (None, deny access)
| WordPress role to Azure AD group map | <table><tr><td>Administrator</td><td>5d1915c4-2373-42ba-9796-7c092fa1dfc6</td></tr><tr><td>Editor</td><td>21c0f87b-4b65-48c1-9231-2f9295ef601c</td></tr><tr><td>Author</td><td>f5784693-11e5-4812-87db-8c6e51a18ffd</td></tr><tr><td>Contributor</td><td>780e055f-7e64-4e34-9ff3-012910b7e5ad</td></tr><tr><td>Subscriber</td><td>f1be9515-0aeb-458a-8c0a-30a03c1afb67</td></tr></table>

### Group membership-based roles with default role

Users are matched by their login names in WordPress, and WordPress roles are dictated by membership to a given Azure AD group. If the user is not a part of any of these groups, they are assigned the *Author* role.

| Setting | Example value
| --- | ---
| Display name | Contoso
| Client ID | 9054eff5-bfef-4cc5-82fd-8c35534e48f9
| Client Secret | NTY5MmE5YjMwMGY2MWQ0NjU5MzYxNjdjNzE1OGNiZmY=
| Reply URL | https://www.example.com/blog/wp-login.php
| Field to match to UPN | Login Name
| Enable Azure AD group to WordPress role association | Yes
| Default WordPress role if not in Azure AD group | Author
| WordPress role to Azure AD group map | <table><tr><td>Administrator</td><td>5d1915c4-2373-42ba-9796-7c092fa1dfc6</td></tr><tr><td>Editor</td><td>21c0f87b-4b65-48c1-9231-2f9295ef601c</td></tr><tr><td>Author</td><td>f5784693-11e5-4812-87db-8c6e51a18ffd</td></tr><tr><td>Contributor</td><td>780e055f-7e64-4e34-9ff3-012910b7e5ad</td></tr><tr><td>Subscriber</td><td>f1be9515-0aeb-458a-8c0a-30a03c1afb67</td></tr></table>

### Group membership-based roles, default role, auto-provision

Users are matched by their email in WordPress, and WordPress roles are dictated by membership to a given Azure AD group. If the user doesn't exist in WordPress yet, they will be auto-provisioned. If the user is not a part of any of these groups, they are assigned the *Subscriber* role.

| Setting | Example value
| --- | ---
| Display name | Contoso
| Client ID | 9054eff5-bfef-4cc5-82fd-8c35534e48f9
| Client Secret | NTY5MmE5YjMwMGY2MWQ0NjU5MzYxNjdjNzE1OGNiZmY=
| Reply URL | https://www.example.com/blog/wp-login.php
| Field to match to UPN | Email Address
| Enable auto-provisioning | Yes
| Enable Azure AD group to WP role association | Yes
| Default WordPress role if not in Azure AD group | Subscriber
| WordPress role to Azure AD group map | <table><tr><td>Administrator</td><td>5d1915c4-2373-42ba-9796-7c092fa1dfc6</td></tr><tr><td>Editor</td><td>21c0f87b-4b65-48c1-9231-2f9295ef601c</td></tr><tr><td>Author</td><td>f5784693-11e5-4812-87db-8c6e51a18ffd</td></tr><tr><td>Contributor</td><td>780e055f-7e64-4e34-9ff3-012910b7e5ad</td></tr><tr><td>Subscriber</td><td>f1be9515-0aeb-458a-8c0a-30a03c1afb67</td></tr></table>

## Groups

As described above, you can map Azure AD groups to WordPress roles. Users who are members of the Azure AD group will be granted the WordPress role(s) the groups were mapped to.

There are several ways Azure AD groups can be created/managed. Some of them require the group owner/creator to be a tenant administrator, others not necessarily (depending on your organization's policy):

 * **Azure portal**. The Azure portal ([https://portal.azure.com](https://portal.azure.com)), under [Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) > [Groups](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups) > New group, allows admins and (optionally) users to create and manage groups.
 * **Access Panel**. The Azure AD Access Panel ([https://myapps.microsoft.com](https://myapps.microsoft.com)) provides an interface for users to create and manage [groups](https://account.activedirectory.windowsazure.com/#/groups).
 * **Outlook**. The Outlook web interface ([https://outlook.office.com/](https://outlook.office.com/)) offers users the option to create Office 365 Groups. These groups are stored in Azure AD and can be used with this plugin.
 * **Microsoft Teams**. Creating a team in Microsoft Teams ([https://teams.microsoft.com](https://teams.microsoft.com)) also results in an Office 365 Group getting created.
 * **Azure AD PowerShell**. The [Azure AD PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/active-directory/install-adv2?view=azureadps-2.0) allows admins and (optionally) users to create and manage groups. (e.g. [New-AzureADGroup](https://docs.microsoft.com/en-us/powershell/module/azuread/new-azureadgroup?view=azureadps-2.0), and [Add-AzureADGroupMember](https://docs.microsoft.com/en-us/powershell/module/azuread/add-azureadgroupmember?view=azureadps-2.0) cmdlets.)
 * **On-premises**. Many large organizations use Azure AD Connect to sync their on-premises AD to Azure AD. This usually includes all on-premises AD groups and memberships. Once these groups are synced to Azrue AD, they can be used with this plugin.

## Advanced

### Refreshing the OpenID Connect configuration cache

Most of the OpenID Connect endpoints and configuration (e.g. signing keys, etc.) are obtained from the OpenID Connect configuration endpoint. These values are cached for one hour, but can always be forced to re-load by adding `aadsso_reload_openid_config=1` to the query string in the login page. (This shouldn't really be needed, but it has shown to be useful during development.)

### Bypassing automatic redirect to Azure AD to prevent lockouts

If you've configured this plugin to automatically redirect to Azure AD for sign-in, but something is misconfigured, you may find yourself locked out of your site's admin dashboard.

To log in to your site *without* automatically redirecting to Azure AD (thus giving you an opportunity to enter a regular username and password), you can append `?aadsso_no_redirect=please` to the login URL. For example, if your login URL is `https://example.com/wp-login.php`, navigating to `https://example.com/wp-login.php?aadsso_no_redirect=please` will prevent any automatic redirects.
