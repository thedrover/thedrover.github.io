---
layout: post
title: Multi-module Unit Test Support with Gradle
date:       2016-07-06
categories: Android Gradle testing
---

When writing tests for multi-module projects there may be some test
helper classes or base test classes that you want to use in more
than one module. One way to do this is to create a separate project
for the re-usable test classes. The disadvantage is that the test code is separated from the source code 
and tests may not be executed or maintained. This post describes an
easy way to create and use test classes for Android projects using
Gradle and Android studio without having to create a separate project for the test classes.

It follows the approach of Maven where a [test JAR](http://maven.apache.org/plugins/maven-jar-plugin/examples/create-test-jar.html)
can be created and used by adding some simple config. The Gradle
Android plugin doesn't package a JAR of test classes so we need to
create a task to create the test JAR before using it.


Consider the following module structure. `ProviderModule` contains a some test helper classes that are used in
the unit tests in `ProviderModule/src/test/java`.  `DependentModule` uses test classes from `ProviderModule` in its
own unit tests in `DependentModule/src/test/java`.

```
project
│   build.gradle   
│
└───ProviderModule
    │
    ├───build.gradle
        │   
        ├───src/main/java
        ├───src/test/java
        └───src/androidTest/java
        
└───DependentModule
    │
    ├───build.gradle
        │ 
        ├───src/main/java
        ├───src/test/java
        └───src/androidTest/java
           
```

 The following steps show how to re-use test helper classes or base classes in the unit test 
 source tree of the ProviderModule. It is also possible to re-use code in the instrumentation test source tree using a similar approach.
 
1. Define a task in the `allProjects{}` section of the top-level
module `build.gradle` that can be used in any sub-module to create a test JAR for
that package. This assumes the location of the test class output
directory. A property (`testJarLocation`) that points to the test JAR location is
created.

 
        task packageUnitTests(type: Jar) {
          from file("${buildDir}/intermediates/classes/test/debug/")
          archiveName "${project.name}-tests.jar"
          // dstinationDir null so use explicit property
          ext.testJarLocation="${project.buildDir}/libs/${archivePath}"
          description 'Package test JAR for sharing'
          doLast {
            println "Packaged test JAR for sharing: ${testJarLocation}"
          }
        }

2. Configure the packaging task to run in the `ProviderModule` after the test code has been compiled. This is added to the outermost scope of the
`build.gradle` file.

         tasks.whenTaskAdded { task ->
           if (task.name == 'compileDebugUnitTestSources') {
             task.finalizedBy packageTests
           }
         }

3. Add the test dependencies to the `DependentModule` by referencing the test JAR location
property of the desired project.

        dependencies {
    
          ...other dependencies...

          // If the test code from ProviderModule is to be used in unit tests
          testCompile files(project(':ProviderModule').packageUnitTests.testJarLocation)
          
          // If the test code from ProviderModule is to be used in instrumentation tests
          androidTestCompile files(project(':ProviderModule').packageUnitTests.testJarLocation)
        }
4. Ensure the `ProviderModule` test JAR is created before `DependentModule` attempts to use it

        tasks.whenTaskAdded { task ->
          if (task.name == 'compileDebugUnitTestSources') {
              task.dependsOn ":ProviderModule:packageUnitTests"
          }
        } 

   
A simple project that demonstrates this technique using trivial tests is available at 
[https://github.com/thedrover/multi-module-test-support.git](https://github.com/thedrover/multi-module-test-support.git).

