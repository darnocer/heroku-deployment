# Heroku Deployment Tips

## Creating a Heroku app using the GUI

- Navigate to your Heroku account
- Click New > Create new app and enter a name for the app
- Under Deploy > Deployment Method > Select Github
- Enter the Github repo name and connect
- Enable Automatic deploys to update Heroku with updates to master (no need for `git push heroku master`)
- Navigate to your repo in the terminal
- View remotes, there should be no heroku remotes yet, just origin from github

  `git remote -v`

- Create a remote for your app

  `heroku git:remote -a [app name]`

- Verify you see a heroku remote

  `git remove -v`

## JawsDB Setup (with Sequelize)

- From the Overview tab on the Heroku app, select "Configure Add-ons"
- Search for JawsDB MySQL and attach the add-on
- Click the JawsDB link for connection info
- Go to MySQL Workbench to setup a new connection
- Enter the host, username, and password info and test the connection (ensure you don't accidentally paste extra spaces)
- Open the connection and if it was successful, you should see a sole database that is a string of gibberish (don't rename)
- No need to create tables as that will created via Sequelize
- In Heroku, go to Settings > Config Vars > Reveal Config Vars
- Verify there is a config var for JAWSDB_URL with the URL value
- In `config.json`, configure the "production" property as follows:

```
"production": {
    "dialect": "mysql",
    "use_env_variable": "JAWSDB_URL"
  }
```

## OAuth Setup

- Add the `clientID` and `clientSecret` as keys with the corresponding values in Config Vars on Heroku
- In the OAuth app, add the Heroku URL as the Homepage and Callback URL
- Ensure the passport routes to "/" as the homepage. Redirect to the authentication path if necessary

```
app.get("/", passport.authenticate("github"), function (req, res) {
  res.redirect("/auth/github");
});
```

- Setup for `clientID` and `clientSecret` in `server.js`:

```
    {
      clientID: process.env.clientID,
      clientSecret: process.env.clientSecret,
      callbackURL: "/auth/github/callback",
    },
```

## Options for testing locally

_After changes made to deploy to Heroku_

1. In server.js, update `clientID` and `clientSecret` to be hardcoded string values; return to `process.env.clientID` and `process.env.clientSecret` prior to pushing changes

2. Create a new OAuth app and configure for `localhost`. Setup the `clientID` and `clientSecret` to use `process.env` on Heroku, but the localhost configured app otherwise:

```
clientID: process.env.clientID || [new_locahost_clientID_here],
clientSecret: process.env.clientSecret || [new_localhost_clientSecret_here],
```

3. Create a `.env` file, add `clientID="{clientID}"` and `clientSecret="{clientSecret}"`. Be sure to add `.env` to `.gitignore`. `npm i dotenv`and follow `dotenv` setup instructions
