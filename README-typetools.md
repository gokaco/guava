This is a version of Guava with additional type annotations for the Checker Framework.

The annotations are only in the main Guava project, not in the "Android" variant that supports JDK 7 and Android.


To build this project
---------------------

Optionally change `guava/pom.xml` to use a locally-built version of the Checker Framework

Create file `guava/target/guava-HEAD-jre-SNAPSHOT.jar`:

```
(cd guava && mvn package -Dmaven.test.skip=true -Danimal.sniffer.skip=true)
```


To update to a newer version of the upstream library
----------------------------------------------------

In the upstream repository, find the commit corresponding to a public release.

Date of release: https://github.com/google/guava/releases
Commits: https://github.com/google/guava/commits/master

Guava version 24.0 is commit 538d60aed09e945f59077770686df9cbd4e0048d
Guava version 24.1 is commit 444ff98e688b384e73d7b599b4168fed8003eb3f

Pull in that commit:
```
git fetch https://github.com/google/guava
git pull https://github.com/google/guava <commitid>
```

Also change pom.xml files that have the most recent release hard-coded.

```
preplace '23\.5' 24.1 `findfile pom.xml` guava/cfMavenCentral.xml
```


To upload to Maven Central
--------------------------

# Ensure the version number is set properly in file guava/cfMavenCentral.xml.
# Then, set this variable to the same version.
PACKAGE=guava-24.1-jre

cd guava


# Compile, and create Javadoc jar file
mvn package -Dmaven.test.skip=true -Danimal.sniffer.skip=true && \
mvn source:jar && \
mvn javadoc:javadoc && (cd target/site/apidocs && jar -cf ${PACKAGE}-javadoc.jar com)
# To get this to work for 24.1, I had to fix syntax errors with periods in serialVersionUID.
# Search for: long serialVersionUID.*\.

## This does not seem to work for me:
# -Dhomedir=/projects/swlab1/checker-framework/hosting-info

mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/${PACKAGE}.jar \
&& \
mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/${PACKAGE}-sources.jar -Dclassifier=sources \
&& \
mvn gpg:sign-and-deploy-file -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=cfMavenCentral.xml -Dgpg.publicKeyring=/projects/swlab1/checker-framework/hosting-info/pubring.gpg -Dgpg.secretKeyring=/projects/swlab1/checker-framework/hosting-info/secring.gpg -Dgpg.keyname=ADF4D638 -Dgpg.passphrase="`cat /projects/swlab1/checker-framework/hosting-info/release-private.password`" -Dfile=target/site/apidocs/${PACKAGE}-javadoc.jar -Dclassifier=javadoc

# Browse to https://oss.sonatype.org/#stagingRepositories to complete the release.