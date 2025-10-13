# Maven model

- [project](#project)
- [parent](#parent)
- [organization](#organization)
- [license](#license)
- [developer](#developer)
- [contributor](#contributor)
- [mailingList](#mailinglist)
- [prerequisites](#prerequisites)
- [scm](#scm)
- [issueManagement](#scm)
- [ciManagement](#cimanagement)
- [notifier](#notifier)
- [distributionManagement](#distributionmanagement)
    1. [repository](#repository)
    2. [snapshotRepository](#snapshotrepository)
    3. [site](#site)
    4. [releases](#releases)
    5. [snapshots](#snapshots)
    6. [relocation](#relocation)
- [dependencyManagement](#dependencymanagement)
- [dependency](#dependency)
- [exclusion](#exclusion)
- [repository](#repository-1)
- [pluginRepository](#pluginrepository)
- [build](#build)
- [extension](#extension)
- [resource](#resource)
- [testResource](#testresource)
- [pluginManagement](#pluginmanagement)
    1. [plugin](#plugin)
- [execution](#execution)
- [reporting](#reporting)
- [plugin](#plugin-1)
- [reportSet](#reportset)
- [profile](#profile)
    1. [activation](#activation)
    2. [os](#os)
    3. [property](#property)
    4. [file](#file)
    5. [build](#build-1)

Maven project descriptor used in Maven.

An XSD is available at:

- [https://maven.apache.org/xsd/maven-v3_0_0.xsd](https://maven.apache.org/xsd/maven-v3_0_0.xsd) for Maven 1.1.
- [https://maven.apache.org/xsd/maven-4.0.0.xsd](https://maven.apache.org/xsd/maven-4.0.0.xsd) for Maven 3.0.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd"
         child.project.url.inherit.append.path="..">

    <modelVersion/>

    <parent>
        <groupId/>
        <artifactId/>
        <version/>
        <relativePath/>
    </parent>

    <groupId/>
    <artifactId/>
    <version/>
    <packaging/>
    <name/>
    <description/>
    <url/>
    <inceptionYear/>

    <organization>
        <name/>
        <url/>
    </organization>

    <licenses>
        <license>
            <name/>
            <url/>
            <distribution/>
            <comments/>
        </license>
    </licenses>

    <developers>
        <developer>
            <id/>
            <name/>
            <email/>
            <url/>
            <organization/>
            <organizationUrl/>
            <roles/>
            <timezone/>
            <properties>
                <key>value</key>
            </properties>
        </developer>
    </developers>

    <contributors>
        <contributor>
            <name/>
            <email/>
            <url/>
            <organization/>
            <organizationUrl/>
            <roles/>
            <timezone/>
            <properties>
                <key>value</key>
            </properties>
        </contributor>
    </contributors>

    <mailingLists>
        <mailingList>
            <name/>
            <subscribe/>
            <unsubscribe/>
            <post/>
            <archive/>
            <otherArchives/>
        </mailingList>
    </mailingLists>

    <prerequisites>
        <maven/>
    </prerequisites>

    <modules/>

    <scm child.scm.connection.inherit.append.path=".." 
         child.scm.developerConnection.inherit.append.path=".." 
         child.scm.url.inherit.append.path="..">
        <connection/>
        <developerConnection/>
        <tag/>
        <url/>
    </scm>

    <issueManagement>
        <system/>
        <url/>
    </issueManagement>

    <ciManagement>
        <system/>
        <url/>
        <notifiers>
            <notifier>
                <type/>
                <sendOnError/>
                <sendOnFailure/>
                <sendOnSuccess/>
                <sendOnWarning/>
                <address/>
                <configuration>
                    <key>value</key>
                </configuration>
            </notifier>
        </notifiers>
    </ciManagement>

    <distributionManagement>
        <repository>
            <uniqueVersion/>
            <releases>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </releases>
            <snapshots>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </snapshots>
            <id/>
            <name/>
            <url/>
            <layout/>
        </repository>

        <snapshotRepository>
            <uniqueVersion/>
            <releases>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </releases>
            <snapshots>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </snapshots>
            <id/>
            <name/>
            <url/>
            <layout/>
        </snapshotRepository>

        <site child.site.url.inherit.append.path="..">
            <id/>
            <name/>
            <url/>
        </site>

        <downloadUrl/>
        <relocation>
            <groupId/>
            <artifactId/>
            <version/>
            <message/>
        </relocation>
        <status/>
    </distributionManagement>

    <properties>
        <key>value</key>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId/>
                <artifactId/>
                <version/>
                <type/>
                <classifier/>
                <scope/>
                <systemPath/>
                <exclusions>
                    <exclusion>
                        <groupId/>
                        <artifactId/>
                    </exclusion>
                </exclusions>
                <optional/>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId/>
            <artifactId/>
            <version/>
            <type/>
            <classifier/>
            <scope/>
            <systemPath/>
            <exclusions>
                <exclusion>
                    <groupId/>
                    <artifactId/>
                </exclusion>
            </exclusions>
            <optional/>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <releases>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </releases>
            <snapshots>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </snapshots>
            <id/>
            <name/>
            <url/>
            <layout/>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <releases>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </releases>
            <snapshots>
                <enabled/>
                <updatePolicy/>
                <checksumPolicy/>
            </snapshots>
            <id/>
            <name/>
            <url/>
            <layout/>
        </pluginRepository>
    </pluginRepositories>

    <build>
        <sourceDirectory/>
        <scriptSourceDirectory/>
        <testSourceDirectory/>
        <outputDirectory/>
        <testOutputDirectory/>
        <extensions>
            <extension>
                <groupId/>
                <artifactId/>
                <version/>
            </extension>
        </extensions>
        <defaultGoal/>
        <resources>
            <resource>
                <targetPath/>
                <filtering/>
                <directory/>
                <includes/>
                <excludes/>
            </resource>
        </resources>
        <testResources>
            <testResource>
                <targetPath/>
                <filtering/>
                <directory/>
                <includes/>
                <excludes/>
            </testResource>
        </testResources>
        <directory/>
        <finalName/>
        <filters/>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId/>
                    <artifactId/>
                    <version/>
                    <extensions/>
                    <executions>
                        <execution>
                            <id/>
                            <phase/>
                            <goals/>
                            <inherited/>
                            <configuration/>
                        </execution>
                    </executions>
                    <dependencies>
                        <dependency>
                            <groupId/>
                            <artifactId/>
                            <version/>
                            <type/>
                            <classifier/>
                            <scope/>
                            <systemPath/>
                            <exclusions>
                                <exclusion>
                                    <groupId/>
                                    <artifactId/>
                                </exclusion>
                            </exclusions>
                            <optional/>
                        </dependency>
                    </dependencies>
                    <goals/>
                    <inherited/>
                    <configuration/>
                </plugin>
            </plugins>
        </pluginManagement>
        <plugins>
            <plugin>
                <groupId/>
                <artifactId/>
                <version/>
                <extensions/>
                <executions>
                    <execution>
                        <id/>
                        <phase/>
                        <goals/>
                        <inherited/>
                        <configuration/>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId/>
                        <artifactId/>
                        <version/>
                        <type/>
                        <classifier/>
                        <scope/>
                        <systemPath/>
                        <exclusions>
                            <exclusion>
                                <groupId/>
                                <artifactId/>
                            </exclusion>
                        </exclusions>
                        <optional/>
                    </dependency>
                </dependencies>
                <goals/>
                <inherited/>
                <configuration/>
            </plugin>
        </plugins>
    </build>

    <reports/>

    <reporting>
        <excludeDefaults/>
        <outputDirectory/>
        <plugins>
            <plugin>
                <groupId/>
                <artifactId/>
                <version/>
                <reportSets>
                    <reportSet>
                        <id/>
                        <reports/>
                        <inherited/>
                        <configuration/>
                    </reportSet>
                </reportSets>
                <inherited/>
                <configuration/>
            </plugin>
        </plugins>
    </reporting>

    <profiles>
        <profile>
            <id/>
            <activation>
                <activeByDefault/>
                <jdk/>
                <os>
                    <name/>
                    <family/>
                    <arch/>
                    <version/>
                </os>
                <property>
                    <name/>
                    <value/>
                </property>
                <file>
                    <missing/>
                    <exists/>
                </file>
            </activation>
            <build>
                <defaultGoal/>
                <resources>
                    <resource>
                        <targetPath/>
                        <filtering/>
                        <directory/>
                        <includes/>
                        <excludes/>
                    </resource>
                </resources>
                <testResources>
                    <testResource>
                        <targetPath/>
                        <filtering/>
                        <directory/>
                        <includes/>
                        <excludes/>
                    </testResource>
                </testResources>
                <directory/>
                <finalName/>
                <filters/>
                <pluginManagement>
                    <plugins>
                        <plugin>
                            <groupId/>
                            <artifactId/>
                            <version/>
                            <extensions/>
                            <executions>
                                <execution>
                                    <id/>
                                    <phase/>
                                    <goals/>
                                    <inherited/>
                                    <configuration/>
                                </execution>
                            </executions>
                            <dependencies>
                                <dependency>
                                    <groupId/>
                                    <artifactId/>
                                    <version/>
                                    <type/>
                                    <classifier/>
                                    <scope/>
                                    <systemPath/>
                                    <exclusions>
                                        <exclusion>
                                            <groupId/>
                                            <artifactId/>
                                        </exclusion>
                                    </exclusions>
                                    <optional/>
                                </dependency>
                            </dependencies>
                            <goals/>
                            <inherited/>
                            <configuration/>
                        </plugin>
                    </plugins>
                </pluginManagement>
                <plugins>
                    <plugin>
                        <groupId/>
                        <artifactId/>
                        <version/>
                        <extensions/>
                        <executions>
                            <execution>
                                <id/>
                                <phase/>
                                <goals/>
                                <inherited/>
                                <configuration/>
                            </execution>
                        </executions>
                        <dependencies>
                            <dependency>
                                <groupId/>
                                <artifactId/>
                                <version/>
                                <type/>
                                <classifier/>
                                <scope/>
                                <systemPath/>
                                <exclusions>
                                    <exclusion>
                                        <groupId/>
                                        <artifactId/>
                                    </exclusion>
                                </exclusions>
                                <optional/>
                            </dependency>
                        </dependencies>
                        <goals/>
                        <inherited/>
                        <configuration/>
                    </plugin>
                </plugins>
            </build>
            <modules/>
            <distributionManagement>
                <repository>
                    <uniqueVersion/>
                    <releases>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </releases>
                    <snapshots>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </snapshots>
                    <id/>
                    <name/>
                    <url/>
                    <layout/>
                </repository>
                <snapshotRepository>
                    <uniqueVersion/>
                    <releases>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </releases>
                    <snapshots>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </snapshots>
                    <id/>
                    <name/>
                    <url/>
                    <layout/>
                </snapshotRepository>
                <site child.site.url.inherit.append.path="..">
                    <id/>
                    <name/>
                    <url/>
                </site>
                <downloadUrl/>
                <relocation>
                    <groupId/>
                    <artifactId/>
                    <version/>
                    <message/>
                </relocation>
                <status/>
            </distributionManagement>
            <properties>
                <key>value</key>
            </properties>
            <dependencyManagement>
                <dependencies>
                    <dependency>
                        <groupId/>
                        <artifactId/>
                        <version/>
                        <type/>
                        <classifier/>
                        <scope/>
                        <systemPath/>
                        <exclusions>
                            <exclusion>
                                <groupId/>
                                <artifactId/>
                            </exclusion>
                        </exclusions>
                        <optional/>
                    </dependency>
                </dependencies>
            </dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId/>
                    <artifactId/>
                    <version/>
                    <type/>
                    <classifier/>
                    <scope/>
                    <systemPath/>
                    <exclusions>
                        <exclusion>
                            <groupId/>
                            <artifactId/>
                        </exclusion>
                    </exclusions>
                    <optional/>
                </dependency>
            </dependencies>
            <repositories>
                <repository>
                    <releases>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </releases>
                    <snapshots>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </snapshots>
                    <id/>
                    <name/>
                    <url/>
                    <layout/>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <releases>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </releases>
                    <snapshots>
                        <enabled/>
                        <updatePolicy/>
                        <checksumPolicy/>
                    </snapshots>
                    <id/>
                    <name/>
                    <url/>
                    <layout/>
                </pluginRepository>
            </pluginRepositories>
            <reports/>
            <reporting>
                <excludeDefaults/>
                <outputDirectory/>
                <plugins>
                    <plugin>
                        <groupId/>
                        <artifactId/>
                        <version/>
                        <reportSets>
                            <reportSet>
                                <id/>
                                <reports/>
                                <inherited/>
                                <configuration/>
                            </reportSet>
                        </reportSets>
                        <inherited/>
                        <configuration/>
                    </plugin>
                </plugins>
            </reporting>
        </profile>
    </profiles>
</project>
```

# project

The `<project>` tag is the **root element of a POM (Project Object Model) file** (`pom.xml`). It defines your Maven project and contains all the configuration, dependencies, and metadata required to build the project.

| **Attribute** | **Type** | **Description** |
| --- | --- | --- |
| `child.project.url.inherit.append.path` | `String` | Sometimes, URLs (like `project.url`) need to be **inherited and appended** for child modules, e.g., when generating site documentation, the semantic type is actually `Boolean`**Default value is**: `true`**Since**: Maven 3.6.1 |

**Example scenario:**

Suppose your parent POM defines a project URL: (true) 

```xml
<project>
    ...
    <url>https://github.com/my-org/my-project</url>
    <properties>
        <child.project.url.inherit.append.path>true</child.project.url.inherit.append.path>
    </properties>
    ...
</project>
```

Then a child module POM might do:

```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>child-module</artifactId>

    <url>
        ${project.url}/child-module
    </url>
</project>
```

**Explanation**:

- `child.project.url.inherit.append.path` (if your Maven setup or plugin recognizes it) signals that **the child module should take the parent URL and append its module path**.
- The result:
    
    Parent URL: `https://github.com/my-org/my-project`
    
    Child URL: `https://github.com/my-org/my-project/child-module`
    

**Parent POM:(false)**

```xml
<project>
    <url>https://github.com/my-org/my-project</url>
    <properties>
        <child.project.url.inherit.append.path>false</child.project.url.inherit.append.path>
    </properties>
</project>
```

**Child POM:**

```xml
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>child-module</artifactId>
    <url>${project.url}</url>
</project>
```

**Result:**

- Child module URL: `https://github.com/my-org/my-project` (same as parent)
- No `/child-module` appended at the end.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `modelVersion` | `String` | Declares to which version of project descriptor this POM conforms. |
| `parent` | `Parent` | The location of the parent project, if one exists. Values from the parent project will be the default for this project if they are left unspecified. The location is given as a group ID, artifact ID and version. |
| `groupId` | `String` | A universally unique identifier for a project. It is normal to use a fully-qualified package name to distinguish it from other projects with a similar name (eg. `org.apache.maven`). |
| `artifactId` | `String` | The identifier for this artifact that is unique within the group given by the group ID. An artifact is something that is either produced or used by a project. Examples of artifacts produced by Maven for a project include: JARs, source and binary distributions, and WARs. |
| `version` | `String` | The current version of the artifact produced by this project. |
| `packaging` | `String` | The type of artifact this project produces, for example `jar` `war` `ear` `pom`. Plugins can create their own packaging, and therefore their own packaging types, so this list does not contain all possible types.
**Default value**: `jar` |
| `name` | `String` | The full name of the project. |
| `description` | `String` | A detailed description of the project, used by Maven whenever it needs to describe the project, such as on the web site. While this element can be specified as CDATA to enable the use of HTML tags within the description, it is discouraged to allow plain text representation. If you need to modify the index page of the generated web site, you are able to specify your own instead of adjusting this text. |
| `url` | `String` | The URL to the project's homepage.**Default value is**: parent value [+ path adjustment] + (artifactId or project.directory property), or just parent value if project's `child.project.url.inherit.append.path="false"` |
| `inceptionYear` | `String` | The year of the project's inception, specified with 4 digits. This value is used when generating copyright notices as well as being informational. |
| `organization` | `Organization` | This element describes various attributes of the organization to which the project belongs. These attributes are utilized when documentation is created (for copyright notices and links). |
| `licenses/license*` | `List<License>` | **(Many)** This element describes all of the licenses for this project. Each license is described by a `license` element, which is then described by additional elements. Projects should only list the license(s) that applies to the project and not the licenses that apply to dependencies. If multiple licenses are listed, it is assumed that the user can select any of them, not that they must accept all. |
| `developers/developer*` | `List<Developer>` | **(Many)** Describes the committers of a project. |
| `contributors/contributor*` | `List<Contributor>` | **(Many)** Describes the contributors to a project that are not yet committers. |
| `mailingLists/mailingList*` | `List<MailingList>` | **(Many)** Contains information about a project's mailing lists. |
| `prerequisites` | `Prerequisites` | Describes the prerequisites in the build environment for this project. |
| `modules/module*` | `List<String>` | **(Many)** The modules (sometimes called subprojects) to build as a part of this project. Each module listed is a relative path to the directory containing the module. To be consistent with the way default urls are calculated from parent, it is recommended to have module names match artifact ids. |
| `scm` | `Scm` | Specification for the SCM used by the project, such as CVS, Subversion, etc. |
| `issueManagement` | `IssueManagement` | The project's issue management system information. |
| `ciManagement` | `CiManagement` | The project's continuous integration information. |
| `distributionManagement` | `DistributionManagement` | Distribution information for a project that enables deployment of the site and artifacts to remote web servers and repositories respectively. |
| `properties/*key*=*value**` | `Properties` | **(Many)** Properties that can be used throughout the POM as a substitution, and are used as filters in resources if enabled. The format is `<name>value</name>`. |
| `dependencyManagement` | `DependencyManagement` | Default dependency information for projects that inherit from this one. The dependencies in this section are not immediately resolved. Instead, when a POM derived from this one declares a dependency described by a matching groupId and artifactId, the version and other values from this section are used for that dependency if they were not already specified. |
| `dependencies/dependency*` | `List<Dependency>` | **(Many)** This element describes all of the dependencies associated with a project. These dependencies are used to construct a classpath for your project during the build process. They are automatically downloaded from the repositories defined in this project. See [the dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html) for more information. |
| `repositories/repository*` | `List<Repository>` | **(Many)** The lists of the remote repositories for discovering dependencies and extensions. |
| `pluginRepositories/pluginRepository*` | `List<Repository>` | **(Many)** The lists of the remote repositories for discovering plugins for builds and reports. |
| `build` | `Build` | Information required to build the project. |
| `reports` | `DOM` | **Deprecated**. Now ignored by Maven. |
| `reporting` | `Reporting` | This element includes the specification of report plugins to use to generate the reports on the Maven-generated site. These reports will be run when a user executes `mvn site`. All of the reports will be included in the navigation bar for browsing. |
| `profiles/profile*` | `List<Profile>` | **(Many)** A listing of project-local build profiles which will modify the build process when activated. |

# parent

The `<parent>` tag is used in a POM (`pom.xml`) to **declare that the current project is a child module of another Maven project (the parent project)**. It allows the child project to **inherit configurations, dependencies, plugins, and properties** defined in the parent POM. This is especially common in multi-module Maven projects.

 The children of this element are not interpolated and must be given as literal values.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group id of the parent project to inherit from. |
| `artifactId` | `String` | The artifact id of the parent project to inherit from. |
| `version` | `String` | The version of the parent project to inherit. |
| `relativePath` | `String` | The relative path of the parent `pom.xml` file within the check out. If not specified, it defaults to `../pom.xml`. Maven looks for the parent POM first in this location on the filesystem, then the local repository, and lastly in the remote repo. `relativePath` allows you to select a different location, for example when your structure is flat, or deeper without an intermediate parent POM. However, the group ID, artifact ID and version are still required, and must match the file in the location given or it will revert to the repository for the POM. This feature is only for enhancing the development in a local checkout of that project. Set the value to an empty string in case you want to disable the feature and always resolve the parent POM from the repositories.
**Default value**: `../pom.xml` |

### Example Directory Structure

```
my-project/
├─ parent-project/
│  └─ pom.xml
├─ modules/
│  ├─ module-a/
│  │  └─ pom.xml
│  └─ module-b/
│     └─ pom.xml
```

In this structure:

- The parent POM is at `my-project/parent-project/pom.xml`.
- The child modules are inside `my-project/modules/module-a` and `module-b`.

---

**Child POM Using `<relativePath>`**

**module-a/pom.xml:**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0</version>
        <relativePath>../../parent-project/pom.xml</relativePath>
    </parent>

    <artifactId>module-a</artifactId>

</project>

```

**Explanation of `<relativePath>`:**

- `../../parent-project/pom.xml` means:
    1. Go **two levels up** from `modules/module-a`.
    2. Enter the `parent-project` folder.
    3. Use `pom.xml` as the parent POM.
- If you **omit `<relativePath>`**, Maven assumes `../pom.xml` by default.
- If the parent POM is **already installed in your local Maven repository**, you can set `<relativePath/>` to empty to skip local search:

```xml
<relativePath/>
```

# **organization**

Specifies the organization that produces this project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The full name of the organization. |
| `url` | `String` | The URL to the organization's home page. |

# **license**

Describes the licenses for this project. This is used to generate the license page of the project's web site, as well as being taken into consideration in other reporting and validation. The licenses listed for the project are that of the project itself, and not of dependencies.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The full legal name of the license. |
| `url` | `String` | The official url for the license text. |
| `distribution` | `String` | The primary method by which this project may be distributed.**repo**may be downloaded from the Maven repository**manual**user must manually download and install the dependency. |
| `comments` | `String` | Addendum information pertaining to this license. |

# **developer**

Information about one of the committers on this project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `id` | `String` | The unique ID of the developer in the SCM. |
| `name` | `String` | The full name of the contributor. |
| `email` | `String` | The email address of the contributor. |
| `url` | `String` | The URL for the homepage of the contributor. |
| `organization` | `String` | The organization to which the contributor belongs. |
| `organizationUrl` | `String` | The URL of the organization. |
| `roles/role*` | `List<String>` | **(Many)** The roles the contributor plays in the project. Each role is described by a `role` element, the body of which is a role name. This can also be used to describe the contribution. |
| `timezone` | `String` | The timezone the contributor is in. Typically, this is a number in the range [-12](http://en.wikipedia.org/wiki/UTC%E2%88%9212:00) to [+14](http://en.wikipedia.org/wiki/UTC%2B14:00) or a valid time zone id like "America/Montreal" (UTC-05:00) or "Europe/Paris" (UTC+01:00). |
| `properties/*key*=*value**` | `Properties` | **(Many)** Properties about the contributor, such as an instant messenger handle. |

# **contributor**

Description of a person who has contributed to the project, but who does not have commit privileges. Usually, these contributions come in the form of patches submitted.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The full name of the contributor. |
| `email` | `String` | The email address of the contributor. |
| `url` | `String` | The URL for the homepage of the contributor. |
| `organization` | `String` | The organization to which the contributor belongs. |
| `organizationUrl` | `String` | The URL of the organization. |
| `roles/role*` | `List<String>` | **(Many)** The roles the contributor plays in the project. Each role is described by a `role` element, the body of which is a role name. This can also be used to describe the contribution. |
| `timezone` | `String` | The timezone the contributor is in. Typically, this is a number in the range [-12](http://en.wikipedia.org/wiki/UTC%E2%88%9212:00) to [+14](http://en.wikipedia.org/wiki/UTC%2B14:00) or a valid time zone id like "America/Montreal" (UTC-05:00) or "Europe/Paris" (UTC+01:00). |
| `properties/*key*=*value**` | `Properties` | **(Many)** Properties about the contributor, such as an instant messenger handle. |

# **mailingList**

This element describes all of the mailing lists associated with a project. The auto-generated site references this information.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The name of the mailing list. |
| `subscribe` | `String` | The email address or link that can be used to subscribe to the mailing list. If this is an email address, a `mailto:` link will automatically be created when the documentation is created. |
| `unsubscribe` | `String` | The email address or link that can be used to unsubscribe to the mailing list. If this is an email address, a `mailto:` link will automatically be created when the documentation is created. |
| `post` | `String` | The email address or link that can be used to post to the mailing list. If this is an email address, a `mailto:` link will automatically be created when the documentation is created. |
| `archive` | `String` | The link to a URL where you can browse the mailing list archive. |
| `otherArchives/otherArchive*` | `List<String>` | **(Many)** The link to alternate URLs where you can browse the list archive. |

# **prerequisites**

Describes the prerequisites a project can have.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `maven` | `String` | For a plugin project (packaging is `maven-plugin`), the minimum version of Maven required to use the resulting plugin.
**Default value**: `2.0` |

# **scm**

**SCM** stands for **Source Code Management**. The `<scm>` element in a POM (`pom.xml`) specifies **where the project’s source code is stored**, how it can be accessed, and how Maven plugins (like the Release Plugin or Site Plugin) can interact with it. This information is mainly used when generating project reports, performing releases, or integrating with continuous integration systems.

| **Attribute** | **Type** | **Description** |
| --- | --- | --- |
| `child.scm.connection.inherit.append.path` | `String` | When children inherit from scm connection, append path or not? Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`**Default value is**: `true`**Since**: Maven 3.6.1 |
| `child.scm.developerConnection.inherit.append.path` | `String` | When children inherit from scm developer connection, append path or not? Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`**Default value is**: `true`**Since**: Maven 3.6.1 |
| `child.scm.url.inherit.append.path` | `String` | When children inherit from scm url, append path or not? Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`**Default value is**: `true`**Since**: Maven 3.6.1 |

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `connection` | `String` | The source control management system URL that describes the repository and how to connect to the repository. For more information, see the [URL format](https://maven.apache.org/scm/scm-url-format.html) and [list of supported SCMs](https://maven.apache.org/scm/scms-overview.html). This connection is read-only.**Default value is**: parent value [+ path adjustment] + (artifactId or project.directory property), or just parent value if scm's `child.scm.connection.inherit.append.path="false"` |
| `developerConnection` | `String` | Just like `connection`, but for developers, i.e. this scm connection will not be read only.**Default value is**: parent value [+ path adjustment] + (artifactId or project.directory property), or just parent value if scm's `child.scm.developerConnection.inherit.append.path="false"` |
| `tag` | `String` | The tag of current code. By default, it's set to HEAD during development.**Default value**: `HEAD` |
| `url` | `String` | The URL to the project's browsable SCM repository, such as ViewVC or Fisheye.**Default value is**: parent value [+ path adjustment] + (artifactId or project.directory property), or just parent value if scm's `child.scm.url.inherit.append.path="false"` |

# **issueManagement**

Information about the issue tracking (or bug tracking) system used to manage this project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `system` | `String` | The name of the issue management system, e.g. Bugzilla |
| `url` | `String` | URL for the issue management system used by the project. |

# **ciManagement**

The `<CiManagement>` element contains informations required to the continuous integration system of the project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `system` | `String` | The name of the continuous integration system, e.g. `continuum`. |
| `url` | `String` | URL for the continuous integration system used by the project if it has a web interface. |
| `notifiers/notifier*` | `List<Notifier>` | **(Many)** Configuration for notifying developers/users when a build is unsuccessful, including user information and notification mode. |

# **notifier**

Configures one method for notifying users/developers when a build breaks.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `type` | `String` | The mechanism used to deliver notifications.**Default value**: `mail` |
| `sendOnError` | `boolean` | Whether to send notifications on error.**Default value**: `true` |
| `sendOnFailure` | `boolean` | Whether to send notifications on failure.**Default value**: `true` |
| `sendOnSuccess` | `boolean` | Whether to send notifications on success.**Default value**: `true` |
| `sendOnWarning` | `boolean` | Whether to send notifications on warning.**Default value**: `true` |
| `address` | `String` | **Deprecated**. Where to send the notification to - eg email address. |
| `configuration/*key*=*value**` | `Properties` | **(Many)** Extended configuration specific to this notifier goes here. |

# **distributionManagement**

This elements describes all that pertains to distribution for a project. It is primarily used for deployment of artifacts and the site produced by the build.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `repository` | `DeploymentRepository` | Information needed to deploy the artifacts generated by the project to a remote repository. |
| `snapshotRepository` | `DeploymentRepository` | Where to deploy snapshots of artifacts to. If not given, it defaults to the `repository` element. |
| `site` | `Site` | Information needed for deploying the web site of the project. |
| `downloadUrl` | `String` | The URL of the project's download page. If not given users will be referred to the homepage given by `url`. This is given to assist in locating artifacts that are not in the repository due to licensing restrictions. |
| `relocation` | `Relocation` | Relocation information of the artifact if it has been moved to a new group ID and/or artifact ID. |
| `status` | `String` | Gives the status of this artifact in the remote repository. This must not be set in your local project, as it is updated by tools placing it in the repository. Valid values are: `none` (default), `converted` (repository manager converted this from a Maven 1 POM), `partner` (directly synced from a partner Maven repository), `deployed` (was deployed from a Maven instance), `verified` (has been hand verified as correct and final). |

# **repository**

Deployment repository contains the information needed for deploying to the remote repository, which adds uniqueVersion property to usual repositories for download.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `uniqueVersion` | `boolean` | Whether to assign snapshots a unique version comprised of the timestamp and build number, or to use the same version each time**Default value**: `true` |
| `releases` | `RepositoryPolicy` | How to handle downloading of releases from this repository. |
| `snapshots` | `RepositoryPolicy` | How to handle downloading of snapshots from this repository. |
| `id` | `String` | A unique identifier for a repository. This is used to match the repository to configuration in the `settings.xml` file, for example. Furthermore, the identifier is used during POM inheritance and profile injection to detect repositories that should be merged. |
| `name` | `String` | Human readable name of the repository. |
| `url` | `String` | The url of the repository, in the form `protocol://hostname/path`. |
| `layout` | `String` | The type of layout this repository uses for locating and storing artifacts - can be `legacy` or `default`.
**Default value**: `default` |

# **releases**

Download policy.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `enabled` | `String` | Whether to use this repository for downloading this type of artifact. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `updatePolicy` | `String` | The frequency for downloading updates - can be `always,` `daily` (default), `interval:XXX` (in minutes) or `never` (only if it doesn't exist locally). |
| `checksumPolicy` | `String` | What to do when verification of an artifact checksum fails. Valid values are `ignore` , `fail` or `warn` (the default). |

# **snapshots**

Download policy.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `enabled` | `String` | Whether to use this repository for downloading this type of artifact. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `updatePolicy` | `String` | The frequency for downloading updates - can be `always,` `daily` (default), `interval:XXX` (in minutes) or `never` (only if it doesn't exist locally). |
| `checksumPolicy` | `String` | What to do when verification of an artifact checksum fails. Valid values are `ignore` , `fail` or `warn` (the default). |

# **snapshotRepository**

Deployment repository contains the information needed for deploying to the remote repository, which adds uniqueVersion property to usual repositories for download.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `uniqueVersion` | `boolean` | Whether to assign snapshots a unique version comprised of the timestamp and build number, or to use the same version each time**Default value**: `true` |
| [`releases`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_releases) | `RepositoryPolicy` | How to handle downloading of releases from this repository. |
| [`snapshots`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_snapshots) | `RepositoryPolicy` | How to handle downloading of snapshots from this repository. |
| `id` | `String` | A unique identifier for a repository. This is used to match the repository to configuration in the `settings.xml` file, for example. Furthermore, the identifier is used during POM inheritance and profile injection to detect repositories that should be merged. |
| `name` | `String` | Human readable name of the repository. |
| `url` | `String` | The url of the repository, in the form `protocol://hostname/path`. |
| `layout` | `String` | The type of layout this repository uses for locating and storing artifacts - can be `legacy` or `default`.
**Default value**: `default` |

# **site**

Contains the information needed for deploying websites.

| **Attribute** | **Type** | **Description** |
| --- | --- | --- |
| `child.site.url.inherit.append.path` | `String` | When children inherit from distribution management site url, append path or not? Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`**Default value is**: `true`**Since**: Maven 3.6.1 |

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `id` | `String` | A unique identifier for a deployment location. This is used to match the site to configuration in the `settings.xml` file, for example. |
| `name` | `String` | Human readable name of the deployment location. |
| `url` | `String` | The url of the location where website is deployed, in the form `protocol://hostname/path`.**Default value is**: parent value [+ path adjustment] + (artifactId or project.directory property), or just parent value if site's `child.site.url.inherit.append.path="false"` |

# **relocation**

Describes where an artifact has moved to. If any of the values are omitted, it is assumed to be the same as it was before.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group ID the artifact has moved to. |
| `artifactId` | `String` | The new artifact ID of the artifact. |
| `version` | `String` | The new version of the artifact. |
| `message` | `String` | An additional message to show the user about the move, such as the reason. |

# **dependencyManagement**

Section for management of default dependency information for use in a group of POMs.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `dependencies/[dependency](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_dependency)*` | `List<Dependency>` | **(Many)** The dependencies specified here are not used until they are referenced in a POM within the group. This allows the specification of a "standard" version for a particular dependency. |

# **dependency**

The `<dependency>` element contains information about a dependency of the project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The project group that produced the dependency, e.g. `org.apache.maven`. |
| `artifactId` | `String` | The unique id for an artifact produced by the project group, e.g. `maven-artifact`. |
| `version` | `String` | The version requirement of the dependency such as `3.2.1`. The actual version will be resolved by dependency mediation. The version requirement can also be specified as a range of versions such as `[3.2.0,)`. However, this is discouraged since it may break *predictability* of the resolved version. See the [Dependency Version Requirement Specification](https://maven.apache.org/pom.html#Dependency_Version_Requirement_Specification) and [Transitive Dependencies](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies) for more details. |
| `type` | `String` | The type of dependency, that will be mapped to a file extension, an optional classifier, and a few other attributes. Some examples are `jar`, `war`, `ejb-client` and `test-jar`: see [default artifact handlers](https://maven.apache.org/ref/3.9.11/maven-core/artifact-handlers.html) for a list. New types can be defined by extensions, so this is not a complete list.
**Default value**: `jar` |
| `classifier` | `String` | The classifier of the dependency. It is appended to the filename after the version. This allows:
• referring to attached artifact, for example `sources` and `javadoc`: see [default artifact handlers](https://maven.apache.org/ref/3.9.11/maven-core/artifact-handlers.html) for a list,
• distinguishing two artifacts that belong to the same POM but were built differently. For example, `jdk14` and `jdk15`. |
| `scope` | `String` | The scope of the dependency - `compile`, `runtime`, `test`, `system`, and `provided`. Used to calculate the various classpaths used for compilation, testing, and so on. It also assists in determining which artifacts to include in a distribution of this project. For more information, see [the dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html). The default scope is `compile`. |
| `systemPath` | `String` | FOR SYSTEM SCOPE ONLY. Note that use of this property is **discouraged** and may be replaced in later versions. This specifies the path on the filesystem for this dependency. Requires an absolute path for the value, not relative. Use a property that gives the machine specific absolute path, e.g. `${java.home}`. |
| `exclusions/[exclusion](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_exclusion)*` | `List<Exclusion>` | **(Many)** Lists a set of artifacts that should be excluded from this dependency's artifact list when it comes to calculating transitive dependencies. |
| `optional` | `String` | Indicates the dependency is optional for use of this library. While the version of the dependency will be taken into account for dependency calculation if the library is used elsewhere, it will not be passed on transitively. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `false`. |

# **exclusion**

The `<exclusion>` element contains informations required to exclude an artifact to the project.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group ID of the project to exclude. |
| `artifactId` | `String` | The artifact ID of the project to exclude. |

# **repository**

A repository contains the information needed for establishing connections with remote repository.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `releases` | `RepositoryPolicy` | How to handle downloading of releases from this repository. |
| `snapshots` | `RepositoryPolicy` | How to handle downloading of snapshots from this repository. |
| `id` | `String` | A unique identifier for a repository. This is used to match the repository to configuration in the `settings.xml` file, for example. Furthermore, the identifier is used during POM inheritance and profile injection to detect repositories that should be merged. |
| `name` | `String` | Human readable name of the repository. |
| `url` | `String` | The url of the repository, in the form `protocol://hostname/path`. |
| `layout` | `String` | The type of layout this repository uses for locating and storing artifacts - can be `legacy` or `default`. |

# **pluginRepository**

A repository contains the information needed for establishing connections with remote repository.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `releases` | `RepositoryPolicy` | How to handle downloading of releases from this repository. |
| `snapshots` | `RepositoryPolicy` | How to handle downloading of snapshots from this repository. |
| `id` | `String` | A unique identifier for a repository. This is used to match the repository to configuration in the `settings.xml` file, for example. Furthermore, the identifier is used during POM inheritance and profile injection to detect repositories that should be merged. |
| `name` | `String` | Human readable name of the repository. |
| `url` | `String` | The url of the repository, in the form `protocol://hostname/path`. |
| `layout` | `String` | The type of layout this repository uses for locating and storing artifacts - can be `legacy` or `default`.
**Default value**: `default` |

# **build**

The `<build>` element contains informations required to build the project. Default values are defined in Super POM.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `sourceDirectory` | `String` | This element specifies a directory containing the source of the project. The generated build system will compile the sources from this directory when the project is built. The path given is relative to the project descriptor. The default value is `src/main/java`. |
| `scriptSourceDirectory` | `String` | This element specifies a directory containing the script sources of the project. This directory is meant to be different from the sourceDirectory, in that its contents will be copied to the output directory in most cases (since scripts are interpreted rather than compiled). The default value is `src/main/scripts`. |
| `testSourceDirectory` | `String` | This element specifies a directory containing the unit test source of the project. The generated build system will compile these directories when the project is being tested. The path given is relative to the project descriptor. The default value is `src/test/java`. |
| `outputDirectory` | `String` | The directory where compiled application classes are placed. The default value is `target/classes`. |
| `testOutputDirectory` | `String` | The directory where compiled test classes are placed. The default value is `target/test-classes`. |
| `extensions/extension*` | `List<Extension>` | **(Many)** A set of build extensions to use from this project. |
| `defaultGoal` | `String` | The default goal to execute when none is specified for the project. In a multi-module build, only the default goal of the top-level project is relevant. That is, the default goals of child modules are ignored. Multiple goals can be separated by whitespace. |
| `resources/resource*` | `List<Resource>` | **(Many)** This element describes all of the classpath resources such as properties files associated with a project. These resources are often included in the final package. The default value is `src/main/resources`. |
| `testResources/testResource*` | `List<Resource>` | **(Many)** This element describes all of the classpath resources such as properties files associated with a project's unit tests. The default value is `src/test/resources`. |
| `directory` | `String` | The directory where all files generated by the build are placed. The default value is `target`. |
| `finalName` | `String` | The filename (excluding the extension, and with no path information) that the produced artifact will be called. The default value is `${artifactId}-${version}`. |
| `filters/filter*` | `List<String>` | **(Many)** The list of filter properties files that are used when filtering is enabled. |
| `pluginManagement` | `PluginManagement` | Default plugin information to be made available for reference by projects derived from this one. This plugin configuration will not be resolved or bound to the lifecycle unless referenced. Any local configuration for a given plugin will override the plugin's entire definition here. |
| `plugins/plugin*` | `List<Plugin>` | **(Many)** The list of plugins to use. |

# **extension**

Describes a build extension to utilise.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group ID of the extension's artifact. |
| `artifactId` | `String` | The artifact ID of the extension. |
| `version` | `String` | The version of the extension. |

# **resource**

This element describes all of the classpath resources associated with a project or unit tests.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `targetPath` | `String` | Describe the resource target path. The path is relative to the target/classes directory (i.e. `${project.build.outputDirectory}`). For example, if you want that resource to appear in a specific package (`org.apache.maven.messages`), you must specify this element with this value: `org/apache/maven/messages`. This is not required if you simply put the resources in that directory structure at the source, however. |
| `filtering` | `String` | Whether resources are filtered to replace tokens with parameterised values or not. The values are taken from the `properties` element and from the properties in the files listed in the `filters` element. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `false`. |
| `directory` | `String` | Describe the directory where the resources are stored. The path is relative to the POM. |
| `includes/include*` | `List<String>` | **(Many)** A list of patterns to include, e.g. `**/*.xml`. |
| `excludes/exclude*` | `List<String>` | **(Many)** A list of patterns to exclude, e.g. `**/*.xml` |

# **testResource**

This element describes all of the classpath resources associated with a project or unit tests.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `targetPath` | `String` | Describe the resource target path. The path is relative to the target/classes directory (i.e. `${project.build.outputDirectory}`). For example, if you want that resource to appear in a specific package (`org.apache.maven.messages`), you must specify this element with this value: `org/apache/maven/messages`. This is not required if you simply put the resources in that directory structure at the source, however. |
| `filtering` | `String` | Whether resources are filtered to replace tokens with parameterised values or not. The values are taken from the `properties` element and from the properties in the files listed in the `filters` element. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `false`. |
| `directory` | `String` | Describe the directory where the resources are stored. The path is relative to the POM. |
| `includes/include*` | `List<String>` | **(Many)** A list of patterns to include, e.g. `**/*.xml`. |
| `excludes/exclude*` | `List<String>` | **(Many)** A list of patterns to exclude, e.g. `**/*.xml` |

# **pluginManagement**

Section for management of default plugin information for use in a group of POMs.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `plugins/plugin*` | `List<Plugin>` | **(Many)** The list of plugins to use. |

# **plugin**

The `<plugin>` element contains informations required for a plugin.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group ID of the plugin in the repository.**Default value**: `org.apache.maven.plugins` |
| `artifactId` | `String` | The artifact ID of the plugin in the repository. |
| `version` | `String` | The version (or valid range of versions) of the plugin to be used. |
| `extensions` | `String` | Whether to load Maven extensions (such as packaging and type handlers) from this plugin. For performance reasons, this should only be enabled when necessary. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `false`. |
| `executions/execution*` | `List<PluginExecution>` | **(Many)** Multiple specifications of a set of goals to execute during the build lifecycle, each having (possibly) a different configuration. |
| `dependencies/dependency*` | `List<Dependency>` | **(Many)** Additional dependencies that this project needs to introduce to the plugin's classloader. |
| `goals` | `DOM` | **Deprecated**. Unused by Maven. |
| `inherited` | `String` | Whether any configuration should be propagated to child POMs. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `configuration` | `DOM` | The configuration as DOM object.
By default, every element content is trimmed, but starting with Maven 3.1.0, you can add `xml:space="preserve"` to elements you want to preserve whitespace.
You can control how child POMs inherit configuration from parent POMs by adding `combine.children` or `combine.self` attributes to the children of the configuration element:
• `combine.children`: available values are `merge` (default) and `append`,
• `combine.self`: available values are `merge` (default) and `override`.
See [POM Reference documentation](https://maven.apache.org/pom.html#Plugins) and [Xpp3DomUtils](https://codehaus-plexus.github.io/plexus-utils/apidocs/org/codehaus/plexus/util/xml/Xpp3DomUtils.html) for more information. |

# **execution**

The `<execution>` element contains informations required for the execution of a plugin.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `id` | `String` | The identifier of this execution for labelling the goals during the build, and for matching executions to merge during inheritance and profile injection.**Default value**: `default` |
| `phase` | `String` | The build lifecycle phase to bind the goals in this execution to. If omitted, the goals will be bound to the default phase specified by the plugin. |
| `goals/goal*` | `List<String>` | **(Many)** The goals to execute with the given configuration. |
| `inherited` | `String` | Whether any configuration should be propagated to child POMs. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `configuration` | `DOM` | The configuration as DOM object.
By default, every element content is trimmed, but starting with Maven 3.1.0, you can add `xml:space="preserve"` to elements you want to preserve whitespace.
You can control how child POMs inherit configuration from parent POMs by adding `combine.children` or `combine.self` attributes to the children of the configuration element:
• `combine.children`: available values are `merge` (default) and `append`,
• `combine.self`: available values are `merge` (default) and `override`.
See [POM Reference documentation](https://maven.apache.org/pom.html#Plugins) and [Xpp3DomUtils](https://codehaus-plexus.github.io/plexus-utils/apidocs/org/codehaus/plexus/util/xml/Xpp3DomUtils.html) for more information. |

# **reporting**

Section for management of reports and their configuration.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `excludeDefaults` | `String` | If true, then the default reports are not included in the site generation. This includes the reports in the "Project Info" menu. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `false`. |
| `outputDirectory` | `String` | Where to store all of the generated reports. The default is `${project.build.directory}/site`. |
| `plugins/plugin*` | `List<ReportPlugin>` | **(Many)** The reporting plugins to use and their configuration. |

# **plugin**

The `<plugin>` element in `<reporting><plugins>` contains informations required for a report plugin.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `groupId` | `String` | The group ID of the reporting plugin in the repository.**Default value**: `org.apache.maven.plugins` |
| `artifactId` | `String` | The artifact ID of the reporting plugin in the repository. |
| `version` | `String` | The version of the reporting plugin to be used. Starting with Maven 3, if no version is defined explicitely, version is searched in `build/plugins` then in `build/pluginManagement`. |
| `reportSets/[reportSet](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_reportSet)*` | `List<ReportSet>` | **(Many)** Multiple specifications of a set of reports, each having (possibly) different configuration. This is the reporting parallel to an `execution` in the build. |
| `inherited` | `String` | Whether any configuration should be propagated to child POMs. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `configuration` | `DOM` | The configuration as DOM object.
By default, every element content is trimmed, but starting with Maven 3.1.0, you can add `xml:space="preserve"` to elements you want to preserve whitespace.
You can control how child POMs inherit configuration from parent POMs by adding `combine.children` or `combine.self` attributes to the children of the configuration element:
• `combine.children`: available values are `merge` (default) and `append`,
• `combine.self`: available values are `merge` (default) and `override`.
See [POM Reference documentation](https://maven.apache.org/pom.html#Plugins) and [Xpp3DomUtils](https://codehaus-plexus.github.io/plexus-utils/apidocs/org/codehaus/plexus/util/xml/Xpp3DomUtils.html) for more information. |

# **reportSet**

Represents a set of reports and configuration to be used to generate them.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `id` | `String` | The unique id for this report set, to be used during POM inheritance and profile injection for merging of report sets.**Default value**: `default` |
| `reports/report*` | `List<String>` | **(Many)** The list of reports from this plugin which should be generated from this set. |
| `inherited` | `String` | Whether any configuration should be propagated to child POMs. Note: While the type of this field is `String` for technical reasons, the semantic type is actually `Boolean`. Default value is `true`. |
| `configuration` | `DOM` | The configuration as DOM object.
By default, every element content is trimmed, but starting with Maven 3.1.0, you can add `xml:space="preserve"` to elements you want to preserve whitespace.
You can control how child POMs inherit configuration from parent POMs by adding `combine.children` or `combine.self` attributes to the children of the configuration element:
• `combine.children`: available values are `merge` (default) and `append`,
• `combine.self`: available values are `merge` (default) and `override`.
See [POM Reference documentation](https://maven.apache.org/pom.html#Plugins) and [Xpp3DomUtils](https://codehaus-plexus.github.io/plexus-utils/apidocs/org/codehaus/plexus/util/xml/Xpp3DomUtils.html) for more information. |

# **profile**

Modifications to the build process which is activated based on environmental parameters or command line arguments.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `id` | `String` | The identifier of this build profile. This is used for command line activation, and identifies profiles to be merged.**Default value**: `default` |
| [`activation`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_activation) | `Activation` | The conditional logic which will automatically trigger the inclusion of this profile. |
| [`build`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_profile_build) | `BuildBase` | Information required to build the project. |
| `modules/module*` | `List<String>` | **(Many)** The modules (sometimes called subprojects) to build as a part of this project. Each module listed is a relative path to the directory containing the module. To be consistent with the way default urls are calculated from parent, it is recommended to have module names match artifact ids. |
| [`distributionManagement`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_distributionManagement) | `DistributionManagement` | Distribution information for a project that enables deployment of the site and artifacts to remote web servers and repositories respectively. |
| `properties/*key*=*value**` | `Properties` | **(Many)** Properties that can be used throughout the POM as a substitution, and are used as filters in resources if enabled. The format is `<name>value</name>`. |
| [`dependencyManagement`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_dependencyManagement) | `DependencyManagement` | Default dependency information for projects that inherit from this one. The dependencies in this section are not immediately resolved. Instead, when a POM derived from this one declares a dependency described by a matching groupId and artifactId, the version and other values from this section are used for that dependency if they were not already specified. |
| `dependencies/[dependency](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_dependency)*` | `List<Dependency>` | **(Many)** This element describes all of the dependencies associated with a project. These dependencies are used to construct a classpath for your project during the build process. They are automatically downloaded from the repositories defined in this project. See [the dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html) for more information. |
| `repositories/[repository](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_repository)*` | `List<Repository>` | **(Many)** The lists of the remote repositories for discovering dependencies and extensions. |
| `pluginRepositories/[pluginRepository](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_pluginRepository)*` | `List<Repository>` | **(Many)** The lists of the remote repositories for discovering plugins for builds and reports. |
| `reports` | `DOM` | **Deprecated**. Now ignored by Maven. |
| [`reporting`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_reporting) | `Reporting` | This element includes the specification of report plugins to use to generate the reports on the Maven-generated site. These reports will be run when a user executes `mvn site`. All of the reports will be included in the navigation bar for browsing. |

# **activation**

The conditions within the build runtime environment which will trigger the automatic inclusion of the build profile. Multiple conditions can be defined, which must be all satisfied to activate the profile.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `activeByDefault` | `boolean` | If set to true, this profile will be active unless another profile in this pom is activated using the command line -P option or by one of that profile's activators.**Default value**: `false` |
| `jdk` | `String` | Specifies that this profile will be activated when a matching JDK is detected. For example, `1.4` only activates on JDKs versioned 1.4, while `!1.4` matches any JDK that is not version 1.4. Ranges are supported too: `[1.5,)` activates when the JDK is 1.5 minimum. |
| [`os`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_os) | `ActivationOS` | Specifies that this profile will be activated when matching operating system attributes are detected. |
| [`property`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_property) | `ActivationProperty` | Specifies that this profile will be activated when this property is specified. |
| [`file`](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#class_file) | `ActivationFile` | Specifies that this profile will be activated based on existence of a file. |

# **os**

This is an activator which will detect an operating system's attributes in order to activate its profile.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The name of the operating system to be used to activate the profile. This must be an exact match of the `${os.name}` Java property, such as `Windows XP`. |
| `family` | `String` | The general family of the OS to be used to activate the profile, such as `windows` or `unix`. |
| `arch` | `String` | The architecture of the operating system to be used to activate the profile. |
| `version` | `String` | The version of the operating system to be used to activate the profile. |

# **property**

This is the property specification used to activate a profile. If the value field is empty, then the existence of the named property will activate the profile, otherwise it does a case-sensitive match against the property value as well.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `name` | `String` | The name of the property to be used to activate a profile. |
| `value` | `String` | The value of the property required to activate a profile. |

# **file**

This is the file specification used to activate the profile. The `missing` value is the location of a file that needs to exist, and if it doesn't, the profile will be activated. On the other hand, `exists` will test for the existence of the file and if it is there, the profile will be activated.

Variable interpolation for these file specifications is limited to `${project.basedir}`, system properties and user properties.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `missing` | `String` | The name of the file that must be missing to activate the profile. |
| `exists` | `String` | The name of the file that must exist to activate the profile. |

# **build**

Build configuration in a profile.

| **Element** | **Type** | **Description** |
| --- | --- | --- |
| `defaultGoal` | `String` | The default goal to execute when none is specified for the project. In a multi-module build, only the default goal of the top-level project is relevant. That is, the default goals of child modules are ignored. Multiple goals can be separated by whitespace. |
| `resources/resource*` | `List<Resource>` | **(Many)** This element describes all of the classpath resources such as properties files associated with a project. These resources are often included in the final package. The default value is `src/main/resources`. |
| `testResources/testResource*` | `List<Resource>` | **(Many)** This element describes all of the classpath resources such as properties files associated with a project's unit tests. The default value is `src/test/resources`. |
| `directory` | `String` | The directory where all files generated by the build are placed. The default value is `target`. |
| `finalName` | `String` | The filename (excluding the extension, and with no path information) that the produced artifact will be called. The default value is `${artifactId}-${version}`. |
| `filters/filter*` | `List<String>` | **(Many)** The list of filter properties files that are used when filtering is enabled. |
| `pluginManagement` | `PluginManagement` | Default plugin information to be made available for reference by projects derived from this one. This plugin configuration will not be resolved or bound to the lifecycle unless referenced. Any local configuration for a given plugin will override the plugin's entire definition here. |
| `plugins/plugin*` | `List<Plugin>` | **(Many)** The list of plugins to use. |