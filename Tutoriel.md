# How to deploy to Maven Central by Travis CI

Dans cet article nous allons découvrir étape par étape comment déployer un projet java dans Maven central via Travis.

Schema:

![alt text](.\schema.jpg "Logo Title Text 1")

## Prérequis

* [Un projet déposé sur Github](github.com)

* [Un Compte Travis](travis-ci.org)

* [Lier Travis avec un projet java](https://github.com/skokaina/devops)

## Les étapes à suivre

* Préparer Sonatype Nexus OSS

* Générer la paire des clés public et privée

* Crypter les clés

* Ajouter les informations nécessaires au pom.xml

* Signer les clés (pub, privé)

* Configurer le fichier Setting.xml

* Configurer le fichier .travis.yml et publish.sh

* Lancer le déploiement

## Préparer Sonatype Nexus OSS

1. Nous allons commencer par la création d'un compte [sonatype](https://issues.sonatype.org/secure/Signup!default.jspa)

2. Après nous allons créer un [Jira Issue](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134),

    pour ce faire il faut fournir les information suivante:

    2.1. Un résumé sur le projet

    2.2. le groupe Id:

    la première partie représente un nom de domaine (ex:ma.mydomaine.myprojectname) si on le possède, on doit fournir une preuve qui le justifie, si non, on peut utiliser des sous noms de domaines par example com.github.mydomaine.myprojectname

    2.3. Le Project URL: c'est l'url de votre projet, par exemple: <https://github.com/sonatype/nexus-oss>

    2.4. Le SCM url: la source de système de contrôle de votre projet, par exemple: <https://github.com/sonatype/nexus-oss.git>,

    puis cliquer sur créer

    2.5. Ensuite, il faut attendre la Issue pour qu'elle soit résolue (NB: ca peut prendre des heures)

## Générer la paire des clés public et privée

1. Tout d'abord, il faut installer l'outil [GNU PG.](https://www.gnupg.org/download/)

    C'est un outil qui permet de crypter et signer les données.

2. Vérifier l'installation avec la commande:

    ``` console
    C:\Users\username>gpg --version
    ```

3. Générer la paire des clés

    ``` console
    C:\Users\username>gpg --full-gen-key

    gpg (GnuPG) 2.1.15; Copyright (C) 2016 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    ```

    L’invite de commande va vous demander de saisir et choisir les informations suivantes:

    * Le type de clé : RSA and RSA

    * La taille de la clé: 2048 bits

    * La durée de validité de la clé: 0 = key does not expire

    * Key does not expire at all Is this correct? (y/N) y

    * Real name: housseine

    * Email address: example@gmail.com

    * Comment: (On peut le laisser vide)

    Après vous pouvez modifier ces informations:

    ``` console
    You selected this USER-ID:

    "housseine <example@gmail.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.

    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    gpg: C:/Users/username/AppData/Roaming/gnupg/trustdb.gpg: trustdb created
    gpg: key 27835B8BD2A2067F marked as ultimately trusted
    gpg: directory 'C:/Users/username/AppData/Roaming/gnupg/openpgp-revocs.d' created
    gpg: revocation certificate stored as 'C:/Users/username/AppData/Roaming/gnupg/openpgp-revocs.d\5694AA563793429557F1727835B8BD2A2067F.rev'
    public and secret key created and signed.
    pub   rsa2048 2016-08-29 [SC]
        5694AA563793429557F1727835B8BD2A2067F
    uid                      Nadeem Mohammad <coolmind182006@gmail.com>
    sub   rsa2048 2016-08-29 [E]

    ```

    Une fenetre popup va s'afficher vous demande de saisir une passphrase: vous devez saisir une passphrase puissante.

4. Afficher La liste des clés :

    ```console
    gpg --list-keys
    ```

    Votre paire de clés doit apparaitre parmi la liste des clés

5. Publier la clé:

    Vous devez publier vos clés dans un ou plusieurs serveur de clés:

    ```console
        gpg2 --keyserver hkp://pool.sks-keyservers.net --send-keys 5694AA563793429557F1727835B8BD2A2067F
    ```

    Voici quelques serveur de clés:

    * pool.sks-keyservers.net
    * gnupg.net:11371
    * keys.pgp.net
    * surfnet.nl
    * mit.edu

6. Exporter les clés public et privée

    ```console
    gpg --export-secret-keys example@gmail.com > private-key.gpg
    gpg --export example@gmail.com > public-key.gpg
    ```

## Crypter les clés

1. Installer OPENSSL

    Pour ce faire on doit avoir l’outil openssl installé

    * Sur mac exécuter la commande:

        ```console
        brew install openssl
        ```

    * Sur windows
    Télécharger et installer l'outil à partir de [ce site](https://wiki.openssl.org/index.php/Binaries)

2. Crypter la paire des clés avec openssl

    ```console
    openssl aes-256-cbc -pass pass: ENCRYPTION_PASSWORD -in ~/.gnupg/private-key.gpg -out private-key.gpg.enc

    openssl aes-256-cbc -pass pass: ENCRYPTION_PASSWORD -in ~/.gnupg/public-key.gpg -out public-key.gpg.enc
    ```

    Maintenant nous allons créer dans la racine de notre projet un dossier deploy dans lequel on va mettre la paire des clés cryptées

## Ajouter les informations nécessaires au pom.xml

NB: votre version de projet peut être snapshot ou release exemple: 

```xml
<version>0.0.1-SNAPSHOT</version>
```
la différence c'est que contrairement au version Release la version Snapshot n'est pas stable.
Dans le fichier pom.xml:

1. Ajoutez la section SCM:

    ```xml
        <scm>
            <url>https://github.com/marocraft/trackntrace</url>
            <connection>scm:git:git://github.com/marocraft/trackntrace.git</connection>
            <developerConnection>scm:git:git@github.com:marocraft/trackntrace.git</developerConnection>
            <tag>tnt-core-0.0.1</tag>
        </scm>
    ```

2. Créez un profile dans lequel mettez toutes les informations relative au déploiement:

    ```xml
        <profiles>
            <profile>
                <id>ossrh</id>
                <activation>
                    <property>
                        <name>ossrh</name>
                    </property>
                </activation>
            </profile>
        </profiles>
    ```

    Dans ce profile:
3. ajoutez le plugin de déploiement:

    ```xml
    <plugin>
        <artifactId>maven-deploy-plugin</artifactId>
        <version>2.8.2</version>
        <executions>
            <execution>
                <id>default-deploy</id>
                <phase>deploy</phase>
                <goals>
                    <goal>deploy</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ```

4. Ajoutez le distribution management:

    ```xml
        <distributionManagement>
            <snapshotRepository>
                <id>ossrh</id>
                <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            </snapshotRepository>
            <repository>
                <id>ossrh</id>
                <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
            </repository>
        </distributionManagement>
    ```

5. Ajoutez le Maven release plugin:

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
        <version>${maven-release-plugin.version}</version>
        <configuration>
            <autoVersionSubmodules>true</autoVersionSubmodules>
            <useReleaseProfile>false</useReleaseProfile>
            <releaseProfiles>ossrh</releaseProfiles>
            <goals>deploy</goals>
        </configuration>
    </plugin>
    ```

6. Ajoutez le Nexus staging Maven plugin:

    ```xml
    <plugin>
        <groupId>org.sonatype.plugins</groupId>
        <artifactId>nexus-staging-maven-plugin</artifactId>
        <version>1.6.8</version>
        <extensions>true</extensions>
        <configuration>
            <serverid>ossrh</serverid>
            <nexusurl>https://oss.sonatype.org/</nexusurl>
            <autoreleaseafterclose>true</autoreleaseafterclose>
        </configuration>
    </plugin>
    ```

7. Add the source and the javadoc plugin

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>3.0.1</version>
        <executions>
            <execution>
                <id>attach-sources</id>
                <goals>
                    <goal>jar</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
        <configuration>
            <javadocExecutable>${env.JAVA_HOME}/bin/javadoc</javadocExecutable>
        </configuration>
        <version>2.10.4</version>
        <executions>
            <execution>
                <id>attach-javadocs</id>
                <goals>
                    <goal>jar</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ```

8. Ajoutez la configuration pour signer l'artifact pendant le release

    ```xml
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-gpg-plugin</artifactId>
        <version>1.5</version>
        <executions>
            <execution>
                <id>sign-artifacts</id>
                <phase>verify</phase>
                <goals>
                    <goal>sign</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ```

    Voir le code sur [github](https://github.com/marocraft/trackntrace/blob/develop/pom.xml)

9. Dans le dossier deploy créer un fichier .maven.xml et remplissez le avec les  détails du serveur ossrh et les données de GPG:

    ```xml
    <settings>
        <servers>
            <server>
                <id>ossrh</id>
                <username>${SONATYPE_USERNAME}</username>
                <password>${SONATYPE_PASSWORD}</password>
            </server>
        </servers>
        <mirrors>
        </mirrors>

        <profiles>
            <profile>
                <id>ossrh</id>
                <activation>
                    <activeByDefault>false</activeByDefault>
                </activation>
                <properties>
                    <gpg.executable>${GPG_EXECUTABLE}</gpg.executable>
                    <gpg.passphrase>${GPG_PASSPHRASE}</gpg.passphrase>
                    <gpg.keyname>${GPG_KEYNAME}</gpg.keyname>
                    <gpg.defaultKeyring>false</gpg.defaultKeyring>
                    <gpg.publicKeyring>./deploy/public-key.gpg</gpg.publicKeyring>
                    <gpg.secretKeyring>./deploy/private-key.gpg</gpg.secretKeyring>
                </properties>
            </profile>
        </profiles>

        <activeProfiles>
            <activeProfile>ossrh</activeProfile>
        </activeProfiles>

    </settings>

    ```

    Après, dans les settings de Travis en doit ajouter les valeurs des variables d'environnement:

    * SONATYPE_USERNAME: identifiant de sonatype
    * SONATYPE_PASSWORD: le mot de passe de sonatype
    * GPG_EXECUTABLE: gpg
    * GPG_PASSPHRASE: la passphrase que vous avez utiliser
    * GPG_KEYNAME: le email ou le nom que vous avez utiliser pour générer les clé via gpg

    voir [ici](https://docs.travis-ci.com/user/environment-variables/)

## configurer les fichiers .travis.yml et le fichier .publish.sh

1. Dans le fichier .travis.yml ajoutez la section suivante:

    ```yml
    after_success:
    - echo "after_success"
    - openssl aes-256-cbc -in $GPG_DIR/private-key.gpg.enc -out $GPG_DIR/private-key.gpg -pass pass:$ENCRYPTION_PASSWORD -d -debug
    - openssl aes-256-cbc -in $GPG_DIR/public-key.gpg.enc -out $GPG_DIR/public-key.gpg -pass pass:$ENCRYPTION_PASSWORD -d -debug

    "$GPG_DIR/publish.sh"

    env:
    global:
        - GPG_DIR="`pwd`/deploy"
    ```

2. Dans le dossier deploy, créer un fichier publish.sh:

    ```sh
    echo "Deploy for tag: $TRAVIS_TAG - branch: $TRAVIS_BRANCH"

    if [ ${TRAVIS_PULL_REQUEST} = 'false' ] && [ ${TRAVIS_BRANCH} = 'snapshot' ]; then
        echo 'Deploying snapshot to OSS in branch ${TRAVIS_BRANCH}'
        mvn clean deploy --settings ./.maven.xml -DskipTests=true -Possrh -X;
        exit $?;
    fi
    ```

## Lancer le deploiment

Mainetenant switchez vers une branche "snapshot" et commitez et pushez votre projet, et voilà normalement si le build passe bien, vous devez le voir dans [Sonatype OSS]( https://oss.sonatype.org)

### Author: Tassa housseine
