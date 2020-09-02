---
layout: default
title: Developer Guide
---
# 1 Prerequisites
You should have read the [User Guide](user-guide.md) and the [Admin Guide](admin-guide.md) so you are aware of
the concepts.

To develop Overbård locally you need to set up your development environment. You will need:

* the Atlassian SDK to build the plugin, and also to debug the Java server side classes
* Yarn to download the Javascript libraries, and other tooling for working on the UI in isolation.

This initial setup of your development environment will take about 10 minutes.

Since Angular 2 is used for the display logic, it is worth looking at the quickstart at
[https://angular.io](https://angular.io).

## 1.1 Atlassian SDK

The Atlassian SDK provides the APIs used for developing the plugin. It has tools for packaging the plugin to be
deployed in the server, and also provides a development Jira instance where you can run and debug the plugin in a
local Jira instance.

Download the Atlassian SDK and install it as outlined on the
[Atlassian site](https://developer.atlassian.com/docs/getting-started/set-up-the-atlassian-plugin-sdk-and-build-a-project).

Alternatively, you can use the tgz based version from on the
[Atlassian Marketplace](https://marketplace.atlassian.com/download/plugins/atlassian-plugin-sdk-tgz)
if you hit a problem on your platform. If you do this, remember to adjust your `PATH` environment variable to include
`${ATLASSIAN_SDK}/bin`. You can check proper setup by invoking the `atlas-version` command. Optionally you can modify
.m2/settings.xml to include the local repository `${ATLASSIAN_SDK}/repository` and the online repository
`https://maven.atlassian.com/content/groups/public`.

## 1.2 Yarn
To install Yarn, follow the installation instructions on the [Yarn site](https://yarnpkg.com).

# 2 First time setup

Fork the [https://github.com/overbaard/overbaard](https://github.com/overbaard/overbaard)
repository to your own GitHub account, and then clone the repository. For this guide we will use the environment
variable `$OB_DIR` to denote the location of the Git clone. `cd` into the `$OB_DIR` and first build it (this assumes Jira8 is the target, we will discuss other Jira versions later):
```
$atlas-mvn install
Executing: /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/bin/mvn  -gs /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/conf/settings.xml install -Dob.jira8
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
[INFO] Scanning for projects...
...
[INFO] Overbård Plugin for Jira 8.x 1.0.0.Beta6-SNAPSHOT .. SUCCESS [  6.493 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 19.587 s
[INFO] Finished at: 2019-06-23T16:53:36+02:00
[INFO] ------------------------------------------------------------------------
```
Next start the Jira 8 server in debug mode:
```
$atlas-debug -pl jira/plugin/jira8 -Dob.jira8
Executing: /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/bin/mvn com.atlassian.maven.plugins:amps-dispatcher-maven-plugin:8.0.0:debug -gs /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/conf/settings.xml -pl jira/plugin/jira8 -Dob.jira8
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
[INFO] Scanning for projects...
[INFO]
[INFO] ---------------< org.overbaard:overbaard-jira-plugin-8 >----------------
[INFO] Building Overbård Plugin for Jira 8.x 1.0.0.Beta6-SNAPSHOT
[INFO] --------------------------[ atlassian-plugin ]--------------------------
```
You will now see a lot of information scroll by while your local Jira instance is started for the first time. If you pay attention, you might see that the Overbård plugin fails to start. This is to expected and will be dealt with shortly. Jira has successfully started once you see this in the log:
```
[INFO] [talledLocalContainer] INFO: Server startup in 59695 ms
[INFO] [talledLocalContainer] Tomcat 8.x started on port [2990]
[INFO] jira started successfully in 88s at http://localhost:2990/jira
[INFO] Type Ctrl-C to shutdown gracefully
```

Open your favourite browser and go to [http://localhost:2990/jira](http://localhost:2990/jira). You now
need to set up Jira. The order of the steps to do this seem to change somewhat as time goes by but the
concepts should remain the same.

First follow the steps to get Jira initialised. Use `admin` as the username, and `admin` as the password and click
through the stuff you need to to do. If you are forced to create a project, give it a name like `Test`. The type of
the `Test` project does not matter as we will not be using it later.

Next install evaluation licenses for both `Jira Core (Server)` and `Jira Software (Server)` as mentioned
[here](jira-software-license.md). The steps to get a license for Jira Core are the same as for Jira Software.
You might already have been forced to create the Core license during these initial steps, if that happened don't
forget to do the Jira Software license. On a fresh install you might have to install `Jira Software (Server)` from the [Applications](http://localhost:2990/jira/plugins/servlet/applications/versions-licenses) page of the Jira Adminstration pages.

Once Jira Core and Jira Software are both installed and licensed, open another terminal window and cd into the `$OB_DIR`folder.

Now run
```
$atlas-package -Dob.ui.deps -Dob.ui.dev -Dob.jira8
Executing: /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/bin/mvn package -gs /Applications/Atlassian/atlassian-plugin-sdk-8.0.7/apache-maven-3.5.4/conf/settings.xml -Dob.ui.deps -Dob.ui.dev -Dob.jira8
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
...
[INFO] Overbård Plugin for Jira 8.x 1.0.0.Beta6-SNAPSHOT .. SUCCESS [  4.454 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:11 min
[INFO] Finished at: 2019-06-23T17:05:05+02:00
[INFO] ------------------------------------------------------------------------
```
The `-Dob.ui` and `-Dob.ui.deps` system properties tell the build to
build the user interface and include it in the final plugin archive. We will disuss these in more detail later. Note that the first time you do this it will take quite some time - your build is most likely not hanging!

Once the plugin is refreshed in Jira, go to [http://localhost:2990/jira](http://localhost:2990/jira) again (or refresh the page). Click on the top `Boards` menu and make sure it contains an `Overbård` entry.

Let's populate Jira with some sample data. Open another terminal, and clone the
[https://github.com/overbaard/overbaard-jira-populator](https://github.com/overbaard/overbaard-jira-populator)
project. `cd` into the folder and run:
```
$mvn install exec:java -Dexec.mainClass="org.overbaard.jira.populator.JiraPopulatorMain"
...
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ overbaard-jira-populator ---
Creating projects....
====== UP
Checking if UP exists...
Project UP does not exist
Creating project UP...
Created project UP(10100)
Creating issue...
Created issue UP-1
Creating issue...
Created issue UP-2
Creating issue...
...
Linked SUP-30 to UP-30
Created projects
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 53.796 s
[INFO] Finished at: 2018-09-28T21:59:30+01:00
[INFO] Final Memory: 29M/335M
[INFO] ------------------------------------------------------------------------
```

This sets up users, three projects (`FEAT`, `SUP` and `UP`) and populates them with components, fix versions,
labels and issues distributed across the various states so you don't have to do that yourself.

Go to [http://localhost:2990/jira/rest/api/2/field](http://localhost:2990/jira/rest/api/2/field) and
make a note of the id of the `Rank` custom field ID as well as for the fields `Epic Link` and `Epic Name`. (These ID's are normally in the low end of 10000.)

Go to [http://localhost:2990/jira](http://localhost:2990/jira), and in the `Boards` menu select `Overbård`.
Once Overbård has loaded, click the hamburger icon in the top left and choose `Config`.
Set the `Rank`, `Epic Link` and `Epic Name` IDs in the `Custom Field IDs` section on the bottom of the page, using the values you noted in the last step.

Next, enter the contents of
[https://github.com/overbaard/overbaard/blob/master/jira/plugin/core/src/setup/board.json](https://github.com/overbaard/overbaard/blob/master/jira/plugin/core/src/setup/board.json)
into the `Create a new board` text area. Click on the hamburger icon in the top left again and go to `Boards`.
Select the `Test board`, and you should see the board fully populated.

Jira seems a bit haphazard in what it calls the `Done` state in the projects we generated (in some cases it is called
`Done`, in others `DONE`). If you see the alarm clock icon on the lower right when viewing the board, click it
and it will tell you what the state should be. If you don't see any alarm clock icon all is well! If it needs fixing go
back to the config page and edit the `state-links` of the `FEAT` and `SUP` projects by changing the left hand side of
the `Done` mappings to what the state should be. Similarly edit the `Done` entry in the `states` array of the
`UP` linked project.

Your Jira instance is now all set up. To understand better what the `board.json` config we pasted in means, see the
[Admin Guide](admin-guide.md).


The last step is to get the web layer environment set up properly. In another terminal go to the `webapp/` directory
of the Overbård checkout:
```
$cd $OB_DIR/webapp
```
Then:
```
$yarn install
...
✨  Done in 54.86s.
```
And finally:
```
$ng serve
** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **

Date: 2018-09-28T21:20:36.736Z
Hash: b5680d064aa1deccee97
Time: 18936ms
chunk {main} main.js, main.js.map (main) 2.15 MB [initial] [rendered]
chunk {polyfills} polyfills.js, polyfills.js.map (polyfills) 325 kB [initial] [rendered]
chunk {runtime} runtime.js, runtime.js.map (runtime) 5.22 kB [entry] [rendered]
chunk {styles} styles.js, styles.js.map (styles) 77.6 kB [initial] [rendered]
chunk {vendor} vendor.js, vendor.js.map (vendor) 6.8 MB [initial] [rendered]
ℹ ｢wdm｣: Compiled successfully.
```
If you go to [http://localhost:4200](http://localhost:4200) you should see the board running using some sample data that does not come from Jira. Note that only the top two board links will actually take you anywhere.

# 3 Project structure
We have the following main folders/maven modules making up the project:

* `webapp` - Contains the Angular web application files. The built Angular web application gets bundled into the Jira plugin. The system properties starting with `ob.ui` mentioned in the next section, only have an effect on this module.
* `jira/api-adapter` - This deals with differences between the various Jira versions supported. The supported Jira versions have some changes in the APIs that we use. This folder contains an  `spi` module which has the common interfaces for the abstracted behavious. Then there is a `jira7` and a `jira8` module which contain the implementations of the differing calls to the Jira SDK APIs.
* `jira/bom` - This contains the different Jira API dependencies, and the relevant imports from the `jira/api-adapter` for the pluging version.
* `jira/plugin` - This has a `core` module, which contains the core of the Overbård back-end. It calls out to classes in  `jira/api-adapter/spi` as necessary. The output from the `webapp` build, gets put into the `core` target directory. Finally, there are `jira7` and a `jira8` sub-modules. These each bundle the output of the `core` plugin for the target Jira version, pulling in the correct sub-module dependencies from `jira/api-adapter` and `jira/bom`. In short, the jar created by `jira/plugin/jira8` targets Jira 8, and the jar created by `jira/plugin/jira7` targets Jira7.



# 4 Building/running/debugging the project
The UI can be developed separately from the plugin itself, since it is a fat client interacting with the server via a REST API. So when modifying the UI files, you don't need to build, package and deploy in the server. The client
logic is written in Typescript, and the UI steps are responsible for compiling the Typescript to Javascript. So
depending on the focus of your current task, you can either do:

* just the UI steps (if working on purely display logic)
* or both the UI steps and the Atlassian SDK steps if you are working on something involving the server. The SDK steps
will package the jar containing the plugin.

## 4.1 Angular UI

Each of the following UI steps happen from the `$OB_DIR/webapp` folder, and a separate terminal window is needed
for each one.

1. Run `ng serve`. This builds the webapp files and serves them up at [http://localhost:4200](http://localhost:4200).
Any errors from compiling will show up in this terminal window. Your files will be 'watched' so you can leave this
running, and any changes will be recompiled (if you start seeing strange  behaviour, restart this job).
The `$OB_DIR/webapp/src/rest/overbaard/1.0` sub-folder contains some mock data files for running without a real
jira instance so you can see how your changes affect the layout. As you make changes and they compile, the browser
window refreshes automatically.
2. Run `ng test` which runs the client-side test suite. The tests mainly focus on the integrity of the redux store
and the calculated view model.

## 4.2 Atlassian SDK

**WARNING**: If you run `atlas-clean` you will have to set everything up again according to the steps we saw above. Sometimes cleaning is the only way to 'recover'. To make your life easier, back up your Jira instance now and again so you can simply restore the backup. This is also a good way to 'share' the data if you want to check your work in all versions of Jira. It is advisable to do this from the Jira 7 instance of your setup, as if you do the backup from the Jira 8 instance the Jira 7 instance will complain about the backup being from a newer version.

The commands happen from the root (`$OB_DIR`) folder of the project (i.e where `pom.xml` is located). I normally use one window for running the server instance and another to package the project. Stopping and starting the server takes a lot of time. When you repackage the plugin it gets picked up by the running server. We typically use two terminal windows, one for running the server, and one for packaging it.

### 4.2.1 Running the server

Before we can run the server, we need to build it and make the core artifacts available in the local Maven repository. This is done by running `atlas-mvn install`. This is generally only needed before you are going to start the server before starting a development session.

The next step is to run the server. If you want to run the Jira 7 server, you run `atlas-debug -pl jira/plugin/jira7 -Dob.jira7`. To run the Jira 8 server you run `atlas-debug -pl jira/plugin/jira8 -Dob.jira8`.

The `jira/plugin/jira7` folder is only available in a Maven profile activated by the `ob.jira7` system property, and the `jira/plugin/jira8` folder is only activated by the `ob.jira8` system property.

### 4.2.2 Repackaging the application
Once you change something, and want to deploy it into Jira, run one of the following from another terminal window:

* `atlas-package -Dob.jira7` - if you started the Jira 7 server, or
* `atlas-package -Dob.jira8` - if you started the Jira 8 server

These commands build the plugin again, and deploys it into the running Jira instance we started in the previous step. It simply builds the Java files from the various modules, and bundles any already
built UI files from the `webapp` bundle into the resulting
Overbård plugin jar. We have some system properties to do more work for the UI:
  * `-Dob.ui.deps` - this installs a copy of yarn and node so that they are usable from the maven
   plugin used to bundle the UI. If, when pulling changes from git, any of the dependencies in
   `$OB_DIR/webapp/package.json` have changed, you should delete the `$OB_DIR/webapp/node-modules` folder, and run
   `atlas-package -Dob.ui.deps to get the fresh dependencies. This should not be needed very often.
  * `-Dob.ui.dev` - this runs the Angular CLI build which refreshes the web application files to be used in the
   Overbård plugin jar. Since the Angular CLI build takes some time to do its work, `atlas-package` on its own does not build and refresh the web application files. This means that you can work on server-side code without the delay. If you are working on the web application files, and want to see the changes in your local Jira instance, `atlas-package -Dob.jira8 -Dob.ui` (or `atlas-package -Dob.jira7 -Dob.ui` depending on your Jira version) to trigger the bundling and refreshing of the web application files on the Overbård
   plugin jar.
  * `-Dob.ui` - this is like `-Dob.ui` but slower as it does optimisations for building the web application
   files for a production environment. This option should be used if you ever do a proper release of Overbård.

Some sample full commands:

* `atlas-package -Dob.jira7 -Dob.ui.deps -Dob.ui.dev` - This builds the Jira 7 version of the plugin, downloads the Angular dependencies and triggers a development environment build of the web application to be bundled in the plugin.
* `atlas-package -Dob.jira8` - This builds the Jira 8 version of the plugin, without building the UI (simply using what is already in the `webapp/target` directory as input to the plugin build)
* `atlas-package -Dob.jira8 -Dob.jira7 -Dob.ui` - This builds both the Jira 7 and the Jira 8 versions of the plugin, and triggers a production environment build of the web application to be bundled in the plugin.

### 4.3 Doing a release
The packaging of the application as shown above works fine for deploying in the locally running SDK. But it seems that it is more strict when we need to deploy the plugin in a proper Jira instance. I found that the safest way to do this is to start from scratch. The steps to do this are (hopping a bit between two terminals):

**Terminal 1:**

Check out the tag to release

`$atlas-clean -Dob.jira8 -Dob.jira7`

`atlas-mvn install`

`atlas-package -Dob.ui.deps -Dob.ui -Dob.jira7 -Dob.jira8`

`atlas-debug -pl jira/plugin/jira7 -Dob.jira7`

Once the server has started, restore from a backup and download Jira Software if needed. The backup should contain the licenses to enable everything.

**Terminal 2:**

`atlas-package -Dob.ui.deps -Dob.ui -Dob.jira7 -Dob.jira8`

**Terminal 1:**

Make sure the plugin deploys fine, and that you are able to use it in the browser. Grab the Jira7 jar and repeat the steps for Jira 8.

**Final verification**

Make sure that both the Jira 7 and Jira 8 versions of the plugin work properly in standalone Jira 7 and 8 instances.  You can download these from https://www.atlassian.com/software/jira/download?_ga=2.149634613.283978762.1561365767-1681422928.1520267962 (Or google 'Download Jira Software Server').
Once initialised, you can restore data in these from a backup.

Then deploy the correct plugin version into each, and make sure it works.
