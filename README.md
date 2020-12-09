# iSantePlus Package Repository

See https://docs.github.com/en/free-pro-team@latest/packages/guides/configuring-apache-maven-for-use-with-github-packages

## Maven Setup Steps for OpenMRS Modules

1. add or replace the `distributionManagement` of the main `pom.xml` with the following:
  ```
    <!-- Github Packages Integration -->
    <distributionManagement>
      <repository>
        <id>isanteplus-github</id>
        <name>Github iSantePlus Packages</name>
        <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
      </repository>
      <snapshotRepository>
        <id>isanteplus-github</id>
        <name>Github iSantePlus Packages</name>
        <url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
      </snapshotRepository>
    </distributionManagement>
  ```
2. add the following item to the `repositories` section of the main `pom.xml`:
  ```
    <repository>
	<id>isanteplus-github</id>
	<name>Github iSantePlus Packages</name>
	<url>https://maven.pkg.github.com/isanteplus/isanteplus-packages</url>
	<snapshots><enabled>true</enabled></snapshots>
    </repository>
  ```
  
3. Create a `Personal Access Token` with package permissions on Github: 
   https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token

4. Add the following section to your `.m2/settings.xml` file: 
  ```
    <server>
      <id>isanteplus-github</id>
      <username>{{YOUR_GITHUB_USERNAME}}</username>
      <password>{{Your Personal Access Token}}</password>
    </server>
  ```
  
## Package Versioning
See https://wiki.openmrs.org/display/docs/Versioning

## OpenMRS Omod File Hosting
This repo also hosts `.omod` files that can be directly dowloaded. To see and use the available `.omod` files:
1. Sort the packages with the "omod" keyword (https://github.com/orgs/IsantePlus/packages?tab=packages&q=omod)
2. Click on the desired module
3. Download the `*-omod-x.x.x-*.jar` (not `*-tests.jar`) file from the `Assets` section in the right column.
4. Rename this file to, for example, `santedb-mpi-client-x.x.x-SNAPSHOT.omod` - or whatever module and version you're looking at.
5. Use the file as a normal .omod file. 
