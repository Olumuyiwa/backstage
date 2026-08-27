Step 1:  For Backstage GitHub OAuth login, you should instead create an OAuth App, not a GitHub App.
Go to:
https://github.com/settings/developers
Then:
OAuth Apps
New OAuth App
Or go directly to:
https://github.com/settings/applications/new
On that page you'll see only these fields:
Application name
Homepage URL
Application description (optional)
Authorization callback URL
There is no Webhook URL.

Note: When would you use a GitHub App?
A GitHub App is used when Backstage needs to:
Discover repositories
Receive push events
Listen for pull requests
Integrate with GitHub APIs using app authentication
In that case, a webhook is required because GitHub sends events to your application.
For signing into Backstage
If your configuration looks like this:
auth:
  providers:
    github:
      production:
        clientId: ${GITHUB_OAUTH_CLIENT_ID}
        clientSecret: ${GITHUB_OAUTH_CLIENT_SECRET}

then you need an OAuth App, not a GitHub App.

One question
Are you only trying to let users log in with GitHub, or are you also trying to configure the GitHub integration so Backstage can read repositories, catalog files, and scaffolder templates?
The answer determines whether you need just an OAuth App, or both an OAuth App and a GitHub App.

