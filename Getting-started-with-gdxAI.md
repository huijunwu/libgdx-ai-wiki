## Maven and Gradle

gdxAI is available in Maven. The easiest way of going about this is to add the framework as a dependency of an existing [Maven](http://maven.apache.org/) or [Gradle](http://www.gradle.org/) project. There are two kinds of gdxAI releases.

* **Official**: well tested, stable packages that get released periodically. Its URL is: https://oss.sonatype.org/content/repositories/releases
* **Snapshots**: the result of nightly builds, they are more likely to break in unexpected ways. It's URL is: https://oss.sonatype.org/content/repositories/snapshots

### Maven

First, you need to add the desired repository to your `pom.xml` file. Change the URL depending on whether you want releases or snapshots.
```xml
	<repositories>
		<repository>
			<id>sonatype</id>
			<name>Sonatype</name>
			<url>https://oss.sonatype.org/content/repositories/releases</url>
		</repository>
	</repositories>
```
Then you need to add the gdxAI dependency.
```xml
	<dependency>
		<groupId>com.badlogicgames.gdx</groupId>
		<artifactId>gdx-ai</artifactId>
		<version>1.4.0</version>
	</dependency>
```
### Gradle

Add the Maven repositories to your `gradle.build` file.
```groovy
    repositories {
        maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
        maven { url "https://oss.sonatype.org/content/repositories/releases/" }
    }
```
Add the dependency to the `dependencies` element.
```groovy
    dependencies {
        compile "com.badlogicgames.gdx:gdx-ai:1.4.0"
    }
```
## Working from sources

If you want the most bleeding edge version of gdxAI or prefer to make your own modifications, working from sources might be the best option. Follow these steps.

1. Install [Gradle](http://www.gradle.org/downloads)
2. Clone the Git repository `git clone git@github.com:libgdx/gdx-ai.git`
3. Import the `gdx-ai` and `tests` projects into the IDE of your choice. Using the command line is also an option.
4. Now you can either:
  * Run the `uploadArchives` task and generate three jar files: bytecode, sources and javadocs. They will be in your local Maven repository.
  * Add the `gdx-ai` project as a dependency to yours.

## Using gdxAI in a Libgdx project

If you already have a Libgdx project and want to use gdxAI you need to add the appropriate repository to the `gradle.build` file as we saw in the previous section. Additionally, you will need to add the following dependencies to your projects.
```groovy
	project(":core") {
		dependencies {
			compile "com.badlogicgames.gdx:gdx-ai:1.4.0"
		}
	}

	project(":android") {
		dependencies {
			compile "com.badlogicgames.gdx:gdx-ai:1.4.0"
		}
	}

	project(":html") {
		dependencies {
			compile "com.badlogicgames.gdx:gdx-ai:1.4.0:sources"
		}
	}
```
Don't forget to update your GdxDefinition.gwt.xml and GdxDefinitionSuperdev.gwt.xml with the proper inheitance:
```
    <inherits name='com.badlogic.gdx.ai' />
```