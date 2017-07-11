# HQIT Maven Repository

## setup ``~/.m2/settings.xml``, add server info
if *Two-factor Auth* enabled, use access token instead of password.
```
<?xml version="1.0" encoding="UTF-8"?>
<settings 
    xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="ht
tp://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>github</id>
            <username>{YOUR-GITHUB-USERNAME}</username>
            <password>{GITHUB-ACCESS-TOKEN or PASSWORD}</password>
        </server>
    </servers>
</settings>
```

## maven project deploy to this repo
1. in project **pom.xml**, set **project.url**
  a. plugins will use this url find the right url
```
<url>https://github.com/HQIT/maven-repo</url>
```
  a. set the github auth account, **MUST** be exactly same as the one in ``~/.m2/settings.xml``
```
<properties>
  <github.global.server>github</github.global.server>
</properties>
```
1. add ``distributionManagement`` node, create a new local repository in build target dir
```
<distributionManagement>
  <repository>
    <id>internal.repo</id>
    <name>Temporary Staging Repository</name>
    <url>file://${project.build.directory}/mvn-repo</url>
  </repository>
</distributionManagement>
```
2. setup plugin deploy to the **local repository**
```
<plugins>
  ...
  <plugin>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>2.8.1</version>
    <configuration>
      <altDeploymentRepository>internal.repo::default::file://${project.build.directory}/mvn-repo</altDeploymentRepository>
    </configuration>
  </plugin>
  ...
</plugins>
```
3. setup plugin to upload targets to github
**IMPORTANT** ~~use project name as *branch* name~~, set **merge** to *true*, keep multi-projects separated in one github repository (e.g **hqit-maven-repo**)
```
<plugin>
  <groupId>com.github.github</groupId>
  <artifactId>site-maven-plugin</artifactId>
  <version>0.12</version>
  <configuration>
    <message>Maven artifacts for ${project.version}</message> <!-- git commit message -->
    <noJekyll>true</noJekyll> <!-- disable webpage processing -->
    <outputDirectory>${project.build.directory}/mvn-repo</outputDirectory> <!-- matches distribution management repository url above -->
    <branch>refs/heads/master</branch> <!-- remote branch name -->
    <merge>true</merge>
    <includes>
      <include>**/*</include>
    </includes>
  </configuration>
  <executions>
    <!-- run site-maven-plugin's 'site' target as part of the build's normal 'deploy' phase -->
    <execution>
      <goals>
        <goal>site</goal>
      </goals>
      <phase>deploy</phase>
    </execution>
  </executions>
</plugin>
```
## add exists jar(and metadata files)
1. mkdir a empty dir like *local-repo* etc
2. cd into the new created dir
3. copy in jars and metadata files
include the full package dirs:
```
com/smartunit/tcf-tools/
```
inside the last level dir, should be looks like:
```
0.1.3-RELEASE/
maven-metadata.xml
maven-metadata.xml.md5
maven-metadata.xml.sha1
```
*0.1.3-RELEASE* indicated the version and build type, jars are included:
```
tcf-tools-0.1.3-RELEASE.jar
tcf-tools-0.1.3-RELEASE.jar.md5
tcf-tools-0.1.3-RELEASE.jar.sha1
tcf-tools-0.1.3-RELEASE.pom
tcf-tools-0.1.3-RELEASE.pom.md5
tcf-tools-0.1.3-RELEASE.pom.sha1
```
4. push to github **unique-named** branch (*tcf-tools* etc)
```
git add *
git commit -m "update ...."
git remote add origin {github repo url}
git push git push origin master:{**unique-named** branch}
```
## use the github maven repo
1. add *repository* in pom.xml of project
```
<repositories>
  <repository>
    <id>hqit-maven-repo-commons-java</id> <!-- must be unique -->
    <url>https://raw.github.com/{github-repo-name(may-include-org-name)}/{branch-name}</url>
    <snapshots>
      <enabled>true</enabled>
      <updatePolicy>always</updatePolicy>
    </snapshots>
  </repository>
</repositories>
```
**IMPORTANT** if merged into master, **url** should use master as {branch-name}
2. normal *dependency* setting
