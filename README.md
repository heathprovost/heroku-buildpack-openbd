Heroku buildpack: Open Bluedragon
=========================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpack) for [Open Bluedragon](http://openbd.org/). It uses Jetty-Runner to run your app.

Prerequisites
-----

* [Java JVM](http://www.java.com/en/download/index.jsp)
* [OpenBD Desktop](http://openbd.org/downloads/)
* [Heroku Toolbelt](https://toolbelt.heroku.com/)

Overview
-----

This buildpack is targeted for developers wanting to run OpenBD on Heroku without a lot of fuss. It assumes you are already accustomed to using OpenBD Desktop for doing local development. Normally, OpenBD applications
are deployed on J2EE servers using .war files. While Heroku does support [war deployment](https://devcenter.heroku.com/articles/war-deployment), it does not currently work for OpenBD applications. This buildpack is for developers who would prefer to use Heroku's native git deployment model, which allows for fast and efficient
deployment and small slug sizes.

Git
-----

You **must** use git for revision control in order to deploy to Heroku. The easiest way to do this is to grab
the [Heroku Toolbelt](https://toolbelt.heroku.com/). If you are new to git, start [here](https://devcenter.heroku.com/articles/git).

Basic Usage
-----

Starting from scratch, you have an OpenBD web app that looks something like this:

    $ ls
    WEB-INF  bluedragon  index.cfm

You probably want a minimal .gitignore file to keep transient files out of revision control:

    WEB-INF/bluedragon/work/
    WEB-INF/bluedragon/bluedragon.xml.bak.*  

If you have not put your app into git yet, you would do this:

    $ git init
    $ git add .
    $ git commit -m "1st commit"

Now your Heroku application is ready to be created:

    $ heroku create --stack cedar --buildpack http://github.com/heathprovost/heroku-buildpack-openbd.git your-app-name
    Creating your-app-name... done, stack is cedar
    BUILDPACK_URL=http://github.com/heathprovost/heroku-buildpack-openbd.git
    http://your-app-name.herokuapp.com/ | git@heroku.com:your-app-name.git
    Git remote heroku added

All that is left to do is deploy it:    

    $ git push heroku master
    ...
    -----> Fetching custom git buildpack... done
    -----> OpenBD app detected
    -----> Using Developer Supplied OpenBD Engine...
    -----> Installing Default Procfile...done
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size: 39.1MB
    -----> Launching... done, v14
    http://your-app-name.herokuapp.com deployed to Heroku

That's it. Your application should now be running on Heroku. In the basic usage model, you are supplying the OpenBD engine, i.e. Heroku will run and deploy whatever version of OpenBD you are using locally. The buildpack tries to be liberal and will detect your app as OpenBD if it has any of the following files in it:

* index.cfm
* Application.cfm
* Application.cfc
* WEB-INF/bluedragon/bluedragon.xml
* WEB-INF/lib/OpenBlueDragon.jar

Advanced Usage
-------

### Your Own Procfile

The buildpack supplies it's own Prcifle, but if you prefer to use your own [Procfile](https://devcenter.heroku.com/articles/procfile) simply incude it in the root of your project. You may want to do this if you are going to include worker processes, but remember, you do not have Maven or the JDK so the most you can do is execute jar files. If you choose to do this, you need the web process to look like this:

    web: java $JAVA_OPTS -jar WEB-INF/lib/jetty-runner.jar --port $PORT .

### Thin Deployment

You can optionally choose to use a thin deployment model. With the thin deployment model, the OpenBD
engine is provided by the buildpack. The buildpack includes a full install of OpenBD 2.0.2 stable, minus the administration console, and will install and use it for all thin deployments.

If you want to use the thin deployment model, you must remove the bulk of the OpenBD engine from revision control. You could do this by simply deleting all of the engine files - you can in fact deploy nothing but an index.cfm file and it will work. However, this gives you no way to do local testing. Instead, what you can do is use a .gitignore file in the root of your site to ommit the egine from revision control. An example is provided below:

    bluedragon/adminapi
    bluedragon/administrator
    bluedragon/jars
    bluedragon/scripts
    WEB-INF/bin/
    WEB-INF/bluedragon/*
    WEB-INF/classes/
    WEB-INF/lib/
    WEB-INF/webresources/
    !WEB-INF/bluedragon/bluedragon.xml
    !WEB-INF/web.xml
    !WEB-INF/customtags/*

This tells git to ignore the bulk of the OpenBD engine. It makes three exceptions (the lines starting with an !):

* It includes your bluedragon.xml file so you can deploy updated configurations to Heroku.
* It includes web.xml in case you want to customize it.
* It includes the entire customtags directory.

This can be customized to your own needs. Any files you allow into revision control will be used by the buildpack, so you can provided customizations as needed. For example, if you wanted to run the administrator console on Heroku, just remove the line ommitting it from the .gitignore file. The only file you absolutely
must omit from revision control is WEB-INF/lib/OpenBlueDragon.jar. The omision of this file is what triggers
thin deployment in the buildpack.

During deployment, any files you allow into revision control are overlayed over the default engine files.

Note: It is **highly** recommended that if you choose to use thin deployments you use the same version of OpenBD
for development (i.e. 2.0.2). If you want to run a different version or need to use the nightlies, you are better
off for now using the full deployment model (i.e. include everything).

License
-------

Licensed under the MIT License. See LICENSE file.
