﻿# For Maintainers
## Overview
The meeting-registration-form repo is an application running on the free platform Heroku which handles/includes web forms, a PostgreSQL database, a Python program, and some ancillary files needed to describe the app to Heroku.

### Web pages
The main one is `index.html`, which is the form people use to register. `success.html` is displayed after a successful registration. `participants.html` describes the table used to display minimal participant information for public viewing: first name, last name and affiliation.

### Python program
`registration_server.py` describes the database structure (a single table named participants), creates the table when invoked with `--create`  (see `app.json` for such an invocation) and defines some other functions not used within the repo.

### Dependencies
Dependencies are described in `Pipfile` in a general way; `Pipfile.lock` contains detailed information about exactly which versions of everything were used.  Under normal circumstances, the ones in the repo can be used as is, with no changes. In the unlikely event you do have to change something, see the section **Updating Dependencies** below.


## Updating the meeting-registration-form Repo

### Use a fork
Since it’s very unlikely you’ll get everything right the first time (or even that you’ll know exactly what you want initially), fork the production repository, e.g. under your personal github space, and work on the fork until everything is ready for production.

In order to be able to test everything thoroughly on your fork, you should set up github.io:
1. If you don't already have it, create a repository in your space named your-github-name.github.io. My github name is JoanneBogart so the full path to mine is `JoanneBogart/JoanneBogart.github.io`. This repo needs a README.md file but nothing else.
2. In the github web interface
* go to your fork of meeting-registration-form
* click **Settings** (along the top)
* click **Pages** (in the list to the left, under **Options**)
* the drop-down menu under **Source** should say **None**. Choose the branch (either master or main) which will be the source. And you’re done.

### First deploy
You can pretty much follow the instructions in README.md, but I suggest first creating your Heroku account, then starting over from the README page to do the actual deploy (I’m not positive but I think when I tried to continue immediately after creating my account the page I saw didn’t match the description in the README. But the next time around, with my account already created, it did.)

If the deploy is successful, click **Manage app**, then **Settings**, and finally click **Reveal Config Vars**. There are two: DATABASE_URL and SECRET_KEY. Copy and save the values for both. DATABASE_URL at least should be saved in a protected place (no read access for anyone else). These values don’t change if you need to redeploy the application. They of course do change if you delete the application and start over with the same name.

### Useful URLs
Let SERVER_URL be
`https://APPNAME.herokuapp.com` where APPNAME is the value you typed in the App name field. Then the URL people will use to register*, as stated in README.md, is

`https://lsstdesc.github.io/meeting-registration-form/index.html?backend=SERVER_URL&secret=SECRET_KEY`

where SECRET_KEY is the value of the SECRET_KEY config variable.

To see a summary of registered participants to date, use SERVER_URL

**NOTE:** For your fork the registration URL will be a little different. `lsstdesc.github.io` will instead be `your-github-user.github.io`.   If your forked repo is not named meeting-registration-form, that part of the URL will also have to change.

### Updated deploy instructions (August 2024)
There have been enough changes in the way Heroku operates that much of the automation the README used to
provide is broken. As one of the admins of the (paying) account SLAC has with Heroku I did the following
to get everything up and running:

* logged into Heroku site
* created an app
* created a variable (from **Settings** page) SECRET_KEY and gave it a value (doesn't matter too much what but,
  since it will be a parameter in a url, stay away from characters like & or ? which could cause trouble).
* added the add-on "Heroku Postgres" (from **Resources** page)
* go to **Deploy** page
* chose GitHub deploy method
* connect to GitHub repo where your source is, e.g. YourName/meeting-registration-form or
  LSSTDESC/meeting-registration-form. NOTE: you must have sufficient permission (e.g. admin; write permission
  is not enough) in the repo for this to succeed.
* select branch to deploy from and push the **Deploy Branch** button.  I've always used Manual deploy
  rather than automatic.
* when deploy is complete and successful, go to **More** button in the upper right and select "console".
* from the console issue the command

    python registration_server.py --create

* it's no longer the case that the server url is just https://YOUR_APP_NAME.herokuapp.com. There are a
  bunch of hex digits following the app name, like this example:

   https://desc-oct2024-meeting-4fc146491d4c.herokuapp.com/
* If you're deploying from a branch other than the standard (master or main), you need to tell GitHub pages
  to deploy from that branch. From the GitHub repo home page go to "Settings" and click on "Pages" in the column
  at the left.  Under Build and deployment/Branch select the desired branch.



### Querying the database
Connect to the database using DATABASE_URL.   It’s in this format:

    `postgres://user:password@host:port/dbname`

If you use sqlalchemy to connect, depending on version you might need to change the initial `postgres` to `postgresql`.   Then make a file, e.g. `database.ini`, consisting just of that string. Change protection on the file so only you can read it. See the Jupyter notebook DatabaseQueries in this repo for an example.

Alternatively, you can put the database access credentials in a `.pgpass` file in your home directory with the following format
```
hostname:port:database:username:password
```
Set the permissions of the `.pgpass` file with `chmod 600 ~/.pgpass`, and you can then use client programs such as `pgsql` or `pgcli` to access the tables.

### Redeploy
1. Install the Heroku CLI.   There are various ways to do this.  For my (mac) laptop I downloaded the tarball, unpacked, and set my path to include the bin directory so that the heroku command could be found.

2. Do
    `$ heroku login`
This will open a browser window so you can log in to your Heroku account.  You may be asked to set up 2-factor authentication.

3. There are many useful commands.  See the [doc](https://devcenter.heroku.com/articles/heroku-cli-commands)

For example, list all your apps with
  `$ heroku apps`

To find out more about a particular app called “my-app-name”
  `$ heroku apps:info -a my-app-name`

4. If there isn’t already a copy of the git repo on the machine where the CLI is installed, clone one.

5. From within your clone, do
  `$ heroku git:remote -a my-app-name`
Your repo now has a new remote named heroku.

6. To redeploy at any time, just do
`$ git push heroku master`
(or push main if that is the production branch of the repo.) You can only push _to_ master or (main), but the source can be a different branch. For example, if you want to push a development branch, say my-dev-branch, for testing, do
`$ git push heroku my-dev-branch:master`

Note that redeploying will not change SERVER_URL, SECRET_KEY, or DATABASE_URL. It also will not touch the database. If the database structure has changed either
* do not redeploy; instead delete the app and start over. This is fine if you’re not yet in production.
* fix the database by hand somehow. There are various Heroku commands starting with the string `heroku pg:` which might be useful. For example you can create backups, restore them, and open a psql session.

#### Redeploying from the Heroku web dashboard
One can also redeploy the app from the Heroku dashboard if the app is connected to the meeting-registration-form repository on GitHub.  To make that connection, go to the "Deploy" tab for the app instance and select the "GitHub" deployment method.  Once the app is connected to the GitHub repository, you will see options to enable automatic deployment based on pushes to the GitHub repo or to trigger deployments from specific branches.  Note that with the GitHub connection enabled, the Heroku app will appear as an active "Environment" in the GitHub repository, so you may wish to disconnect that app from the GitHub repository after the meeting is over.

### Updating Dependencies
If you have to change the version of something, manually change it only in `Pipfile`. It’s possible to deploy an app with just `Pipfile`; Heroku will generate a suitable `Pipfile.lock`, but for predictability this is not normally the way one should operate.  By generating `Pipfile.lock` yourself you guarantee that the same precise versions are used every time, until you want to change them.   `Pipfile.lock` can be generated by the utility `pipenv` (which, I must admit, I couldn’t manage to use. Instead I copied a `Pipfile.lock` generated by someone else who could.)
