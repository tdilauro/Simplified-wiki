Currently the `contributors` table is a battleground where many different data sources fight it out to produce the best possible data representation of a human being who wrote a book. The situation is similar to what used to happen with the `editions` table. There's no indication as to who created the data, so if the data is wrong, it's not clear which bit of the system is at fault.

This proposal turns the `contributors` table into a place for every data source to record any information it has about the people who worked on the books it knows about.

It introduces the concept of 'contributor equivalencies' (similar to 'identifier equivalencies'), a place for data sources to record their belief that two distinct Contributors refer to the same individual. 

It introduces the concept of a 'presentation contributor' (similar to 'presentation edition'), a place for the LS code itself to record its synthesis of all the available data into the best possible answer to the question: "who wrote this book and which other books did they write?"