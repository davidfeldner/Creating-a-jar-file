# Creating a jar file

## Making a simple jar for the hand-ins

For the hand-ins you don't have to create a 'fat' standalone jar which includes all the JavaFX libraries, since we can just specify the JavaFX libraries manually, or just clone your code from Github.
Simply specify the main class in the build.gradle like this:

```gradle
jar {
  manifest {
    attributes "Main-Class": "{your-mainClass}"
  }
}
```

(You can find the main class above in build.gradle in application -> mainClass, and copy it from there)
For example:
```gradle
jar {
  manifest {
    attributes "Main-Class": "org.example.demo.App"
  }
}
```
Press the little button to reload the build.gradle and simply run your program, running the program will run the 'gradle jar task', you can also just run `gradle jar`. Check in /build/libs, there should be a .jar file which you can use.
**And you are done**, this jar is fine for the hand-ins! But when you develop your final project you will need a 'fat' jar.

If you try to run the program **it should error and say that it is missing JavaFX components. Don't worry about it.**

<details>
  <summary>How are we then going to run it? </summary>
With this command we can run it, but you have to have the javafx libraries in a folder on your pc:

```bash
java --module-path {path-to-java-fx-libs} --add-modules javafx.controls,javafx.fxml -jar {name}.jar
```
</details>

# Making a fat Jar 
Don't worry about this for the hand-ins. :)
<details>
<summary> I want to see</summary>
Making a fat jar with all plugins is usually not recommended in java, since it breaks the separation of modules, and there are other solutions like `jlink`, `jpackage` and `native-image`, but for this course you will, as far as i know, still need to make one, so read on. :)

## Prerequisites
A fat jar file can't start a class which extends the Application class. For this reason you will need to make a launcher class which does not extend anything, and calls the main method of your application. Example of Launcher.java:

```java
public class Launcher {
    public static void main(String[] args) {
        HelloApplication.main(args);
    }
}
```

## Jar with modules (Recommended)

Since java 9, it is recommended to use modules to separate groups of packages. When creating a new JavaFX project in Intellij, you are already using modules. There will be a module-info.java file in /src/main/java, and this file specifies what other modules are used, and what modules can access your code.

### Make a fat jar with modules

Intellij has built in support to create jar files:

- Go to: File -> Project Structure -> Artifacts
- Add a new artifact with +
- Choose JAR -> from modules with dependencies.
- Select your Launcher class (the one that does not extend Application)
- Press OK, unless changed, this will create a META-INF in your resources folder
- Now press the + and add 'Directory Content'
- Pick your main resources folder, under /src/main/java/resources, and add it
- In the overview you will see the compile output, many libraries and your resources directory. The order matters, the upper items will override the lower. Click on the resource folder and turn off sorting, then move it to the top of the list with the up arrow.
- Now click apply and then click OK to close the window.
- Lastly build your jar by choosing: Build -> Build Artifacts... -> Build
  You should shortly after this get a jar in the /out/artifacts/{project-name}_jar/{project-name}.jar

