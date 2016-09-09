SirsiDynix Symphony is a proprietary ILS.

Symphony can be integrated with a Library Simplified circulation manage through [[SIP]]. Symphony also has an API, but access to the documentation [requires a subscription](http://www.sirsidynix.com/files/pdf/SirsiDynix_Symphony_API_Overview.pdf) so I don't know if it provides what we need. However I can glean a few details from records of other people's clients for the Symphony API, and it looks more or less like what we need.

* [This Github repository](https://github.com/mark-cooper/sirsidynix/blob/master/lib/sirsidynix.rb) shows the existence of a `loginUser` method which takes arguments `login` and `password`, and a `lookupMyAccountInfo` method which gets patron info.
* [This Code4Lib Journal article](http://journal.code4lib.org/articles/4810) gives some examples of these methods in use.