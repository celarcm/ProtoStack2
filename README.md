ProtoStack
==========

ProtoStack is a tool for experimentation with dynamic composition of communication services.
The implementation of ProtoStack was triggered by a wireless sensor network testbed
and is used for experimentation with cognitive radio and cognitive networking in the
frame of the CREW project. As such, ProtoStack is designed in a way to
ease research and experimentation with communication networks, particularly with
cognitive networks. The system was designed so that an advanced user, such as the
component developer needs, to focus on developing the component and make it work
with Contiki OS and a novice user needs only to focus on composing services in a stack
using the workbench.

ProtoStack needs the following components:
- JRE6 or newer
- Apache
- Tomcat
- Sesame 2.x (http://www.openrdf.org/download.jsp)
- The WireIT library
- Contiki OS
- The toolchain allowing building the Contiki image and programming your HW with it

ProtoStack's components are:
- The server (https://github.com/sensorlab/ProtoStack/tree/master/src)
- The ontology (https://github.com/sensorlab/ProtoStack/tree/master/owl)
- The workbench (https://github.com/sensorlab/ProtoStack/tree/master/crimeLayers)
- The CRime library (https://github.com/sensorlab/CRime)


More detailed documentation will come soon.

Set up guide
=================

This a step-by-step guite to set up the ProtoStack tool. All required components are listed in the Readme file.

1. Clone the repository into your local filesystem
2. Install Apache server and set it up to serve ProtoStack/crimeLayers folder
3. Install Apache Tomcat using:
 - apt-get install apache7
 - sudo apt-get install tomcat7-admin (to get the manager webapp)
4. Set up Tomcat:
 - Change the tomcat7 port to 8090: http://www.mkyong.com/tomcat/how-to-change-tomcat-default-port/
 - add a user to access manager webapp. Run "sudo nano /etc/tomcat7/tomcat-users.xml" and add the following line to the file:
 <user username="admin" password="password" roles="manager-gui,admin-gui"/>
 For more info visit: https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-7-on-ubuntu-14-04-via-apt-get
5. Download and extract the OpenRDF Sesame framework. I used version 2.5.1.
6. Set up and deploy OpenRDF Sesame to Tomcat:
 - open the openrdf-sesame/war folder and copy the two .war files to your /var/lib/tomcat7/webapps folder
 - Open the tomcat manager webapp, under Applications navigate to /openrdf-workbench. If you'll get prompted to change the server name, enter http://localhost:8090/openrdf-sesame
 - If this doesn't work, try the following:
 *Symptom:* openrdf-workbench gives an "Invalid Server URL" error when 
 attempting to set the server URL at
 http://localhost:8080/openrdf-workbench/repositories/NONE/server
 *Possible solution:* Ownership of /usr/share/tomcat7 directory may
be wrong.  Try accessing the sesame backend directly, by browsing
to http://localhost:8080/openrdf-sesame/ .   If you get a java
exception with a stack trace that says (among other things):
{{{
java.io.IOException: Unable to create logging directory /usr/share/tomcat7/.aduna/openrdf-sesame/logs
}}}
then check the ownership and change it:
{{{
ls -ld /usr/share/tomcat7
sudo chown -R tomcat7:tomcat7 /usr/share/tomcat7
sudo service apache2 restart
}}}
 - under "Repositories" click on "New repository". Enter this data:
Type: "Native Java Store RDF Schema"
ID: "crime"
Title: "crime"
Click next and then create.
7. Configure Apache to route requests to Jetty (to avoid browser's cross origin policy violation):
 - add these two lines in the apache2.conf file:
    ProxyPass /jetty/ http://localhost:8080/jetty
    ProxyPassReverse / http://localhost:8080/
 - run: "sudo nano /etc/apache/httpd.conf" and add:
<VirtualHost *:80>
     ServerName jetty
     ProxyRequests off
     ProxyPass / http://localhost:8080/jetty/
     ProxyPassReverse / http://localhost:8080/jetty/
</VirtualHost>

8. Download required libraries for ProtoStack
 - for Jetty, I found that this worked fine: http://central.maven.org/maven2/org/eclipse/jetty/jetty-distribution/8.1.3.v20120416/
 - to run Jetty embedded server jars "jetty-all.jar" and "servlet-api.jar" are required, I used these:
 - http://repo1.maven.org/maven2/org/eclipse/jetty/aggregate/jetty-all/7.0.2.v20100331/jetty-all-7.0.2.v20100331.jar and http://repo1.maven.org/maven2/javax/servlet/servlet-api/2.5/servlet-api-2.5.jar
 - OpenRDF Sesame 2.7.13
 - AWS SDK for Java 1.8.11 https://aws.amazon.com/releasenotes/6786824462413999
9. Download Contiki and CRime:
 - Contiki: https://github.com/contiki-os/contiki/releases/tag/2.5-release
 - CRime: https://github.com/sensorlab/CRime
 - I placed the CRime folder inside contiki/core/net/rime/ folder, because the previous code used similar setup
9. Import/Open the Protostack project in a Java IDE (I used Netbeans). Edit RunServer.java, specifically lines 13, 15 and 20 (strings cSrcPath, ontologFNm and outSrcPath) to represent where crime files are located. Check TripleStore.java in lines: 44, 53, 61, 69, 77, 85, 93, 102, 167, 301 and 326 if you placed crime.owl file in a different location (if no changes to the folder structure were made, it should be automatically served on apache server together with the crimeLayers folder at localhost/owl/cfrime.owl). If you are running Tomcat on a port that is different than 8090/8081, check line 113 of TripleStore.java (with newer versions of Sesame, it runs on /openrdf-sesame). The following is not necessary, because this part of code will not be used in your case: open RequstHandler and change line 112 to point to contiki rime examples.
9. If no changes were made to the project (but it probably won't work, because the file paths are different), you can go to ProtoStack/build/classes folder and use the existing build to run the application using the following command: "java -cp .:servlet-api.jar:jetty-l.jar:../../dist/ProtoStack.jar RunServer"
(Packages servlet-api.jar and jetty-l.jar are needed, because this project uses embedded Jetty to provide server functionality.)
i. If changes were made to the project files, build the project first and then run it as described above.


Running Jetty and Apache on same port (apache routes the reqs from port 80 to 8080, where jetty is):
http://stackoverflow.com/questions/12184479/running-multiple-java-jetty-instances-with-same-port-80
