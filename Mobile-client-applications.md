Library Simplified is a collection of middleware, server software and mobile client applications for iOS and Android.

The pages below document various best practices and workflows important for anyone contributing to the mobile client applications. What's considered a best practice will no doubt evolve over time, as should this document.

But first, a bit of wisdom.

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is the most important.
>
> However, know when to be inconsistent -- sometimes style guide recommendations just aren't applicable. When in doubt, use your best judgment. Look at other examples and decide what looks best. And don't hesitate to ask! &mdash; [PEP 8 -- Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/#a-foolish-consistency-is-the-hobgoblin-of-little-minds)

## Git workflow

Historically, we've used [git flow](https://nvie.com/posts/a-successful-git-branching-model/) as our basis for branching and tagging releases.

We recommend installing [Git Flow AVH Edition](https://github.com/petervanderdoes/gitflow-avh) to automate some of the work of branching and tagging. Using `gitflow-avh` is not required, but by automating the underlying repository operations, it eliminates the possibility of making mistakes, and keeps the various branches consistent.

## Android app
- https://github.com/NYPL-Simplified/Simplified-Android-Core
- https://github.com/NYPL-Simplified/Simplified-Android-SimplyE

### Code style

We use a mix of Java and Kotlin when developing applications for Android. New work should generally be done in Kotlin, where appropriate. When touching existing code do your best to match the code style of the file you're working on.

New Kotlin code should adhere to the [Kotlin Coding Conventions](https://google.github.io/styleguide/javaguide.html) and [Android style guide](https://developer.android.com/kotlin/style-guide).

We use a tool called [ktlint](https://github.com/pinterest/ktlint) to enforce a consistent code style. You can run this check against your code using the command `./gradlew ktlint`.

The [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) is a good reference when writing new Java code.

#### Books we recommend
- [Effective Java](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
- [Kotlin in Action](https://www.manning.com/books/kotlin-in-action)

### Releasing

TBD

## iOS app
- https://github.com/NYPL-Simplified/Simplified-iOS

### Dependent frameworks that we maintain
- https://github.com/NYPL-Simplified/NYPLAEToolkit
- https://github.com/NYPL-Simplified/NYPLAudiobookToolkit
- https://github.com/NYPL-Simplified/CardCreator-iOS
- https://github.com/NYPL-Simplified/PDFRendererProvider-iOS
- https://github.com/NYPL-Simplified/DRM-iOS-Adobe
- https://github.com/NYPL-Simplified/Adobe-Content-Filter
- https://github.com/NYPL/tenprintcover-ios

### Code style

We use a mix of Swift and Objective-C when developing applications for iOS. New work should be done in Swift, whenever possible and appropriate. For example, if you need to write a significant amount of Objective-C code to fix a bug or extend a feature, we strongly prefer to write the same functionality in Swift, even if it costs a bit of refactoring work. 

When touching existing code do your best to match the code style of the file you're working on.

New Swift code should adhere to Apple's [Swift Style Guide](https://google.github.io/swift/).

Objective-C code should again follow [Apple's Conventions](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW1) and more in detail [Google's coding style](https://github.com/google/styleguide/blob/gh-pages/objcguide.md).

### Releasing

For releasing we follow the standard [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/) method of branching.

Currently we do not have CI for SimplyE on iOS, although it's something that is desired and has been discussed for the future. In the mean time we follow these steps for distributing / releasing a new version of SimplyE:

1. Clone the [official Simplified-iOS repo](https://github.com/NYPL-Simplified/Simplified-iOS/) and follow the [instructions for building third party dependencies](https://github.com/NYPL-Simplified/Simplified-iOS/blob/develop/README.md).
2. `git checkout develop` # new release/RC builds always start from `develop`, per Git Flow.
3. If you see changes related to any submodule pointing to a checkin different, run `git submodule update --init`
4. If you're preparing a new release, create a new release branch, e.g. `git checkout -b release/3.4.0`
5. If you have any changes related to cURL, OpenSSL, Adobe DRM SDK, run `./build-openssl-curl.sh`
6. `./build-carthage.sh Debug`
7. Verify your Certificates repo is up-to-date and then run `./update-certificates.sh`
8. `open Simplified.xcodeproj`
9. Verify the version and build number are correct.
10. Create a new Archive.
11. Verify there are no new warnings: if you do see them, either file a ticket or fix them in a PR.
12. In Xcode Organizer, Validate the app and if all goes well Distribute it.
13. For QA builds:
- Choose Ad-Hoc Distribution.
- Choose no App Thinning, check Rebuild from Bitcode and Strip Swift symbols.
- Select the Distribution certificate and the NYPL_AdHoc_Wildcard provisioning profile.
- Export to disk.
- Drag .ipa to https://console.firebase.google.com/project/simplye-nypl/appdistribution/app/ios:org.nypl.labs.SimplyE/releases
- Add internal QA team member(s). Usually this is a QA engineer that has requested a build, but if unsure ask your team lead who should be added. DO NOT add anyone else available in Firebase without checking in with Product team first.
- Add release notes. For QA team this can probably be a list of tickets / features. For a wider release, check in with Product.
14. For TestFlight builds:
- Choose App Store Connect.
- Choose no App Thinning, check Rebuild from Bitcode and Strip Swift symbols. 
- Select Distribution certificate and follow the prompts to upload to TestFlight.
- For release notes, check in with Product to gather the approved release notes to add to TestFlight.
