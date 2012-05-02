---
layout: post
title: Deploying Broadleaf Commerce on Heroku
tags: programming, Java, Broadleaf, Spring, Maven
---

This post is exactly what the title says; a short how-to to deploy [Broadleaf](http://www.broadleafcommerce.org) to Heroku, a popular cloud platform. This has been done with the Braodleaf 1.6.0 and all the CLI commands were run on OSX, but the commands should be 1-1 for Linux. If you use Windows, you're gonna have a bad time.

First, let's pop open a command line and build the archetype with maven with the following command:

{% highlight console %}
~/broadleaf/heroku > mvn archetype:generate -DarchetypeGroupId=org.broadleafcommerce -DarchetypeArtifactId=ecommerce-archetype -DarchetypeVersion=1.6.0-SNAPSHOT
{% endhighlight %}

Fill in the relevant prompts, and you should see some output like this:
{% highlight console %}
   [INFO] Scanning for projects...
    [INFO]                                                                         
    [INFO] ------------------------------------------------------------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] ------------------------------------------------------------------------
    [INFO] 
    [INFO] >>> maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom >>>
    [INFO] 
    [INFO] <<< maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom <<<
    [INFO] 
    [INFO] --- maven-archetype-plugin:2.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Interactive mode
    [INFO] Archetype repository missing. Using the one from [org.broadleafcommerce:ecommerce-archetype:1.5.4-GA] found in catalog remote
    Define value for property 'groupId': : com.phillip
    Define value for property 'artifactId': : heroku
    Define value for property 'version':  1.0-SNAPSHOT: : 
    Define value for property 'package':  com.phillip: : 
    Confirm properties configuration:
    groupId: com.phillip
    artifactId: heroku
    version: 1.0-SNAPSHOT
    package: com.phillip
     Y: : 
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: ecommerce-archetype:1.6.0-SNAPSHOT
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.phillip
    [INFO] Parameter: artifactId, Value: heroku
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.phillip
    [INFO] Parameter: packageInPathFormat, Value: com/phillip
    [INFO] Parameter: package, Value: com.phillip
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: groupId, Value: com.phillip
    [INFO] Parameter: artifactId, Value: heroku
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/admin/pom.xml
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/admin-war/pom.xml
    [WARNING] CP Don't override file /Users/phillipverheyden/broadleaf/heroku/heroku/admin-war/pom.xml
    [WARNING] CP Don't override file /Users/phillipverheyden/broadleaf/heroku/heroku/admin-war/src/main/webapp/META-INF/MANIFEST.MF
    [WARNING] CP Don't override file /Users/phillipverheyden/broadleaf/heroku/heroku/admin-war/src/main/webapp/WEB-INF/tld/spring-form.tld
    [WARNING] CP Don't override file /Users/phillipverheyden/broadleaf/heroku/heroku/admin-war/src/main/webapp/WEB-INF/tld/spring.tld
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/core/pom.xml
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/site/pom.xml
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/site-war/pom.xml
    [WARNING] CP Don't override file /Users/phillipverheyden/broadleaf/heroku/heroku/site-war/pom.xml
    [INFO] Parent element not overwritten in /Users/phillipverheyden/broadleaf/heroku/heroku/test/pom.xml
    [INFO] project created from Archetype in dir: /Users/phillipverheyden/broadleaf/heroku/heroku
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 30.095s
    [INFO] Finished at: Sat Apr 14 19:58:02 CDT 2012
    [INFO] Final Memory: 7M/81M
    [INFO] ------------------------------------------------------------------------ 
{% endhighlight %}

As you can see, the archetype ends up generating 2 applications: admin and site. The wars are built from admin-war and site-war.  In a perfect world, we would leave the structure as-is and deploy both applications to Heroku simultaneously. However, Heroku limits the 'slug' size (how big of an application you are deploying) to 100MB and both admin.war and site.war are right around 60MB apiece. I'll deal with the admin application in a different post (it's a little tricky since it's in GWT), but for now we're just going to remove it (or move it to a different location).

