# Meta / Instagram Connector Setup

The Instagram connector now expects a Meta app that uses the **Manage messaging and content on Instagram** use case with the **API setup with Facebook login** flow. Follow these steps before running the stack locally:

1. **Create the Instagram app**
   - In [developers.facebook.com](https://developers.facebook.com/apps) click **Create App** and choose the *Business* type.
   - Select **Manage messaging and content on Instagram** as the use case (this automatically adds the Instagram Graph API product).
   - Complete the wizard and note the App ID and App Secret from **App settings → Basic**.

2. **Enable Facebook Login for the use case**
   - Open **Use cases → Instagram API → API setup with Facebook login** and click **Set up**.
   - Add the required permissions: `business_management`, `instagram_basic`, `instagram_content_publish`, `pages_manage_posts`, `pages_read_engagement`, `pages_show_list`.
   - Under **Set up Instagram business login** add the same redirect URI used by the SPA (`http://localhost:3000/meta/callback` for local dev). Also configure the deauthorization and data deletion callbacks.

3. **Link assets in Business Manager**
   - Convert the Instagram account you plan to test with into a Professional (Business or Creator) account.
   - Create (or reuse) a Facebook Page owned by the same Business Manager and link the Instagram account to that page in Meta Business Suite (**Page Settings → Linked accounts**).
   - In **Business Settings → Accounts → Instagram accounts**, assign the Instagram account to the page under **Connected assets** and grant yourself full control.

4. **Assign tester roles**
   - In the new app, go to **App roles → Instagram Testers** and invite your Instagram handle.
   - Accept the invite from the Instagram mobile app (**Settings → Account Center → Apps and websites → Tester invites**).

5. **Configure environment variables**
   - Update `META_CLIENT_ID`, `META_CLIENT_SECRET`, and `META_REDIRECT_URI` in `application-local.properties` (or via `setx`) with the new app credentials.
   - Restart the Spring API so the new configuration takes effect.

6. **Verify with Graph API Explorer (optional)**
   - Use the Graph API Explorer to generate a user token with the permissions above and run `GET /me/accounts?fields=id,name,instagram_business_account{id,username,name}`. You should see every Facebook Page and its linked Instagram account.

With this setup every developer can complete the Meta OAuth flow locally, and the UI will show all eligible pages/Instagram accounts when multiple assets are available.

