The binary deployment process will save us downloading those dependencies, however to be really quick we need to short cut he S2I process totally. 

For that we can use the ``oc rsync`` command to set up live synchronization of files between your local computer and the container. When there is a change to a file, the changed file will be automatically copied up to the container.

This same process can also be run in the opposite direction if required, with changes made in the
container being automatically copied back to your local computer.

An example of where it can be useful to have changes automatically copied from your local computer
into the container is during the development of an application.

For scripted programming languages such as JavaScript, PHP, Python or Ruby, where no separate compilation
phase is required you can perform live code development with your application running inside of OpenShift.

For JBoss EAP applications you can sync individual files (such as HTML/CSS/JS files), or sync entire application
.WAR files. It's more challenging to synchronize individual files as it requires that you use an *exploded*
archive deployment, so the use of [JBoss Developer Studio](https://developers.redhat.com/products/devstudio/overview/) is
recommended, which automates this process (see [these docs](https://tools.jboss.org/features/livereload.html) for more info).

## Live synchronization of project files

For this workshop, we'll Live synchronize the entire WAR file.

First, click on the [Coolstore application link](http://www-coolstore-bin.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com)
to open the application in a browser tab so you can watch changes.

**1. Turn on Live Sync**

First we need to save the name of the pod into an environment variable called `COOLSTORE_DEV_POD_NAME` so that we can use it for future
commands:

`export COOLSTORE_DEV_POD_NAME=$(oc get pods --selector deploymentconfig=coolstore -o jsonpath='{.items[?(@.status.phase=="Running")].metadata.name}')`{{execute T2}}

Verify the variable holds the name of your pod with:

`echo $COOLSTORE_DEV_POD_NAME`{{execute T2}}

Turn on **Live sync** by executing this command:

`oc --server https://master:8443 rsync ${HOME}/projects/monolith/deployments/ $COOLSTORE_DEV_POD_NAME:/deployments --watch --no-perms `{{execute T2}}

Now `oc` is watching the `deployments/` directory for changes to the `ROOT.war` file. Anytime that file changes,
`oc` will copy it into the running container and we should see the changes immediately (or after a few seconds). This is
much faster than waiting for a full re-build and re-deploy of the container image.

**2. Make a change to the UI**

Next, let's make a change to the app that will be obvious in the UI.

First, open `src/main/webapp/app/css/coolstore.css`{{open}}, which contains the CSS stylesheet for the
CoolStore app.

Add the following CSS to turn the header bar background to Red Hat red (click **Copy To Editor** to automatically add it):

<pre class="file" data-filename="src/main/webapp/app/css/coolstore.css" data-target="append">

.navbar-header {
    background: #CC0000
}

</pre>

**2. Rebuild application For RED background**

Let's re-build the application using this command:

`mvn package -Popenshift`{{execute T1}}

This will update the ROOT.war file and cause the application to change.

Re-visit the app by reloading the Coolstore webpage (or clicking again on the [Coolstore application link](http://www-coolstore-bin.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com)).

You should now see the red header:

> **NOTE** If you don't see the red header, you may need to do a full reload of the webpage.
On Windows/Linux press `CTRL`+`F5` or hold down `CTRL` and press the Reload button, or try
`CTRL`+`SHIFT`+`F5`. On Mac OS X, press `SHIFT`+`CMD`+`R`, or hold `SHIFT` while pressing the
Reload button.

![Red](/redhat-middleware-workshops/assets/developer-intro/nav-red.png)

**3. Rebuild again for BLUE background**

Repeat the process, but replace the background color to be blue (click **Copy to Editor** to do this automatically):

<pre class="file" data-filename="src/main/webapp/app/css/coolstore.css" data-target="insert" data-marker="background: #CC0000">
background: blue
</pre>

Again, re-build the app:

`mvn package -Popenshift`{{execute T1}}

This will update the ROOT.war file again and cause the application to change.

Re-visit the app by reloading the Coolstore webpage (or clicking again on the [Coolstore application link](http://www-coolstore-bin.[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com)).

![Blue](/redhat-middleware-workshops/assets/developer-intro/nav-blue.png)

It's blue! You can do this as many times as you wish, which is great for speedy development and testing.

We'll leave the blue header for the moment, but will change it back to the original color soon.

Because we used `oc rsync` to quickly re-deploy changes to the running pod, the changes will be lost if we restart the pod. Let's update the container image
to contain our new blue header. Execute:

`oc start-build coolstore --from-file=deployments/ROOT.war`{{execute T1}}

And again, wait for it to complete by executing:

`oc rollout status -w dc/coolstore`{{execute T1}}

## Before continuing

Kill the `oc rsync` processes we started earlier in in terminal 2. Execute:

`clear`{{execute T2 interrupt}}

On to the next challenge!







