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

### Code style

TBD

### Releasing

TBD