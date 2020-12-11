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

## Github Actions - CI and Releases
`.github/workflows/ci.yml`
```yml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Cache Maven packages
        uses: actions/cache@v2
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build with Maven
        run: mvn -B install
        
      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo It Ran!
```

`.github/workflows/release.yml`
```yml
name: Publish package to GitHub Packages
on:
  release:
    types: [created]
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: 1.8
      - name: Publish package
        run: mvn -B -DskipTests deploy
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
### Release Process
1. Finalize PR from feature branch into master branch

2. Use the GitHub UI to draft a new release and add a new tag with the version from the project pom file.
