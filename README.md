# wwwhisper Cloud Native Buildpack for Heroku Fir stack

Provides application-independent access control for Heroku-hosted web
applications. The access control is based on
verified email addresses of visitors.

The Cloud Native Buildpack works with Heroku new generation Fir stack
available in Fir Private Spaces. A buildpack for Heroku Common
Runtime and Cedar Privates Spaces is at:
https://github.com/wwwhisper-auth/wwwhisper-heroku-buildpack

## Enabling the Buildpack

The buildpack requires the Heroku [wwwhisper
add-on](https://elements.heroku.com/addons/wwwhisper).

1. Subscribe to the wwwhisper add-on. In your application folder run:

   ```
   heroku addons:create wwwhisper:team-3 [--admin=your_email]
   ```

   `team-3` is the name of the plan to enable. Run `heroku addons:plans
   wwwhisper` to see all available plans and prices.

2. Add the following section to your application's `project.toml` file
   to enable the buildpack:

   ```
   [[io.buildpacks.group]]
   id = "wwwhisper/wwwhisper-cnb"
   ```

   See sample [project.toml](./sample-configs/project.toml) for a
   reference.

3. Modify your `Procfile` to start wwwhisper authorization proxy in
front of your web app process. Modify the `web:` entry to call
`wwwhisper-auth` with a single argument that contains your usual
application launch command. The command should be enclosed in single
quotes ''. For example, a `Procfile` that starts Python fastapi server
looks like this:

   ```
   web: wwwhisper-auth 'fastapi run --port $PORT --host ::'
   ```

   See sample [Procfile](./sample-configs/Procfile) for a reference.

4. Commit the changes:

   ```
   git add project.toml Procfile;
   git commit -m "Enable wwwhisper buildpack";
   git push heroku main # or master;
   ```

After these operations, opening your application URL will show a login
prompt. Enter your Heroku application owner email to receive a login
link.

## Technical Details

The buildpack runs a reverse proxy that authenticates and authorizes
visitors. Authorized requests are passed to the app; unauthorized ones
are rejected with 401 or 403 HTTP errors. Sessions and access control
rules used by the proxy are stored by the wwwhisper backend. For
efficiency, the proxy caches this data allowing most authorization
decisions to be made locally in sub-millisecond time, without
requiring requests to the wwwhisper backend.

The reverse proxy listens on an externally accessible `PORT`
configured by the Heroku dyno manager. The `PORT` environment variable
passed to the application is reassigned to a private port that is not
externally accessible.

