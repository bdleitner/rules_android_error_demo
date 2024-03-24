# Demo for `rules_android` issue:

Working on moving my code to bzlmod from "straight" bazel, I ran into a problem where `android_sdk_repository` is no longer supported.

Trying to find the equivalent I found the `rules_android` module in the central repo.
The documentation on the github repo for `rules_android` suggested the correct thing to do was:

```
# WORKSPACE.bzlmod
load("@rules_android//rules:rules.bzl", "android_sdk_repository")
android_sdk_repository(
    name = "androidsdk",
)
```

and
```
# MODULE.bazel

bazel_dep(name = "rules_java", version = "7.4.0")
bazel_dep(name = "bazel_skylib", version = "1.3.0")

bazel_dep(
    name = "rules_android",
    version = "0.2.0",
)

# Or a later commit
RULES_ANDROID_COMMIT = "0bf3093bd011acd35de3c479c8990dd630d552aa"
git_override(
    module_name = "rules_android",
    remote = "https://github.com/bazelbuild/rules_android",
    commit = RULES_ANDROID_COMMIT,
)

register_toolchains(
    "@rules_android//toolchains/android:android_default_toolchain",
    "@rules_android//toolchains/android_sdk:android_sdk_tools",
)
```

and finally using:
```
load("@rules_android//rules:rules.bzl", "android_binary", "android_library")
android_binary(
    ...
)

android_library(
   ...
)
```

To use the rules.

The given commit didn't work due to a naming mismatch apparently fixed a few months ago so grabbing
the hash of the latest commit I got the `MODULE.bazel` entry I have in this repo.
This allows `bazel sync` to run, but the simple command:

```
bazel build java/com/bdl/demo/rules_android:all
```

gives the error:

```
ERROR: Skipping 'java/com/bdl/demo/rules_android:all': while parsing 'java/com/bdl/demo/rules_android:all': error loading package 'java/com/bdl/demo/r
ules_android': Unable to find package for @@rules_android//rules:rules.bzl: The repository '@@rules_android' could not be resolved: Repository '@@rule
s_android' is not defined.
ERROR: while parsing 'java/com/bdl/demo/rules_android:all': error loading package 'java/com/bdl/demo/rules_android': Unable to find package for @@rule
s_android//rules:rules.bzl: The repository '@@rules_android' could not be resolved: Repository '@@rules_android' is not defined.
```

So `bazel sync` works, but the repo doesn't seem to exist:

```
bazel query @rules_java//...
```

Gives a long list of targets, but 

```
bazel query @rules_android//...
```

Gives:

```
ERROR: no such package '@@rules_android//': The repository '@@rules_android' could not be resolved: Repository '@@rules_android' is not defined
```