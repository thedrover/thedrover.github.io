---
layout: post
title: Managing Android Resources in a Library Project
summary: Some simple tips to make your library project resources easy to use
categories: android sdk library
---

No matter what platform is used, it is important to make a library or SDK as easy to use as possible for developers who are integrating the SDK.

One simple trick is to use the Android tools to 


[xxx](https://www.youtube.com/watch?v=Y2GC6P5hPeA)

## Consistent Resource Prefix

It is common practice to use a consistent resource prefix to make it
easy to identify resources from a library and avoid clashes when
merging resoruces.

The Android Gradle plugin provides some support to enforce this.  This
feature was added in 2014 but not widely documented. 

```
android {
  ...other configuration...
  resourcePrefix 'mylib_'
}
```

Resources that do not have this prefix will be flagged as
 errors by Android lint. If for some reason a eresource needs to
 ignored. `tools:ignore` e.g. over-riding a resource from another
 library.

apply the Android Studio intentaion action

```
<string name="string_without_prefix" tools:ignore="ResourceName">Hello</string>
```

generated source. use lint.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<lint>
    <!-- Ignore generated resources that do not use the reource prefix -->
    <issue id="ResourceName">
        <ignore path="build"/>
    </issue>
</lint>
```

## Hide Internal Resources

 1. Declare public reources. `public.xml`. Warnings if not used....