Now you should have a fat jar which can run. (Remember to look at [Making it cross-platform](#making-it-cross-platform))

_(It seems like a bug that you have to add resources manually, since it is marked in Intellij and should be in the compiled output, but it was not for me. If you figure out how to fix this without adding the resource folder manually to the jar, do tell me.)_

## Make a fat jar WITHOUT modules

This is the old way of creating a fat jar file, for when your project does not use modules. To use this method you have to change your project to not use modules: Remove both the module-info.java file and the "mainModule = '{module-name}'" from the build.gradle, under application. (You can also remove the 'org.javamodularity.moduleplugin').

Now that you are not using modules you can add the following to build.gradle:

```java
jar {
  duplicatesStrategy = DuplicatesStrategy.EXCLUDE
  manifest {
    attributes "Main-Class": "org.example.nonmodularjar.Launcher"
  }

  from {
    configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
  }
}
```

Running `gradle build` with this will create a runnable jar in /build/libs. <br>
This step is also described well on: [Jetbrains - Getting started with gradle](https://www.jetbrains.com/help/idea/getting-started-with-gradle.html#deploy_gradle).

## Making it cross-platform

The jar files we have made include JavaFX for the platform you build them on, but won't work if you move them. We currently use the `javafxplugin` to automatically get libraries, but this does not allow you to add multiple platforms (as of 0.1.0). But we can manually add all the libraries, simply add this to the dependencies in build.gradle:

```gradle
dependencies {
  implementation group: 'org.openjfx', name: 'javafx-base', version: javaFxVersion, classifier: 'win'
  implementation group: 'org.openjfx', name: 'javafx-base', version: javaFxVersion, classifier: 'mac'
  implementation group: 'org.openjfx', name: 'javafx-base', version: javaFxVersion, classifier: 'linux'
  implementation group: 'org.openjfx', name: 'javafx-controls', version: javaFxVersion, classifier: 'win'
  implementation group: 'org.openjfx', name: 'javafx-controls', version: javaFxVersion, classifier: 'mac'
  implementation group: 'org.openjfx', name: 'javafx-controls', version: javaFxVersion, classifier: 'linux'
  implementation group: 'org.openjfx', name: 'javafx-fxml', version: javaFxVersion, classifier: 'win'
  implementation group: 'org.openjfx', name: 'javafx-fxml', version: javaFxVersion, classifier: 'mac'
  implementation group: 'org.openjfx', name: 'javafx-fxml', version: javaFxVersion, classifier: 'linux'
  implementation group: 'org.openjfx', name: 'javafx-graphics', version: javaFxVersion, classifier: 'win'
  implementation group: 'org.openjfx', name: 'javafx-graphics', version: javaFxVersion, classifier: 'mac'
  implementation group: 'org.openjfx', name: 'javafx-graphics', version: javaFxVersion, classifier: 'linux'
}
```

Also specify javaFxVersion in the ext part of build.gradle:

```gradle
ext {
  javaFxVersion = '17.0.6'
}
```

You should see the new libraries in the 'external libraries' part of the project/file explorer. New libraries are not automatically added to a artifact, so you will need to go back into artifacts and recreate the jar artifact, as in step [Make a jar with modules](#make-a-jar-with-modules).

</details>

# Running a jar
<details>
<summary>Again don't worry for now, but look if interested</summary>
You can run a jar file with `java -jar {jar-name}.jar` if you have java installed on your system. Otherwise you can run it through Intellij following this guide (The 'before launch' is under code coverage now):
https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-application.html#create_jar_run_config

## Common errors and solutions:

### Error: no main manifest attribute, in {name}.jar

- The Main-Class attribute in MANIFEST.MF is not set correctly. MANIFEST.MF should be in your resources folder, and the resource folder should be in the jar artifact at the top, do not sort the items.

### Error: JavaFX runtime components are missing, and are required to run this application

- Check that you are using a launcher class

### Error: "Caused by: java.lang.IllegalStateException: Location is not set."

- Check that resources are all in the jar

### Error: "Execution failed for task ':run'. ... java finished with non-zero exit value 1"

- Have you added the gradle code for the jar task to build a non modular jar? <br>
Check that you don't have 'mainModule' set under application. Otherwise gradle might think that you are still using modules.

### Warning: : Unsupported JavaFX configuration: classes were loaded from 'unnamed module...'
This is just a warning that we broke modularity by creating a fat jar. As far as i know this can't be avoided, so just ignore it.
</details>

# Debugging a jar
<details>
<summary>Expand</summary>

A jar file is just a zip file with a few required files. So if the jar can't run, just unzip it:
`unzip example.jar`

A correct jar file with JavaFX should have:

- A META-INF directory, with a MANIFEST.MF which specifies "Main-Class: {your-main-module}.{your-launcher-class}".
- A javafx directory with a bunch of things inside
- Your code as .class files, and resources, if fx my main class is at org.example.{project-name}.Launcher, then there should be a /org/example/{project-name} directory with a Launcher.class and all resources used.
</details>