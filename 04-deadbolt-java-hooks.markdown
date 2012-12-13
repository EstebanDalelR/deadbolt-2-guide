# Using Deadbolt 2 with Play 2 Java projects #

Deadbolt 2 for Java provides an idiomatic API for dealing with Java controllers and templates rendered from Java controllers in Play 2 applications.  It takes advantage of the features offered by Play 2 such as access to the HTTP context - vital, of course, for a framework that says it embraces HTTP - to give access to the current request and session, and the annotation-driven interceptor support.

## The Deadbolt Handler ##
For any module - or framework - to be useable, it must provide a mechanism by which it can be hooked into your application.  For D2, the central hook is the `be.objectify.deadbolt.java.DeadboltHandler` interface.  The four methods defined by this interface are crucial to Deadbolt - for example, `DeadboltHandler#getSubject` gets the current user (or subject, to use the correct security terminology), whereas `DeadboltHandler#onAccessFailure` is used to generate a response when authorisation fails.

Despite the use of the definite article in the section title, you can have as many Deadbolt handlers in your app as you wish.  If you explicitly specify handlers at the annotation or tag level instead of using the default handler specified in application.conf, you will see the advantage of this.

### Result beforeAuthCheck(Http.Context context) ###
  To achieve this goal, D2 invokes the beforeRoleCheck method of your DeadboltHandler

### Subject getSubject(Http.Context context) ###
This method returns the current subject, if any.  Returning null indicates there is no user currently logged in - this is a valid scenario.

### Result onAccessFailure(Http.Context context, String content) ###
When authorisation fails, this method is invoked on the current `DeadboltHandler` to deal with the situation.  The result returned from this method is a regular `play.mvc.Result`, so it can be anything you chose.  You might want to return a 403 forbidden, redirect to a location accessible to everyone, etc.

### DynamicResourceHandler getDynamicResourceHandler(Http.Context context) ###
DynamicResourceHandler implementations implement the dynamic constraints of D2.  See below for further details.

## General configuration ##

Now you have implemented a `DeadboltHandler`, the next step is to hook it into your application.

### application.conf ###
declare it in your application.conf.  This tells Deadbolt which handler you wish to use as a default when one isn't explicitly specified in the D2 constraint.  This will be explained in more detail when we look at the individual constraints.

For a `DeadboltHandler` implementation whose fully-qualified name is `com.example.myapp.security.MyDeadboltHandler`, your application.conf would have the entry

    deadbolt.java.handler=com.example.myapp.security.MyDeadboltHandler

Personally, I prefer the HOCON (Human-Optimized Config Object Notation) syntax supported by Play, so I would recommend the following:

    deadbolt {
        java {
            handler=com.example.myapp.security.MyDeadboltHandler
        }
    }

This declaration is used by the `DeadboltPlugin` to determine the default D2 handler for your application.

### The Deadbolt Plugin ###
The initial version of D2 held the default `DeadboltHandler` implementation, as specific in application.conf, as a static variable.  This led immediately to problems with the hot reload carried out by Play when it detects code changes at runtime.  To address this issue, the default handler is now created by and held in a plugin, whose lifecycle is managed by the framework.

To activate the plugin, you need to update your `conf/play.plugins` file.  If this file doesn't already exist (it's not created automatically when you create a new Play app), then add it yourself.  The necessary line is

    10000:be.objectify.deadbolt.java.DeadboltPlugin

The `10000` defines the loading priority of the plugin -lower numbers are loaded first.  If you have no other plugins, this is irrelevant.  If you do have other plugins, you'll need to work out the best way to order them.  D2 should, in theory and so far in practice, play nicely with other modules so its loading priority (again, in theory) should not matter.  File this under "Things I should be aware of".

When the plugin starts, it will attempt to read the class name specified by `deadbolt.java.handler` in application.conf.  If found, an class of the class will then be instantiated.  If the instantiation fails, an exception will be thrown so you receive a fail-fast response on start-up.

As mentioned above, it is a valid scenario to have no default handler specified.  If `deadbolt.java.handler` doesn't appear in your configuration, a warning will be logged to alert you of this.