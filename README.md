Deploying to Heroku can be tricky. Here are some tips to get your deployed application working!
From [DEV](https://dev.to/darnocer/heroku-deployment-tips-3lii)

---

## Table of Contents

- [Deployment](#Deployment)
- [OAuth](#OAuth)
- [JawsDB for MySQL](#JawsDB-for-MySQL)
- [MongoDB Atlas for MongoDB](#MongoDB-Atlas-for-MongoDB)

---

## Deployment
_Creating a Heroku app using the GUI_

1. Navigate to your Heroku account
2. Click New > Create new app and enter a name for the app
3. Under Deploy > Deployment Method > Select Github
4. Enter the Github repo name and connect
5. Enable Automatic deploys to automatically update Heroku with updates to master (otherwise need to use `git push heroku master`)
6. Navigate to your repo in the terminal
7. View remotes, there should be no heroku remotes yet, just origin from github: `git remote -v`
8. Create a remote for your app: `heroku git:remote -a [heroku app name]`
9. Verify you see a heroku remote; `git remote -v`

## OAuth

1. Add the `clientID` and `clientSecret` as keys with the corresponding values in Config Vars on Heroku
2. In the OAuth app, add the Heroku URL as the Homepage and Callback URL
3. Ensure the passport routes to `/` as the homepage. Redirect to the authentication path if necessary

```
app.get("/", passport.authenticate("github"), function (req, res) {
  res.redirect("/auth/github");
});
```

4. Setup for `clientID` and `clientSecret`:

```
    {
      clientID: process.env.clientID,
      clientSecret: process.env.clientSecret,
      callbackURL: "/auth/github/callback",
    },
```

### Options for testing locally with OAuth

_After changes made to deploy to Heroku_

- In server.js, update `clientID` and `clientSecret` to be hardcoded string values; return to `process.env.clientID` and `process.env.clientSecret` prior to pushing changes

- Create a new OAuth app and configure for `localhost`. Setup the `clientID` and `clientSecret` to use `process.env` on Heroku, but the localhost configured app otherwise:

```
clientID: process.env.clientID || [new_locahost_clientID_here],
clientSecret: process.env.clientSecret || [new_localhost_clientSecret_here],
```

- Create a `.env` file, add `clientID="{clientID}"` and `clientSecret="{clientSecret}"`. Be sure to add `.env` to `.gitignore`. `npm i dotenv`and follow `dotenv` setup instructions

## JawsDB for MySQL

1. From the Overview tab on the Heroku app, select "Configure Add-ons"
2. Search for JawsDB MySQL and attach the add-on
3. Click the JawsDB link for connection info
4. Go to MySQL Workbench to setup a new connection
5. Enter the host, username, and password info and test the connection (ensure you don't accidentally paste extra spaces)
6. Open the connection and if it was successful, you should see a sole database that is a string of gibberish (don't rename)
7. No need to create tables as that will created via Sequelize
8. In Heroku, go to Settings > Config Vars > Reveal Config Vars
9. Verify there is a config var for JAWSDB_URL with the URL value
10. In `config.json`, configure the "production" property as follows:

```
"production": {
    "dialect": "mysql",
    "use_env_variable": "JAWSDB_URL"
  }
```

## mLab for MongoDB / Mongoose 
_mLab is now depreciated. Use MongoDB Atlas (below) instead.  

1. Run this command to provision mLab to the app:
   `heroku addons:create mongolab`
2. Navigate to Heroku app and verify the mLab add-on is provisioned
3. Navigate to Settings > Config vars and verify there is a `MONGODB_URL` config variable
4. Ensure `mongoose` is required in `server.js`
5. Update Mongoose connection in `server.js` to be the following:

```
var MONGODB_URI = process.env.MONGODB_URI || "mongodb://localhost/db_name";
const options = {
  useNewUrlParser: true,
  useCreateIndex: true,
  useFindAndModify: false,
};

mongoose.connect(MONGODB_URI,options)
```

## MongoDB Atlas for MongoDB
### MongoDB Atlas Account
1. Create your [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/signup) account
2. Create a new "cluster"
3. Select the free tier
4. Can use any provider - such as AWS
5. Choose the server in the region in the free tier closest to you
6. After creating the cluster, in the cluster, go to Security > Database Access
7. +Add new database user
8. Add username/password 
9. Change privileges to Atlas Admin
10. Go to Security > Network Access
11. Go to Add IP Address and select Allow Access from Anywhere
12. Go to Clusters > Collections > Add My Own Data
13. Add your database name and one collection name (only need to add one to get started even if you have multiple collections since they will be created by Mongoose)

### Connecting MongoDB Atlas to Heroku
1. Click the Connect button in your cluster
2. Select Connect Your Application and choose Heorku
3. Copy the string
4. Go to your app in Heroku > Settings > Config Vars
5. Enter MONGODB_URI in the key
6. For the value, paste the string you copied from MongoDB Atlas
7. Update the username/password and database name in the string to match what you created in the MongoDB Atlas Setup
8. Ensure you look to MONGODB_URI in you Mongoose connection (likely in server.js)

```
mongoose.connect(
  process.env.MONGODB_URI || "mongodb://localhost/databasename",
  {
    useUnifiedTopology: true,
    useNewUrlParser: true,
    useCreateIndex: true,
    useFindAndModify: false,
  }
);
```
