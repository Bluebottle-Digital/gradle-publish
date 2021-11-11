# gradle-publish
Project to guide how to publish your library to Maven repository using gradle via Sonatype OSSHR.
Sonatype OSSRH (OSS Repository Hosting) uses Sonatype Nexus Repository Manager to provide repository hosting service for open source project binaries.

**1. Create your JIRA account** ( Sonatype use JIRA to manage requests)
  - Go to https://issues.sonatype.org/secure/Dashboard.jspa and create an account.
  - Create new project ticket
    * Project type is: Community Support - Open Source Project Repository Hosting
    * If you want to register your business Group, for example bluebottle.digital, you must prove ownership of the domain by adding a TXT record to your DNS ( they     will send information via your registered email)
    * You can use common public group if do not have any real domain ( e.g io.github.bluebottle)
    
  - Wait until a Sonatype admin resolve this ticket, and new repository will be created. This can be several hours or up to 2 business days.
  - After ticket was resolved, you can publish snapshot and release artifacts to http://s01.oss.sonatype.org/
  - After several days they will send email to inform that your Central sync is activated and after you successfully release, your component will be published to     Central https://repo1.maven.org/maven2/, typically within 30 minutes, though updates to https://search.maven.org can take up to four hours


**2. Create a public/private key pair to sign files**
  - Install gnupg:
     * For mac via homebrew: **_brew install gnupg_**
  	 * For ubuntu: **_sudo apt-get install gnupg_**   
  - Generate key: **_gpg --gen-key_** (you must save the passphrase you input somewhere to use later)
  - List keys: **_gpg --list-keys_**
  - Distribute to key server: **_gpg --keyserver SERVER --send-keys YOUR-KEY_**
    * Common used SERVER: keyserver.ubuntu.com, keys.openpgp.org, pgp.mit.edu
  - Export keys to use in another machine, note that passphrases is required: **_gpg --export-secret-keys > private.key_**
  - Import keys in another machine: **_gpg --import private.key_** >> input your passphrases
  
 **3. Export your secret keys to a secring.gpg file**
  - Export: **_gpg --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg_**
  
 **4. Add signing info to gradle.properties file**
 
 Below is sample content in gradle.properties file that contain your signing info and your Sonatype credentials:
 ``` 
 signing.keyId=LAST-8-DIGITS-IN-YOUR-PUBLIC-KEY
 signing.password=YOUR-PASSPHRASES
 signing.secretKeyRingFile=~/.gnupg/secring.gpg
 sonatypeUsername=SONATYPE-USERNAME
 sonatypePassword=SONATYPE-PASSWORD
 ```

**5. Config file build.gradle in your project as below example**
```
plugins {
    id 'java-library'
    id 'maven-publish'
    id 'signing'
}

java {
    withJavadocJar()
    withSourcesJar()
}

group 'io.github.supertidus'
version '1.0-SNAPSHOT'

ext.isReleaseVersion = !version.endsWith("SNAPSHOT")

// setup the signing
signing {
    sign configurations.archives
}

publishing {
    repositories {
        maven {
            def releaseRepo = "https://s01.oss.sonatype.org/content/repositories/releases/"
            def stagingRepo = "https://s01.oss.sonatype.org/content/repositories/staging/"
            def snapshotRepo = "https://s01.oss.sonatype.org/content/repositories/snapshots/"
            url = isReleaseVersion ? releaseRepo : snapshotRepo

            credentials {
                username = sonatypeUsername
                password = sonatypePassword
            }
        }
    }

    publications {
        mavenJava(MavenPublication) {
            pom {
                groupId = 'io.github.bluebottle'
                name = 'OSSRH publishing'
                description = 'Example Project to learn how to deploy to OSSRH'
                url = 'https://github.com/Bluebottle-Digital/gradle-publish'
                from components.java
                licenses {
                    license {
                        name = 'The Apache License, Version 2.0'
                        url = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                scm {
                    connection = 'https://github.com/Bluebottle-Digital/gradle-publish.git'
                    developerConnection = 'https://github.com/Bluebottle-Digital/gradle-publish'
                    url = 'https://github.com/Bluebottle-Digital/gradle-publish'
                }
            }
        }
    }
}
```
