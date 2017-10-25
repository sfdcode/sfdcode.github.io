---
title: "Apex Static Code Analysis With PMD"
categories:
  - Apex
tags:
  - apex
  - pmd
  - code analysis
  - code analysis tool
---
<p align="center">
  <img src="/assets/images/pmd_apex.png">
</p>
In this article we are going to make a static code review for salesforce Apex code using the <a href="https://pmd.github.io/" target="_blank">PMD</a> static code analyzer.  It finds common programming flaws like unused variables, empty catch blocks, unnecessary object creation, and so forth. Additionally it includes CPD, the copy-paste-detector.

It will allow us to have a better quality and avoid maintenance, performance and bug problems in our Apex code. Let's do it.

## Extract the code from the org to a local directory
PMD anlyzes files in directories, then the first thing is to extract the code from our salesforce org to a local directory, to do that we're going to use the <a href="https://developer.salesforce.com/docs/atlas.en-us.daas.meta/daas/forcemigrationtool_install.htm" target="_blank">force.com migration tool</a>. You can find in the link how to install it in your local system, take care with the <a href="https://developer.salesforce.com/docs/atlas.en-us.daas.meta/daas/forcemigrationtool_prereq.htm" target="_blank">requirements</a>. Basically you need to install Java 1.7.x or higher (better Java 8 to avoid additional configurations) and ant 1.6 or later.

Once you have installed the requirements, you are able to see the following outputs in your terminal:

{% highlight bash %}
$ java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

$ ant -version
Apache Ant(TM) version 1.9.7 compiled on April 9 2016
{% endhighlight %}

Download the force.com migration tool:
<p align="center">
  <img src="/assets/images/force_migration_link_screenshot.jpg">
</p>
Extract the zip file salesforce_ant_&lt;version&gt;.zip
<p align="center">
  <img src="/assets/images/force_migration_directory.jpg">
</p>
Copy or rename the sample directory to make your configuration. Then edit the build.properties and provide the parameters to connect to your salesforce org. For sf.password provide the password concatenate with the security token, you can generate it from your settings:
<p align="center">
  <img src="/assets/images/reset_my_security_token_screenshot.jpg">
</p>
I will send you a mail with the security token, then the build.properties files should look like bellow:
{% highlight properties %}
# build.properties
#

# Specify the login credentials for the desired Salesforce organization
sf.username = user@domain.com
sf.password = mypassword5Va3NFGFaTplKilAmKeaW3bX3
#sf.sessionId = <Insert your Salesforce session id here.  Use this or username/password above.  Cannot use both>
#sf.pkgName = <Insert comma separated package names to be retrieved>
#sf.zipFile = <Insert path of the zipfile to be retrieved>
#sf.metadataType = <Insert metadata type name for which listMetadata or bulkRetrieve operations are to be performed>

# Use 'https://login.salesforce.com' for production or developer edition (the default if not specified).
# Use 'https://test.salesforce.com for sandbox.
sf.serverurl = https://login.salesforce.com

sf.maxPoll = 20
# If your network requires an HTTP proxy, see http://ant.apache.org/manual/proxy.html for configuration.
#
{% endhighlight %}

Then the next thing is to select the things that we want to extract from the salesforce org. It is specified in a file named package.xml. Go to unpackaged sub-directory and edit the package.xml file to provide the Apex metadata elements:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>*</members>
        <name>ApexClass</name>
    </types>
    <types>
        <members>*</members>
        <name>ApexTrigger</name>
    </types>
    <version>41.0</version>
</Package>
{% endhighlight %}

Now we have the environment configured to extract the Apex metadata related from the org. from your project directory execute the retrieveUnpackaged ant target:
{% highlight bash %}
~/salesforce_ant_41.0/myorg$ ls -1
build.properties
build.xml
codepkg
mypkg
removecodepkg
retrieveUnpackaged
unpackaged


~/salesforce_ant_41.0/myorg$ ant retrieveUnpackaged
Buildfile: /salesforce_ant_41.0/myorg/build.xml

retrieveUnpackaged:
[sf:retrieve] Request for a retrieve submitted successfully.
[sf:retrieve] Request ID for the current retrieve task: 09S1I000003jm4TUAQ
[sf:retrieve] Waiting for server to finish processing the request...
[sf:retrieve] Request Status: Succeeded
[sf:retrieve] Finished request 09S1I000003jm4TUAQ successfully.

BUILD SUCCESSFUL
Total time: 3 seconds
{% endhighlight %}

Now we have in the retrieveUnpackaged directory of the project folder the code for the Apex classes and triggers.
{% highlight bash %}
~/salesforce_ant_41.0/myorg$ cd retrieveUnpackaged
~/salesforce_ant_41.0/myorg/retrieveUnpackaged$ ls -lRa
total 8
drwxr-xr-x  4 user  wheel  136 25 Oct 11:42 .
drwxr-xr-x@ 9 user  wheel  306 25 Oct 11:30 ..
drwxr-xr-x  8 user  wheel  272 25 Oct 11:42 classes
-rw-r--r--  1 user  wheel  308 25 Oct 11:45 package.xml

