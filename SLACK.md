Getting the Loomio slack app to work on your self- hosted instance (a step-by-step guide!)

- Visit 'https://api.slack.com/apps'
- Click `Create new App`
- Fill out your app name, pick anything as the dev workspace (we won't be installing this to a dev environment)
- Click `add features and functionality`. We want to add 3 pieces of functionality
  - Interactive messages
    - enter `https://loomio.example.com/slack/participate` as the Request URL
  - Slash commands
    - enter `/loomio` as the command name
    - `https://loomio.example.com/slack/initiate` as the Request URL
    - `Make a decision with Loomio` (or translated version) for short description
    - `[poll_type] [title]` as the Usage Hint
  - OAuth & Permissions
    - Add a new redirect url, of your domain (eg `https://loomio.example.org`)
    - Scopes should not be necessary, since we request the appropirate permissions from Loomio
- Display information
  - App name: `Loomio Bot <instance name>`
  - Short Description: `Make decisions together`
  - App icon: https://s3-us-west-2.amazonaws.com/slack-files2/avatars/2017-04-28/177035264918_e3d06443671502b0ad02_512.png
  - Background color: `#cd6600`
- Now, click `Distribute App`, then `Activate public distribution`

- Take note of the generated Client ID, Client Secret and Verification Token. You should now be ready to apply these ENV values to your Loomio instance:
```
SLACK_APP_KEY=<Client ID>
SLACK_APP_SECRET=<Client Secret>
SLACK_VERIFICATION_TOKEN=<Verification token>
```
Then, you should be good to go!
