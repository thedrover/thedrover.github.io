---
layout: post
title: Multi-module Test Support with Gradle
date:       2016-05-28
categories: Android Gradle testing
---

When writing tests for multi-module projects there may be some test
helper classes, or base test classes, that you want to re-used in more than
one module. This post describes an easy way to do this for Android
projects using Gradle and Android studio.

It follows the approach of Maven where a [test JAR] the
box](http://maven.apache.org/plugins/maven-jar-plugin/examples/create-test-jar.html)
can be created and used by adding some simple config. The Gradle
Android pluging doesn't package a JAR of test classes so we need to
create a task to create the test JAR before using it.


Let's use the following module structure as a concrete
example. ModuleA contains a some test helper classes that are used in
the tests for ModuleA.  ModuleB uses test classes from ModuleA in its
own tests.

```
project
│   build.gradle   
│
└───ModuleA
    │
    ├───build.gradle
        │   
        └───src...
└───ModuleB
    │
    ├───build.gradle
        │ 
        └───src...
           
  
```

 
1. Define a task in the `allProjects{}` section of the top-level
module `build.gradle` that can be used in any sub-module to create a test JAR for
that package. This assumes the location of the test class output
directory. A property that points to the test JAR location is
created. Android specific.

 
        task packageTests(type: Jar) {
          from file("${buildDir}/intermediates/classes/test/debug/")
          archiveName "${project.name}-tests.jar"
         // dstinationDir null so use explicit property
         ext.testJarLocation="${project.buildDir}/libs/${archivePath}"
         description 'Package test JAR for sharing'
         doLast {
           println "Packaged test JAR for sharing: ${testJarLocation}"
         }
        }

2. Configure the packing task in each module that has test code that
will be shared. This is added to the outermost scope of the
`build.gradle` file.

         tasks.whenTaskAdded { task ->
           if (task.name == 'compileDebugUnitTestSources') {
             task.finalizedBy packageTests
           }
         }

3. Add the test dependencies by referencing the test JAR location
property of the desired project

        dependencies {
    
          ...other dependencies...

          testCompile files(project(':ModuleA').packageTests.testJarLocation)
        }
