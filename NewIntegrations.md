The goal of the Library Simplified circulation manager is to tie
together many pieces of software: ILS systems for authenticating
patrons, license sources for borrowing books, search interfaces for
finding books, analytics tools for reporting on usage, and so on.

When you're seting up a circulation manager for a library, all the
currently supported integrations can be configured through the
administrative interface. When you're adding _a brand new
integration_, you'll need to go down a level and write some code.

* [[NewPatronAuthenticationIntegration]](A new ILS or other patron authentication technique)
* [[NewLicenseIntegration]](A new source of items for patrons to borrow)

