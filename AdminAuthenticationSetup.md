Each library that wants to use the admin interface needs to authenticate admins. 

Currently, the circulation manager can only use Google OAuth to authenticate admins, but more options will be added in the future.

# Setting up Google OAuth
- Go to [the Google Developer Console](https://console.developers.google.com/apis/dashboard)
- Create a project
- Click "Credentials" in the left sidebar
- Click "Create Credentials" and select "OAuth client ID"
- If you get a warning about the consent screen, click "Configure consent screen" and enter your library name as the product name. Save the consent screen information.
- Back on the Create client ID screen, select "Web application".
- Leave "Authorized JavaScript origins" blank, but under "Authorized redirect URIs", add the url of your circulation manager followed by "/admin/GoogleAuth/callback", e.g. "http://mycircmanager.org/admin/GoogleAuth/callback"
- Click create, and you'll get a popup with your new client ID and secret. Copy these values.
- Visit the admin interface at your circulation manager url followed by "/admin". If you haven't configured admin authentication yet, you'll get a form where you can set up admin authentication. If you've already set up admin authentication but want to change it, click "Configuration" in the header, then click the "Admin Authentication Services" tab and choose the service to edit. 
- Select "Google OAuth" as the provider and enter your client ID and secret.
- Add the domain you use for google under allowed domains. Only google accounts with email addresses that match one of these domains will be able to login.
- Click the save button, and Google OAuth will be enabled. To use the admin interface after this, you'll have to log in with a Google account from one of the allowed domains. 
