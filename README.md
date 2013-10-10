## Template for Jenkins Jobs for PHP Projects

Most web applications are changed and adapted quite frequently and quickly. Their environment, for example the size and the behaviour of the user base, are constantly changing. What was sufficient yesterday can be insufficient today. Especially in a web environment it is important to monitor and continuously improve the internal quality not only when developing, but also when maintaining the software.

[Jenkins](http://jenkins-ci.org/) is the leading [open-source](http://en.wikipedia.org/wiki/Open_Source) [continuous integration](http://martinfowler.com/articles/continuousIntegration.html) server. Thanks to its thriving plugin ecosystem, it supports building and testing virtually any project.

The goal of this project is to provide a standard template for Jenkins jobs for PHP projects.


## Required Jenkins Plugins

You need to install the following plugins for Jenkins:

*   [Checkstyle](http://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin) (for processing [PHP_CodeSniffer](http://pear.php.net/PHP_CodeSniffer) logfiles in Checkstyle format)

*   [Clover PHP](http://wiki.jenkins-ci.org/display/JENKINS/Clover+PHP+Plugin) (for processing [PHPUnit](http://www.phpunit.de/) code coverage xml output)

*   [DRY](http://wiki.jenkins-ci.org/display/JENKINS/DRY+Plugin) (for processing [phpcpd](https://github.com/sebastianbergmann/phpcpd) logfiles in PMD-CPD format)

*   [HTML Publisher](http://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin) (for publishing the [PHPUnit](http://www.phpunit.de/) code coverage report, for instance)

*   [JDepend](http://wiki.jenkins-ci.org/display/JENKINS/JDepend+Plugin) (for processing [PHP_Depend](http://pdepend.org/) logfiles in JDepend format)

*   [Plot](http://wiki.jenkins-ci.org/display/JENKINS/Plot+Plugin) (for processing [phploc](https://github.com/sebastianbergmann/phploc) CSV output)

*   [PMD](http://wiki.jenkins-ci.org/display/JENKINS/PMD+Plugin) (for processing [PHPMD](http://phpmd.org/) logfiles in PMD format)

*   [Violations](http://wiki.jenkins-ci.org/display/JENKINS/Violations) (for processing various logfiles)

*   [xUnit](http://wiki.jenkins-ci.org/display/JENKINS/xUnit+Plugin) (for processing [PHPUnit](http://www.phpunit.de/) logfiles in JUnit format)

You can install these plugins using the web frontend at 
`http://localhost:8080/pluginManager/available` or using the [Jenkins CLI](http://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI):
     ```wget http://localhost:8080/jnlpJars/jenkins-cli.jar

java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin \
checkstyle cloverphp dry htmlpublisher jdepend plot pmd violations xunit

java -jar jenkins-cli.jar -s http://localhost:8080 safe-restart```

In the above, replace `localhost:8080` with the hostname and port of your Jenkins installation.


## Required PHP Tools

The [required PHP tools](http://pear.phpqatools.org/) should be installed using the PEAR Installer, the backbone of the [PHP Extension and Application Repository](http://pear.php.net/) that provides a distribution system for PHP packages.

Depending on your OS distribution and/or your PHP environment, you may need to install PEAR or update your existing PEAR installation before you can proceed with the following instructions. `sudo pear upgrade PEAR` usually suffices to upgrade an existing PEAR installation. The [PEAR Manual](http://pear.php.net/manual/en/installation.getting.php) explains how to perform a fresh installation of PEAR.

The following two commands (which you may have to run as `root`) are all that is required to install the required PHP tools using the PEAR Installer:

     ```pear config-set auto_discover 1
pear install pear.phpqatools.org/phpqatools pear.netpirates.net/phpDox```
   
## Build Automation

[Apache Ant](http://ant.apache.org/) is required for automating the build process.

		 It orchestrates the execution of the various tools in the `build.xml` build script ([download](download/build.xml)).

		 The build script assumes that the rule sets for PHP_CodeSniffer and PHPMD are located at `build/phpcs.xml` and `build/phpmd.xml`.

  ```
  <?xml version="1.0" encoding="UTF-8"?>

<project name="name-of-project" default="build">
 <target name="build"
   depends="prepare,lint,phploc,pdepend,phpmd-ci,phpcs-ci,phpcpd,phpdox,phpunit,phpcb"/>

 <target name="build-parallel"
   depends="prepare,lint,tools-parallel,phpunit,phpcb"/>

 <target name="tools-parallel" description="Run tools in parallel">
  <parallel threadCount="2">
   <sequential>
    <antcall target="pdepend"/>
    <antcall target="phpmd-ci"/>
   </sequential>
   <antcall target="phpcpd"/>
   <antcall target="phpcs-ci"/>
   <antcall target="phploc"/>
   <antcall target="phpdox"/>
  </parallel>
 </target>

 <target name="clean" description="Cleanup build artifacts">
  <delete dir="${basedir}/build/api"/>
  <delete dir="${basedir}/build/code-browser"/>
  <delete dir="${basedir}/build/coverage"/>
  <delete dir="${basedir}/build/logs"/>
  <delete dir="${basedir}/build/pdepend"/>
 </target>

 <target name="prepare" depends="clean" description="Prepare for build">
  <mkdir dir="${basedir}/build/api"/>
  <mkdir dir="${basedir}/build/code-browser"/>
  <mkdir dir="${basedir}/build/coverage"/>
  <mkdir dir="${basedir}/build/logs"/>
  <mkdir dir="${basedir}/build/pdepend"/>
  <mkdir dir="${basedir}/build/phpdox"/>
 </target>

 <target name="lint" description="Perform syntax check of sourcecode files">
  <apply executable="php" failonerror="true">
   <arg value="-l" />

   <fileset dir="${basedir}/src">
    <include name="**/*.php" />
    <modified />
   </fileset>

   <fileset dir="${basedir}/tests">
    <include name="**/*.php" />
    <modified />
   </fileset>
  </apply>
 </target>

 <target name="phploc" description="Measure project size using PHPLOC">
  <exec executable="phploc">
   <arg value="--log-csv" />
   <arg value="${basedir}/build/logs/phploc.csv" />
   <arg path="${basedir}/src" />
  </exec>
 </target>

 <target name="pdepend" description="Calculate software metrics using PHP_Depend">
  <exec executable="pdepend">
   <arg value="--jdepend-xml=${basedir}/build/logs/jdepend.xml" />
   <arg value="--jdepend-chart=${basedir}/build/pdepend/dependencies.svg" />
   <arg value="--overview-pyramid=${basedir}/build/pdepend/overview-pyramid.svg" />
   <arg path="${basedir}/src" />
  </exec>
 </target>

 <target name="phpmd"
         description="Perform project mess detection using PHPMD and print human readable output. Intended for usage on the command line before committing.">
  <exec executable="phpmd">
   <arg path="${basedir}/src" />
   <arg value="text" />
   <arg value="${basedir}/build/phpmd.xml" />
  </exec>
 </target>

 <target name="phpmd-ci" description="Perform project mess detection using PHPMD creating a log file for the continuous integration server">
  <exec executable="phpmd">
   <arg path="${basedir}/src" />
   <arg value="xml" />
   <arg value="${basedir}/build/phpmd.xml" />
   <arg value="--reportfile" />
   <arg value="${basedir}/build/logs/pmd.xml" />
  </exec>
 </target>

 <target name="phpcs"
         description="Find coding standard violations using PHP_CodeSniffer and print human readable output. Intended for usage on the command line before committing.">
  <exec executable="phpcs">
   <arg value="--standard=${basedir}/build/phpcs.xml" />
   <arg path="${basedir}/src" />
  </exec>
 </target>

 <target name="phpcs-ci" description="Find coding standard violations using PHP_CodeSniffer creating a log file for the continuous integration server">
  <exec executable="phpcs" output="/dev/null">
   <arg value="--report=checkstyle" />
   <arg value="--report-file=${basedir}/build/logs/checkstyle.xml" />
   <arg value="--standard=${basedir}/build/phpcs.xml" />
   <arg path="${basedir}/src" />
  </exec>
 </target>

 <target name="phpcpd" description="Find duplicate code using PHPCPD">
  <exec executable="phpcpd">
   <arg value="--log-pmd" />
   <arg value="${basedir}/build/logs/pmd-cpd.xml" />
   <arg path="${basedir}/src" />
  </exec>
 </target>

 <target name="phpdox" description="Generate API documentation using phpDox">
  <exec executable="phpdox"/>
 </target>

 <target name="phpunit" description="Run unit tests with PHPUnit">
  <exec executable="phpunit" failonerror="true"/>
 </target>

 <target name="phpcb" description="Aggregate tool output with PHP_CodeBrowser">
  <exec executable="phpcb">
   <arg value="--log" />
   <arg path="${basedir}/build/logs" />
   <arg value="--source" />
   <arg path="${basedir}/src" />
   <arg value="--output" />
   <arg path="${basedir}/build/code-browser" />
  </exec>
 </target>
</project>
  ```

Here is an overview of the tasks defined in the above `build.xml` ([download](download/build.xml)) script that are intended to be directly invoked:

*   `build` is the default build target. It invokes all the tools in such a way that XML logfiles are written to default locations.

*   `build-parallel` is the same as `build` (see above) except that it executes the tools in parallel.

*   `clean` can be used to clean up (delete) all build artifacts (logfiles, etc. that are produced during the build).

*   `lint` can be used to perform a syntax check of the project sources using `php -l`.

*   `phpdox` can be used to generate API documentation.

*   `phpcs` can be used to find coding standard violations and print human readable output. This task is intended for usage on the command line before committing.

*   `phploc` can be used to measure the project size.

*   `phpmd` can be used to perform project mess detection and print human readable output. This task is intended for usage on the command line before committing.

*   `phpunit` can be used to run unit tests.

*   `phpcb` can be used to generate a browsable representation of the PHP code.

The other tasks can, of course, also be invoked directly but that is not their intended purpose. They are invoked by the tasks listed above.

## PHPUnit

The `phpunit` task in the `build.xml` [above](#build_automation) assumes that an XML configuration file for PHPUnit is used to configure the following logging targets:

You can [download](download/phpunit.xml.dist) a sample `phpunit.xml.dist` and place it in your project root to get started.

More information can be found in the [documentation](http://www.phpunit.de/manual/current/en/appendixes.configuration.html) for PHPUnit.

## phpDox

The `phpdox` task in the `build.xml` [above](#build_automation) assumes that an XML configuration file for phpDox is used to configure the API documentation generation:

More information can be found in the [documentation](http://phpdox.de/getting-started.html) for phpDox.

## PHP_CodeSniffer

The `phpcs` and `phpcs-ci` tasks in the `build.xml` [above](#build_automation) assume that an XML configuration file for PHP_CodeSniffer is used to configure the coding standard:

The build script assumes that the rule sets for PHP_CodeSniffer is located at `build/phpcs.xml`.

More information can be found in the [documentation](http://pear.php.net/manual/en/package.php.php-codesniffer.annotated-ruleset.php) for PHP_CodeSniffer.


## PHPMD

The `phpmd` and `phpmd-ci` tasks in the `build.xml` [above](#build_automation) assume that an XML configuration file for PHPMD is used to configure the coding standard:


The build script assumes that the rule sets for PHPMD is located at `build/phpmd.xml`.

More information can be found in the [documentation](http://phpmd.org/documentation/creating-a-ruleset.html) for PHPMD.


## Build Artifacts

Executing the `build.xml` script [above](#build_automation) will produce the following `build` directory:

     ```build
|-- api ...
|-- code-browser ...
|-- coverage ...
`-- logs
    |-- checkstyle.xml
    |-- clover.xml
    |-- jdepend.xml
    |-- junit.xml
    |-- phploc.csv
    |-- pmd-cpd.xml
    `-- pmd.xml```

These build artifacts will be processed by Jenkins.

## Using the Job Template

1.  Fetch the jenkins-cli.jar from your jenkins server:

          <pre>wget http://localhost:8080/jnlpJars/jenkins-cli.jar</pre>
2.  Download and install the job template:

          ```curl https://raw.github.com/sebastianbergmann/php-jenkins-template/master/config.xml | \
    java -jar jenkins-cli.jar -s http://localhost:8080 create-job php-template```

    **or** add the template manually:

          ```cd $JENKINS_HOME/jobs
mkdir php-template
cd php-template
wget https://raw.github.com/sebastianbergmann/php-jenkins-template/master/config.xml
cd ..
chown -R jenkins:jenkins php-template/```
3.  Reload Jenkins' configuration, for instance using the Jenkins CLI:

          ```java -jar jenkins-cli.jar -s http://localhost:8080 reload-configuration```
4.  Click on "New Job".

5.  Enter a "Job name".

6.  Select "Copy existing job" and enter "php-template" into the "Copy from" field.

7.  Click "OK".

8.  Disable the "Disable Build" option.

9.  Fill in your "Source Code Management" information.

10.  Configure a "Build Trigger", for instance "Poll SCM".

11.  Click "Save".


## Troubleshooting

### No "Copy existing job" option / "php-template" project does not show up

Jenkins cannot find the `php-template` job. Make sure you cloned to the right directory. Check the permissions to make sure the Jenkins user has read and write access to the directory. Restart Jenkins.

### General Setup Issues

Check the management panel `http://localhost:8080/manage` and the [Jenkins log](http://wiki.jenkins-ci.org/display/JENKINS/Logging) for messages.


## Further Reading

_[Integrate Your PHP Project with Jenkins](http://oreilly.com/catalog/0636920021353/)_ is a book that explains how you can leverage Jenkins to monitor the various aspects of software quality in a PHP software project.

The planning, execution, and automation of tests for the different layers and tiers of a PHP-based web application is also outside the scope of the aforementioned book. _[Real-World Solutions for Developing High-Quality PHP Frameworks and Applications](http://phpqabook.com/)_ is an excellent resource on these matters.


## Support

[Issues are tracked on GitHub](https://github.com/sebastianbergmann/php-jenkins-template/issues).

The [#jenkins-php](irc://irc.freenode.net/#jenkins-php) channel on the [FreeNode](http://freenode.net) IRC network can be used to discuss all issues related to Jenkins and PHP.

[thePHP.cc](http://thePHP.cc/ "The PHP Consulting Company (thePHP.cc)") offers consulting and training that set you on a path to create, maintain and extend sustainable software of high quality with PHP and leverage Jenkins to monitor the various aspects of software quality.

* * *
