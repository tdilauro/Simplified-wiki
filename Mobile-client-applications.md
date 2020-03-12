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

We use a mix of Swift and Objective-C when developing applications for iOS. New work should be done in Swift, whenever possible and appropriate. Instead of writing a significant amount of Objective-C code, we strongly prefer to write the same functionality in Swift, even if it costs a bit of refactoring work. 

When touching existing code do your best to match the code style of the file you're working on.

New Swift code should adhere to Apple's [Swift Style Guide](https://google.github.io/swift/).

Objective-C code should again follow [Apple's Conventions](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW1) and more in detail [Google's coding style](https://github.com/google/styleguide/blob/gh-pages/objcguide.md).

### Releasing

For releasing we follow the standard [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/) method of branching.