While we're on the subject of slug size, I found a nice plugin for helping reduce the size of the target directory that Maven builds courtesy of [Chris Auer](http://enlightenmint.com/blog/2012/01/03/reducing-slug-size-for-heroku.html). Add this section to the build -> plugins section of the core, site and site-war pom.xml:
{%highlight xml %}
<plugin>
    <artifactId>maven-clean-plugin</artifactId>
    <version>2.4.1</version>
    <configuration>
        <filesets>
            <fileset>
                <directory>target/</directory>
                <includes>
                    <include>**/*</include>
                </includes>
                <!-- This exclusion is only necessary in the site-war project is for the Tomcat plugin that will be addressed a little later -->
                <excludes>
                    <exclude>cargo/**</exclude>
                </excludes>
                <followSymlinks>false</followSymlinks>
            </fileset>
        </filesets>
        <excludeDefaultDirectories>true</excludeDefaultDirectories>
    </configuration>
    <executions>
        <execution>
            <id>auto-clean</id>
            <phase>install</phase>
            <goals>
                <goal>clean</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

Now let's go in and make the necessary database config changes to work with Heroku.

applicationContext.xml original:
{% highlight diff %}
diff --git a/site-war/src/main/webapp/WEB-INF/applicationContext.xml b/site-war/src/main/webapp/WEB-INF/applicationContext.xml
index 51dba06..9dcd367 100644
--- a/site-war/src/main/webapp/WEB-INF/applicationContext.xml
+++ b/site-war/src/main/webapp/WEB-INF/applicationContext.xml
@@ -37,18 +37,22 @@
         <property name="defaultDataSource" ref="webDS"/>
     </bean>
     
+    <bean class="java.net.URI" id="dbUrl">
+        <constructor-arg value="${DATABASE_URL}"/>
+    </bean>
+    
     <bean id="webDS" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
-               <property name="driverClassName" value="org.hsqldb.jdbcDriver" />
-               <property name="url" value="jdbc:hsqldb:hsql://localhost/broadleaf;ifexists=true" />
-               <property name="username" value="sa" />
-               <property name="password" value="" />
+               <property name="driverClassName" value="org.postgresql.Driver" />
+               <property name="url" value="#{ 'jdbc:postgresql://' + @dbUrl.getHost() + @dbUrl.getPath() }" />
+               <property name="username" value="#{ @dbUrl.getUserInfo().split(':')[0] }" />
+               <property name="password" value="#{ @dbUrl.getUserInfo().split(':')[1] }" />
        </bean>
 
     <bean id="webStorageDS" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
-               <property name="driverClassName" value="org.hsqldb.jdbcDriver" />
-               <property name="url" value="jdbc:hsqldb:hsql://localhost/broadleaf;ifexists=true" />
-               <property name="username" value="sa" />
-               <property name="password" value="" />
+               <property name="driverClassName" value="org.postgresql.Driver" />
+        <property name="url" value="#{ 'jdbc:postgresql://' + @dbUrl.getHost() + @dbUrl.getPath() }" />
+        <property name="username" value="#{ @dbUrl.getUserInfo().split(':')[0] }" />
+        <property name="password" value="#{ @dbUrl.getUserInfo().split(':')[1] }" />
        </bean>
 
     <bean id="blCacheManager"
{% endhighlight %}

Now comes a tricky part. The nice thing about Heroku is that you can use it with any number of applications: Ruby, Node, Java, Python, whatever. But this is a double-edged sword: Heroku doesn't give you a nice application server (like Tomcat or Jetty) to deploy to out of the box. You have to include it yourself in your Maven profiles. The Heroku documentation explicitly talks about using embedded Jetty, but I couldn't get that to work and prefer Tomcat anyway.  The Maven execution that I added for Tomcat actually goes out and downloads a Tomcat zip, unpacks it, then deploys your war inside of it as ROOT. This is achieved with the codehaus cargo plugin:

in site-war/pom.xml
{% highlight xml %}
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <configuration>
        <container>
            <containerId>tomcat6x</containerId>
            <zipUrlInstaller>
                <url>http://archive.apache.org/dist/tomcat/tomcat-6/v6.0.18/bin/apache-tomcat-6.0.18.zip</url>
            </zipUrlInstaller>
            <dependencies>
                <dependency>
                    <groupId>javax.activation</groupId>
                    <artifactId>activation</artifactId>
                </dependency>
                <dependency>
                    <groupId>javax.mail</groupId>
                    <artifactId>mail</artifactId>
                </dependency>
            </dependencies>
        </container>
        <configuration>
            <type>standalone</type>
            <deployables>
                <deployable>
                    <groupId>com.phillip</groupId>
                    <artifactId>site-war</artifactId>
                    <type>war</type>
                    <properties>
                        <context>ROOT</context>
                    </properties>
                </deployable>
            </deployables>
        </configuration>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>install</goal>
                <goal>configure</goal>
                <goal>deploy</goal>
                <goal>package</goal>
            </goals>
        </execution>
    </executions>
</plugin>
{% endhighlight %}

You can obviously change the Tomcat version to whatever you like. I personally chose Tomcat 6. Tomcat 7 would work the same way.

One more Tomcat-specific item. Heroku assigns a random port number for your application, but Tomcat always defaults to 8080. To allow for this, I created a heroku-server.xml that the Heroku Procfile will rename to server.xml and move to the Tomcat deploy directory and allows for the dynamic port binding. I put this in the root of the site-war directory

{% highlight xml%}
<?xml version='1.0' encoding='utf-8'?>
<!-- This is needed in order to tell Tomcat to start up on the dynamic port pass in via the Procfile -->
<Server port="-1"> 
    <Listener className="org.apache.catalina.core.JasperListener" /> 
    <Service name="Catalina"> 
        <!-- http.port will be assigned as an environment variable from the Procfile -->
        <Connector port="${http.port}" protocol="HTTP/1.1" connectionTimeout="20000"/> 
        <Engine name="Catalina" defaultHost="localhost"> 
            <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true"/> 
        </Engine>
    </Service> 
</Server>
{% endhighlight %}

Now the only thing that's left are the Heroku-specific files. The Procfile is extremely simple. All it does is kick off a custom shell script which does all the heavy lifting

Procfile:
{% highlight bash %}
web: sh startup-heroku.sh
{% endhighlight %}

The real meat comes from startup-heroku.sh:
{% highlight bash %}
#point to the correct configuration and webapp
CATALINA_BASE=`pwd`/site-war/target/cargo/configurations/tomcat6x
export CATALINA_BASE

#copy over the Heroku config files
cp ./site-war/heroku-server.xml ./site-war/target/cargo/configurations/tomcat6x/conf/server.xml

#make the Tomcat scripts executable
chmod a+x ./site-war/target/cargo/installs/apache-tomcat-6.0.18/apache-tomcat-6.0.18/bin/*.sh

#set the correct port and database settings
JAVA_OPTS="$JAVA_OPTS -XX:MaxPermSize=256M -Xmx256M -Dhttp.port=$PORT -DDATABASE_URL=$DATABASE_URL"
export JAVA_OPTS

#start Tomcat
./site-war/target/cargo/installs/apache-tomcat-6.0.18/apache-tomcat-6.0.18/bin/catalina.sh run
{% endhighlight %}

And that's it for the configuration!  Now the only thing left is to start up the app!

If you haven't already, go ahead and create the Heroku cedar application which automatically adds a new git remote
{% highlight console %}
~/broadleaf/heroku > heroku create --stack cedar
Creating glowing-river-8401... done, stack is cedar
http://glowing-river-8401.herokuapp.com/ | git@heroku.com:glowing-river-8401.git
Git remote heroku added 
{% endhighlight %}

Then just deploy with git push heroku and you're done! To check for startup problems, you can tail the logs with heroku logs --tail which shows you the real-time logging info
