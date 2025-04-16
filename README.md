[![FINOS - Forming](https://cdn.jsdelivr.net/gh/finos/contrib-toolbox@master/images/badge-forming.svg)](https://community.finos.org/docs/governance/Software-Projects/project-lifecycle)

# Flowave Release Parent

Pom which can be inherited for flowave releases defining some common release properties.
It allows to deploy to two repositories simultaneously. One is a Nexus OSS server, the other one a Nexus Enterprise server.
It will deploy the artifacts at the end of the build to keep the window of failure small when talking to external systems.

## Usage

Inherit the flowave-release-parent pom inside your project like so  
  
    <parent>
      <groupId>org.camunda</groupId>
      <artifactId>camunda-release-parent</artifactId>
      <version>${LATEST_VERSION}</version>
      <!-- do not remove empty tag - http://jira.codehaus.org/browse/MNG-4687 -->
     <relativePath />
    </parent>  
    
If you have a multi-module build, just inherit in your parent pom.  

Specify the <scm> section for your project eg.
    
    <scm>
      <url>https://github.com/camunda/MY_PROJECT_URL</url>
      <connection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</connection>
      <developerConnection>scm:git:git@github.com:camunda/MY_PROJECT_URL.git</developerConnection>
    </scm>

## Release

Prerequisite:  

  Add the following to the profiles and servers section of your local settings.xml file. This allows the signing of your artifacts during the release. <strong>Mandatory for maven central</strong>.
  
    <settings>
      ...
      
      <servers>
        ...
        
        <server>
          <id>camunda-nexus</id>
          <username>MY_CAMUNDA_NEXUS_USER</username>
          <password>MY_CAMUNDA_NEXUS_PASSWORD</password>
        </server>
        <server>
          <id>central</id>
          <username>MY_CENTRAL_USER</username>
          <password>MY_CENTRAL_PASSWORD</password>
        </server>
        
      </servers>
      
      
      <profiles>
        ...
        
        <profile>
          <id>sonatype-oss-release</id>
          <properties>
            <gpg.passphrase>YOUR_SECRET_GPG_PASSPHRASE</gpg.passphrase>
          </properties>
        </profile>
        
      </profiles>
    </settings>

To release your own project use the following command:

    mvn \
    release:prepare release:perform \
    -B -Dresume=false -Dtag=1.0.0 -DreleaseVersion=1.0.0 -DdevelopmentVersion=1.1.0-SNAPSHOT \
    -Darguments="--settings=${PATH_TO_YOUR_SETTINGS_XML_FILE}" \
    --settings=${PATH_TO_YOUR_SETTINGS_XML_FILE}
    
This will trigger the `sonatpye-oss-release` profile inside the `camunda-release-parent` pom automatically.
    
## Customization

You can override some default behaviours through the usage of the command line properties below. Add those properties to the `-Darguments="INSERT_PROPERTIES_HERE"` part of the Maven release command.

<table>
  <tr>
    <th>Property</th><th>Description</th>
  </tr>
  <tr>
    <td>nexus.snapshot.repository</td><td>Specify the url to your snapshot repository.<br/>Default is <strong>https://artifacts.camunda.com/artifactory/camunda-bpm-snapshots</strong>.</td>
  </tr>
  <tr>
    <td>nexus.release.repository</td><td>Specify the url to your release repository.<br/>Default is <strong>https://artifacts.camunda.com/artifactory/camunda-bpm</strong>.</td>
  </tr>
  <tr>
    <td>skip.nexus.release</td><td>When setting the value to true, skip the deployment to the release repository specified in <distributionManagement>.<br/>Default is <strong>false</strong>.</td>
  </tr>
  <tr>
    <td>skip.central.release</td><td>When setting the value to true, skip the deployment to maven central.<br/>Default is <strong>false</strong>.</td>
  </tr>
  <tr>
    <td>serverId</td><td>Points to the corresponding entry in your settings.xml like <server><id>yourServerId</id></server> to specify your login credentials.<br/>Default is <strong>central</strong> for maven central.</td>
  </tr>
  <tr>
    <td>nexusUrl</td><td>The plain url which points to your nexus installation.<br/><strong>Must be a Nexus Enterprise Edition</strong><br/>Default is <strong>https://oss.sonatype.org</strong> aka maven central.</td>
  </tr>
</table>

## Testing

You can test your release steps by using the following Maven release command:

    mvn \
    release:prepare release:perform \
    -B -Dresume=false -Dtag=1.0.0 -DreleaseVersion=1.0.0 -DdevelopmentVersion=1.1.0-SNAPSHOT \
    -Darguments="--settings=${PATH_TO_YOUR_SETTINGS_XML_FILE} -Dskip.central.release=true -Dnexus.release.repository=${URL_OF_YOUR_LOCAL_NEXUS_REPOSITORY}" \
    --settings=${PATH_TO_YOUR_SETTINGS_XML_FILE} \
    -DpushChanges=false -DremoteTagging=false -DlocalCheckout=true
    
This will do a release but will not push anything to the involved git repositories and will not deploy to Maven Central. Instead it will deploy to a local Nexus repository. (Hint: You can use a simple Nexus Docker image to do so.)

## Contributing

Have a look at the main Flowave [CONTRIBUTING.md](https://github.com/finos/flowave-bpm-platform/blob/main/CONTRIBUTING.md) file for our general contribution guidelines and governance policies.

## License

Copyright 2025 FINOS

The source files in this repository are made available under the [Apache License Version 2.0](./LICENSE).