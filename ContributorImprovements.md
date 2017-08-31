Currently the `contributors` table is a battleground where many different data sources fight it out to produce the best possible data representation of a human being who wrote a book. The situation is similar to what used to happen with the `editions` table. There's no indication as to who created the data, so if the data is wrong, it's not clear which bit of the system is at fault.

This proposal turns the `contributors` table into a place for every data source to record any information it has about the people who worked on the books it knows about.

It introduces the concept of 'contributor equivalencies' (similar to 'identifier equivalencies'), a place for data sources to record their belief that two distinct Contributors refer to the same individual. 

It introduces the concept of a 'presentation contributor' (similar to 'presentation edition'), a place for the LS code itself to record its synthesis of all the available data into the best possible answer to the question: "who wrote this book and which other books did they write?"

# Add `data_source_id` to `Contributor`

This stops data sources from stepping on each others' toes. If you have something to say about a person, create a `Contributor` object containing the information, and tag it with the `id` of the `DataSource` that provided the information.

# Add `is_presentation` to `Contributor`

A presentation contributor represents the server's attempt to consolidate data from multiple sources, into an all-round view of an individual. This includes making the determination that, e.g. "Alice McPherson", "Alice Mc Pherson", "Alice Mcpherson", "Alice MacPherson", "Alice Mac Pherson", and "Alice Boggs" (maiden name) are all the same person.

A work's presentation edition will only reference presentation contributors. The `WorkContributor` join table will only reference presentation contributors.