./classes:
total 48
drwxr-xr-x  8 user  wheel  272 25 Oct 11:42 .
drwxr-xr-x  4 user  wheel  136 25 Oct 11:42 ..
-rw-r--r--  1 user  wheel  668 25 Oct 11:45 Class1.cls
-rw-r--r--  1 user  wheel  174 25 Oct 11:45 Class1.cls-meta.xml
-rw-r--r--  1 user  wheel  668 25 Oct 11:45 Class2.cls
-rw-r--r--  1 user  wheel  174 25 Oct 11:45 Class2.cls-meta.xml
-rw-r--r--  1 user  wheel  816 25 Oct 11:45 Class3.cls
-rw-r--r--  1 user  wheel  174 25 Oct 11:45 Class3.cls-meta.xml
{% endhighlight %}

## Install PMD tool
PMD installation is just to uncompress the zip distribution file, choose the binary one, at the moment of this article the last release is 5.8.1-SNAPSHOT, but this is a pre-release available 6.0.0-SNAPSHOT, check in <a href="https://sourceforge.net/projects/pmd/files/pmd/" target="_blank">sourceforge pmd page</a> to install the last one.
{% highlight bash %}
~/pmd-bin-6.0.0-SNAPSHOT$ ls -la
drwxr--r--+ 302 user  wheel  10268 25 Oct 12:05 ..
-rw-r--r--@   1 user  wheel  13190 21 Oct 17:34 LICENSE
drwxr-xr-x@   8 user  wheel    272 25 Oct 12:05 bin
drwxr-xr-x@  52 user  wheel   1768 25 Oct 12:05 lib
{% endhighlight %}


## Analyze the code with PMD
There are different ways to do the analysis, it is possible to use the ```pmd.bat / run.sh pmd``` command located in the ```bin``` directory of the PMD installation. But we're going to use ant to run the tasks for PMD and CPD (PMD Copy/Paster Detector). Two files are necessary ```build.xml```

{% highlight xml %}
<?xml version="1.0"?>
<project xmlns='antlib:org.apache.tools.ant' basedir=".">
  <property file="build.properties"/>
  <path id="pmd.classpath">
    <fileset dir="${pmd.dir}/lib">
      <include name="**/*.jar" />
    </fileset>
  </path>

  <taskdef name="pmd" classname="net.sourceforge.pmd.ant.PMDTask" classpathref="pmd.classpath" />
  <taskdef name="cpd" classname="net.sourceforge.pmd.cpd.CPDTask" classpathref="pmd.classpath" />

  <target name="pmd">
    <pmd shortFilenames="true" >
      <formatter type="${pmd.format}" toFile="outputPMD.${pmd.format}" />
      <ruleset>rulesets/apex/complexity.xml</ruleset>
      <ruleset>rulesets/apex/performance.xml</ruleset>
      <ruleset>rulesets/apex/style.xml</ruleset>
      <ruleset>rulesets/apex/braces.xml</ruleset>
      <ruleset>rulesets/apex/apexunit.xml</ruleset>
      <ruleset>rulesets/apex/security.xml</ruleset>
      <ruleset>rulesets/apex/complexity.xml</ruleset>
      <ruleset>rulesets/apex/empty.xml</ruleset>
      <ruleset>rulesets/apex/metrics.xml</ruleset>
      <ruleset>rulesets/apex/ruleset.xml</ruleset>
      <fileset dir="${src.dir}">
        <include name="**/*.cls"/>
        <include name="**/*.trigger"/>
      </fileset>
    </pmd>
  </target>

  <target name="cpd">
      <cpd minimumTokenCount="${cpd.minimumTokenCount}" language="apex" format="${cpd.format}" outputFile="outputCPD.${cpd.format}" encoding="UTF-8" ignoreLiterals="true">
          <fileset dir="${src.dir}">
              <include name="classes/*.cls"/>
          </fileset>
      </cpd>
  </target>

  <target name="all" depends="cpd,pmd"></target>
</project>
{% endhighlight %}

and the ```build.properties``` where we are going to define several properties (format, pmd dir...)
{% highlight properties %}
pmd.dir=/pmdinstallationdir/pmd-bin-6.0.0-SNAPSHOT
#src.dir = <path to the source code>
src.dir=../salesforce_ant_41.0/myorg/retrieveUnpackaged
#pmd.format = html, text, xml
pmd.format=html
#cpd.format = csv, text, xml
cpd.format=xml
# cpd minimum duplicate size 
cpd.minimumTokenCount=10
{% endhighlight %}

Once we have our files configured adapted to our environment, we can run the ant all to perform the analysis:
{% highlight bash %}
$ ant all
Buildfile: /Users/mbravosanz/Downloads/pmdanalysis/build.xml

cpd:
      [cpd] Starting run, minimumTokenCount is 10
      [cpd] Tokenizing files
      [cpd] Starting to analyze code
      [cpd] Done analyzing code; that took 12 milliseconds
      [cpd] Generating report

pmd:
      [pmd] Oct 25, 2017 3:53:52 PM net.sourceforge.pmd.cache.NoopAnalysisCache <init>
      [pmd] WARNING: This analysis could be faster, please consider using Incremental Analysis: https://pmd.github.io/pmd/pmd_userdocs_getting_started.html#incremenal-analysis

all:

BUILD SUCCESSFUL
Total time: 6 seconds
{% endhighlight %}

The results can be generated in different formats, and you can integrate this tasks in your force.com migration tool ```build.xml``` to perform a code analysis every time you do a deployment request over a sandbox.

All the source code of this post is in this [GitHub repository](https://github.com/sfdcode/apex-pmd-build.git)