# iSantePlus Package Repository

See https://docs.github.com/en/free-pro-team@latest/packages/guides/configuring-apache-maven-for-use-with-github-packages

**Quick note about maven profiles**

To provide the correct setup for deploying to this repository and using packages from this repository, we're utilizing maven profiles. 

Make sure to check your local maven `settings.xml` file to determine which profiles you have active. 

If you want to have the `github-packages` profile active by default, set it as such in that file. Otherwise, make sure to run each deploy command with the option `-P 'github-packages'`. If you have any other profiles active by default that might interfere with the deployment, you can turn them off like so: `-P '!openmrs,!anotherprofile,github-packages'`. 
  
## Maven setup for OpenMRS modules

### Step 1: Add a new profile to `pom.xml`

Add the following to the `profiles` section of your root `pom.xml` file. 

```xml
<profiles>
    <profile>
        <!-- Github Packages Integration -->
        <id>github-packages</id>
        <distributionManagement>
        <!-- Deploy to Github Packages -->
        <repository>
            <id>github-packages</id>
            <name>Github iSantePlus Packages</name>
            <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
            <uniqueVersion>false</uniqueVersion>
        </repository>
        <snapshotRepository>
            <id>github-packages</id>
            <name>Github iSantePlus Packages</name>
            <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
            <uniqueVersion>true</uniqueVersion>
        </snapshotRepository>
        </distributionManagement>
        <repositories>
        <!-- Use the Github Packages Repo first when looking up dependencies -->
        <repository>
            <id>github-packages</id>
            <name>Github iSantePlus Packages</name>
            <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
        </repository>
        </repositories>
        <build>
        <plugins>
            <!-- Disable possible test jar generation -->
            <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <executions>
                <execution>
                <phase>none</phase>
                </execution>
            </executions>
            </plugin>
        </plugins>
        </build>
    </profile>
</profiles>
```
### Step 2: Add a new profile to `omod/pom.xml`

Since the OpenMRS `*.omod` file is not a standard maven artifact, we need to manually push it to the repository. The `omod` file is created in the `omod` submodule, so this profile should be added to `omod/pom.xml`. 

```xml
<profile>
    <id>github-packages</id>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-deploy-plugin</artifactId>
                <executions>
                    <execution>
                        <!-- Deploy OpenMRS omod file -->
                        <id>deploy-file</id>
                        <phase>deploy</phase>
                        <goals>
                            <goal>deploy-file</goal>
                        </goals>
                        <configuration>
                            <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
                            <file>target/${project.parent.artifactId}-${project.version}.omod</file>
                            <repositoryId>github-packages</repositoryId>
                            <pomFile>pom.xml</pomFile>
                            <packaging>omod</packaging>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
```

### Step 3: Github Personal Access Token

Create a Personal Access Token with package permissions on Github. See https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token for more information. 

### Step 4: Configure maven settings

Add the following section to your `.m2/settings.xm`l file:
```xml
<server>
	<id>isanteplus-github</id>
	<username>{{YOUR_GITHUB_USERNAME}}</username>
	<password>{{Your Personal Access Token}}</password>
</server>
```  

### Step 5: Deploy!
`mvn clean package deploy -P 'github-packages'`
  
## Package Versioning
See https://wiki.openmrs.org/display/docs/Versioning

## OpenMRS Omod Files
This repo can also be used to download `.omod` files. To see and use the available `.omod` files:
1. Sort the packages with the "omod" keyword (https://github.com/orgs/IsantePlus/packages?tab=packages&q=omod)
2. Click on the desired module
3. Download the most recent (highest version number) `*.omod` file from the `Assets` section in the right column.
4. Rename this file to, for example, `<artifactId>-<package-version>.omod`.
5. Use the file as a normal `.omod` file. 
