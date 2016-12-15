The original Library Simplified team has significant experience connecting circulation managers to III Sierra ILSes, since both the New York Public Library and the Brooklyn Public Library use Sierra as their ILS system. We've identified a number of pitfalls that can make it difficult to even know which mechanism to use.

Sierra offers two APIs that can be used to connect a circulation manager to a Sierra installation:
 - The [[Millenium Patron API|Millenium]]
 - [[SIP2|SIP]]

Sierra also offers (at least) two APIs that _cannot_ be used to connect a circulation manager to a Sierra installation:
 - The [["Sierra REST API"|Sierra-REST-API]] does not allow a circulation manager to represent a patron to a third party (such as Overdrive).
 - The "PatronIO" SOAP API is useful for _creating_ patron records. It is not useful for validating patron credentials.

If you have created a custom SOAP API for some other purpose, it's theoretically possible to connect a circulation manager to your ILS through that API, but you'd have to write code to talk to that custom API, and there's probably a better way.

The names can get confusing. The Millenium Patron API is a "REST API" provided by Sierra, but it is not the same as the product called "The Sierra REST API". The PatronIO SOAP API is an API for dealing with patrons, but it is not the product called the "Millenium Patron API". The Millenium Patron API is not "The III Patron Update Service" or "The My Millenium Service".

When in doubt, file a support request with Overdrive (assuming you have an Overdrive account) and ask them how Overdrive communicates with your ILS to provide ebook service.

Once you've determined (or decided) how the Library Simplified circulation manager should communicate with your ILS, follow the instructions for that communication mechanism:

 - The [[Millenium Patron API|Millenium]]
 - [[SIP2|SIP]]