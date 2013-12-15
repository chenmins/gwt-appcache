gwt-appcache
============

[![Build Status](https://secure.travis-ci.org/realityforge/gwt-appcache.png?branch=master)](http://travis-ci.org/realityforge/gwt-appcache)

The HTML5 Appcache specification is one mechanism for enabling offline
HTML5 applications. This library provides a simple way to generate the
required cache manifests and serve for each separate permutation. The
library also supports the browser side aspect of appcache
specification. The appendix section includes further references for
the appcache spec.

Quick Start
===========

The simplest way to appcache enable a GWT application is to;

* add the following dependencies into the build system. i.e.

```xml
<dependency>
   <groupId>org.realityforge.gwt.appcache</groupId>
   <artifactId>gwt-appcache-client</artifactId>
   <version>0.4</version>
   <scope>provided</scope>
</dependency>
<dependency>
   <groupId>org.realityforge.gwt.appcache</groupId>
   <artifactId>gwt-appcache-linker</artifactId>
   <version>0.4</version>
   <scope>provided</scope>
</dependency>
<dependency>
   <groupId>org.realityforge.gwt.appcache</groupId>
   <artifactId>gwt-appcache-server</artifactId>
   <version>0.4</version>
</dependency>
```

It should be added that the library is not yet currently deployed
to maven central. The library has been stored in a pseudo-maven
repository layout on github and can be downloaded if you add a
repository url such as;

  http://github.com/realityforge/repository/raw/master

* add the following snippet into the .gwt.xml file.

```xml
<module rename-to='myapp'>
  ...

  <!-- Enable the client-side library -->
  <inherits name="org.realityforge.gwt.appcache.Appcache"/>

  <!-- Enable the linker -->
  <inherits name="org.realityforge.gwt.appcache.linker.Linker"/>

  <!-- enable the linker that generates the manifest -->
  <add-linker name="appcache"/>

  <!-- configure all the static files not managed by the GWT compiler -->
  <extend-configuration-property name="appcache_static_files" value="./"/>
  <extend-configuration-property name="appcache_static_files" value="index.html"/>
</module>
```

* configure html that launches the application to look for the manifest.

```xml
<!doctype html>
<html manifest="myapp.appcache">
   ...
</html>
```

* declare the servlet that serves the manifest.

```java
@WebServlet( urlPatterns = { "/myapp.manifest" } )
public class ManifestServlet
  extends AbstractManifestServlet
{
  public ManifestServlet()
  {
    addPropertyProvider( new UserAgentPropertyProvider() );
  }
}
```

* interact with the application from within the browser.

```java
final ApplicationCache cache = ApplicationCache.get();
if ( null != cache )
{
  cache.addUpdateReadyHandler( new UpdateReadyEvent.Handler()
  {
    @Override
    public void onUpdateReadyEvent( @Nonnull final UpdateReadyEvent event )
    {
      //Force a cache update if new version is available
      cache.swapCache();
    }
  } );

  // Ask the browser to recheck the cache
  cache.update();

  ...
```


This should be sufficient to get your application using the appcache. If you
load the application in a modern browser you should see it making use of the
cache in the console.

How does it work?
=================

For every permutation generated by the GWT compiler, a separate manifest file
is generated. The manifest includes almost all public resources generated by
GWT with the exception of some used during debugging and development (i.e.
myapp.devmode.js and compilation-mappings.txt). The manifest also includes
any additional files declared using the "appcache_static_files" configuration
setting.

After the GWT compiler has generated all the different permutations, a single
xml descriptor permutations.xml is generated that lists all the permutations
and the  deferred-binding properties that were used to uniquely identify the
permutations. Typically these include values of properties such as "user.agent".

If the compiler is using soft permutations then it is possible that multiple
deferred-binding properties will be served using a single permutation, in which
case the descriptor will have comma separate values in the permutations.xml for
that permutation. It is possible to explicitly ignore specific deferred-binding
properties by extending the gwt property "appcache_ignorable_permutation_properties".
This is useful for when runtime detection is used to test for a feature regardless.
This is configuration actually defaults to including some html5 features such as
"audioElementSupport".

The manifest servlet is then responsible for reading the permutations.xml and
inspecting the incoming request and generating properties that enable it to select
the correct permutation and thus the correct manifest file. The selected manifest
file is returned to the requester.

Appendix
--------

* "A Beginner's Guide to Using the Application Cache":http://www.html5rocks.com/en/tutorials/appcache/beginner/
* "Appcache Facts":http://appcachefacts.info/

Credit
------

This library began as a enhancement of similar functionality in the
"MGWT":https://github.com/dankurka/mgwt project by Daniel Kurka. All
credit goes to Daniel for the initial code and idea.
