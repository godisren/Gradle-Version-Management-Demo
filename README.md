# Introduction

This repo shows how to manage central dependencies with specific version via Gradle [version catalog](https://docs.gradle.org/current/userguide/platforms.html#sub:central-declaration-of-dependencies),
and share with other Gradle projects.

# How to use it
The file `libs.versions.toml` defines all dependency versions such as plugin and libraries.
```properties
# libs.versions.toml

[versions]
groovy = "3.0.5"

[libraries]
commons-lang3 = { group = "org.apache.commons", name = "commons-lang3", version = "3.9" }
groovy-core = { module = "org.codehaus.groovy:groovy", version.ref = "groovy" }
groovy-json = { module = "org.codehaus.groovy:groovy-json", version.ref = "groovy" }
groovy-nio = { module = "org.codehaus.groovy:groovy-nio", version.ref = "groovy" }

[bundles]
groovy = ["groovy-core", "groovy-json", "groovy-nio"]

[plugins]
springboot = { id = "org.springframework.boot", version = "3.0.5" }
springboot-dependency-management = { id = "io.spring.dependency-management", version = "1.0.15.RELEASE" }
```


We can set its file path in `settings.gradle` under other projects. In addition, the file `toml` could be published to maven repository.
The other projects can refer to it from remote repository. By this way, 
we have a central place to manage the library dependencies.
More detail refers to [here](https://docs.gradle.org/current/userguide/platforms.html#sec:version-catalog-plugin).
```groovy
# settings.gradle

dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from(files("./libs.versions.toml"))
        }
    }
}
```
Then, declare the libs we'd like to used from `libs.versions.toml` in our project `build.gradle`.
It supports plugins and dependencies.

```groovy
# build.gradle

plugins {
    alias(libs.plugins.springboot)
    alias(libs.plugins.springboot.dependency.management)
}
...
dependencies {
    ...
    implementation(libs.commons.lang3)
    implementation(libs.bundles.groovy)

}
```
According to `libs.versions.toml` definition, Above setting, it equals to :
```groovy
plugins {
    id 'org.springframework.boot' version '3.0.5'
    id 'io.spring.dependency-management' version '1.1.0'
}
...
dependencies {
    ...
    implementation 'org.apache.commons:commons-lang3:3.9'
    implementation 'org.codehaus.groovy:groovy:3.0.5'
    implementation 'org.codehaus.groovy:groovy-json:3.0.5'
    implementation 'org.codehaus.groovy:groovy-nio:3.0.5'

}
```
That's it, we've finished the central library version management. 

# Use case : Upgrading library version
If we want to upgrade the library version, for example, upgrade springboot to `3.0.5` from `2.7.10`.

1. Print the current dependencies, check the springboot version
```shell
gradlew dependencies --configuration runtimeClasspath

# output 
runtimeClasspath - Runtime classpath of source set 'main'.
+--- org.springframework.boot:spring-boot-starter-web -> 2.7.10
|    +--- ...
```

2. Update springboot version to `3.0.5` in `libs.versions.toml`
```groovy
...

[plugins]
springboot = { id = "org.springframework.boot", version = "3.0.5" }
...
```

3. Refresh the project, and check the version again. We will see it is the new version.
```shell
gradlew dependencies  --configuration runtimeClasspath

# output 
runtimeClasspath - Runtime classpath of source set 'main'.
+--- org.springframework.boot:spring-boot-starter-web -> 3.0.5
|    +--- ...
```