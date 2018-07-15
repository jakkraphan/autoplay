# Autoplay
Gradle plugin for publishing Android artifacts to Google Play.

# Fetures

- Autoplay is optimized for CI/CD usage:
  - it does **not** trigger full project rebuild - you can reuse build artifacts from previous build step.
  - it accepts JSON key as **base64-encoded string**, which is a secure and convenient way of providing secret data.
  
- Autoplay is developer friendly:
  - it does **not** fail the build after build-script evaluation, but only when a publish-task is actually called.
  - it does **not** require storing any dummy keys in source control.
  - it has just a single publish task for uploading akp, mapping file and release notes.
  
- Autoplay is reliable and future-proof:
  - it has very clean and concise implementation, which makes it easy to mantainable and to extensible.
  - it is built using latest technologies and tools like Kotlin, Kotlin-DSL for Gradle, Android Gradle plugin etc.
  
# Usage

In the main `build.gradle`

```kotlin
buildscript {

  repositories {
    google()
    jcenter()  
  }
  
  dependencies {
    classpath "de.halfbit:android-autoplay:<version>"
  }
}
```

In the application module's `build.gradle`

```kotlin
apply plugin: 'com.android.application'
apply plugin: 'android-autoplay'

autoplay {
    track "internal"
    secretJsonBase64 project.hasProperty('SECRET_JSON') ? project.property('SECRET_JSON') : ''
}
```

Call `./gradlew tasks` and you will see a new publishing task `publishApk<BuildVariant>` in the list. Autoplay adds this task for each `release` build variant. For a project witout build variants configured, the task is called `publishApkRelease`.

Now you can call this task from a central build script. Here is an example of how to use it with Gitlab CI.

```yml
...

stages:
  - build
  - release

assemble:
  stage: build
  only:
    - master
  script:
    - ./gradlew clean assembleRelease -PSTORE_PASS=${STORE_PASS} -PKEY_PASS=${KEY_PASS}
  artifacts:
    paths:
      - app/build/outputs/

release:
  stage: release
  dependencies:
    - assemble
  only:
    - master
  script:
    - ./gradlew publishApkRelease -PSECRET_JSON=${SECRET_JSON}
    

```

Happy coding and enjoy!
