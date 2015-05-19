# Heroku Java Remote Debug Buildpack

This is a [Heroku Buildpack](https://devcenter.heroku.com/articles/buildpacks)
for running a debug port on a Java process in [dyno](https://devcenter.heroku.com/articles/dynos).
The debug port will be proxied by [ngrok](https://ngrok.com/), which makes it accessible from a remote machine.

## Setup

First, create a free [ngrok account](https://dashboard.ngrok.com/user/signup). This is necessary to use TCP with their service. Then capture your API key, and set it as a config var on your Heroku app like this:

```
$ heroku config:set NGROK_API_TOKEN=xxxxxx
```

Next, add this buildpack to your app:

```
$ heroku buildpacks:add https://github.com/jkutner/heroku-buildpack-jdwp.git
```

Then add your primary buildpack. For example, if you are using Java:

```
$ heroku buildpacks:add https://github.com/heroku/heroku-buildpack-java.git
```

Now modify your `Procfile` by prefixing your `web` process with the `with_jdwp` command. For example:

```
web: with_jdwp java $JAVA_OPTS -cp target/classes:target/dependency/* Main
```

Finally, commit your changes, and redeploy the app:

```
$ git add Procfile
$ git commit -m "Added with_jdwp"
$ git push heroku master
```

## Usage

Once your app is running with the JDWP buildpack and the `with_jdwp` command, you'll see something like
this in your logs:

```
2015-05-19T16:06:36.530988+00:00 app[web.1]: Listening for transport dt_socket at address: 8998
...
2015-05-19T16:06:37.052977+00:00 app[web.1]: [05/19/15 16:06:37] [INFO] [client] Tunnel established at tcp://ngrok.com:39678
```

Then, from your local machine, you can connect to the process using the ngrok URL in the logs. For example:

```sh-session
$ jdb -attach ngrok.com:39678
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
Initializing jdb ...
>
```

Then you can use it and stuff:

```sh-session
> methods my.company.MainServlet
...
javax.servlet.Servlet service(javax.servlet.ServletRequest, javax.servlet.ServletResponse)
javax.servlet.Servlet getServletInfo()
javax.servlet.Servlet destroy()
javax.servlet.ServletConfig getServletName()
javax.servlet.ServletConfig getServletContext()
javax.servlet.ServletConfig getInitParameter(java.lang.String)
javax.servlet.ServletConfig getInitParameterNames()
```

## Customizing

You can customize the execution of ngrok by setting the `NGROK_OPTS` config var like so:

```
$ heroku config:set NGROK_OPTS="-subdomain=my-custom-name"
```

You can also add a `.ngrok` to your app.

You can customize the JDWP settings with the `JDWP_OPTS` config var. For example:

```
$ heroku config:set JDWP_OPTS="server=y,suspend=y"
```
