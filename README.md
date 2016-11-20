### Purpose

This project is created as a PoC of deploying a Spring Boot app with a Maven set up to a private repo on GCP as created by https://github.com/renaudcerrato/appengine-maven-repository

It was created by selecting a Maven project from http://start.spring.io/ (it's bare bones without any additional dependencies added)

This has only been tested with Maven 3.3.9 installed via Homebrew on OS X

### Maven Configuration

To enable usage of the new private repo for deploying snapshots, see the `distributionManagement` configuration in the [pom.xml](pom.xml). It is the standard way of dealing with remote Maven repositories as described [here](http://maven.apache.org/plugins/maven-deploy-plugin/usage.html)

The private GCP project id forms part of the repository url (i.e. https://${gcp-project-id}.appspot.com) and is handled via an externalised property in `~/.m2/settings.xml`

```maven
<profiles>
  <profile>
    <id>gcp-properties</id>
    <properties>
      <gcp-project-id>my-maven-repo</gcp-project-id>
    </properties>
  </profile>
</profiles>

<activeProfiles>
  <activeProfile>gcp-properties</activeProfile>
</activeProfiles>
```

where the GCP project id corresponds to

![](http://i.imgur.com/iSt98wWl.png)

To complete the set up the `server` configuration also needs to be added in `~/.m2/settings.xml`. 

> If you do not add the additional server config below you will see a build failure when you try to `mvn deploy` with this message  `org.apache.maven.wagon.providers.http.httpclient.impl.auth.HttpAuthenticator handleAuthChallenge
WARNING: Malformed challenge: Authentication challenge is empty`

Various workarounds for the `WARNING: Malformed challenge` were considered and attempted which are best described [here on Stack Overflow](http://stackoverflow.com/questions/1280747/accessing-an-artifactory-maven-repo-that-requires-basic-auth) and [here on MyMavenRepo.com](https://mymavenrepo.com/docs/maven.auth.html). The only working solution so far coming from [this answer on StackOverflow](http://stackoverflow.com/a/10985349/752167)

It is necessary to configure Basic Auth in the `httpHeaders` section within the `server` config below. It matches the config of the private repo from the [users.txt](https://github.com/renaudcerrato/appengine-maven-repository/blob/master/src/main/webapp/WEB-INF/users.txt) file where the credentials should be changed during the install of the private repo to GCP.

```maven
<server>
    <id>gcp-snapshots.write</id>
    <username>admin</username>
	<password>admin</password>
    <configuration>
        <httpHeaders>
            <property>
                <name>Authorization</name>
                <!-- Base64-encoded "admin:admin" can be encoded online at https://www.base64encode.org/ -->
                <value>Basic YWRtaW46YWRtaW4=</value>
            </property>
        </httpHeaders>
    </configuration>
</server>
```

With all the config now in place, deploying snapshot artifacts to your private repository can be done via

```bash
$ mvn deploy
```