When a user loads up a list of libraries from the library registry (as opposed to searching for a specific library) we want to show them the libraries most likely to be useful to them.

We know a number of things for each user:

* Where on the planet they're located (roughly, via IP address).
* The language used on their device.
* Whether their device has accessibility settings turned on.

We know a number of things for each library:

* The service area: People living in this area may use the library.
* The focus area (a.k.a the jurisdiction): People living _outside_ this area probably have a library they should use instead of this one. By default, the focus area is the same as the service area.
* The approximate size of the library's collection in each language. (We don't know this for every library, but it will be available for all libraries that use the circulation manager.)
* The intended audience. (the general public, people with print disabilities, academics, etc).

# Sample libraries

Here are a number of libraries to motivate the discussion. The numbers are made up, and in some cases, the libraries are also made up.

* The New York Public Library
  * Service area: New York State
  * Focus area: Bronx County + New York County + Richmond County
  * Collection size: 150k English, 20k Spanish, 5k Russian
  * Intended audience: general public

* Brooklyn Public Library:
  * Service area: New York State
  * Focus area: Brooklyn, NY
  * Collection size: 75k English, 10k Spanish

* Albany Public Library
  * Service area: Albany, NY
  * Focus area: Albany, NY
  * Collection size: 50k English, 5k Spanish
  * Intended audience: general public

* NYU Library
  * Service area: New York City
  * Focus area: New York City
  * Collection size: 100K English
  * Intended audience: secondary education (specifically, NYU students)

* NYU Press
  * Service area: everywhere
  * Focus area: New York City
  * Collection size: 40 English
  * Intended audience: general public (but really, interest is probably limited to academics)

* UNM Press
  * Service area: everywhere
  * Focus Area: New Mexico
  * Collection size: 60 English, 10 Spanish
  * Intended audience: general public (but really, interest is probably limited to academics)

* BARD
  * Service area: the United States
  * Focus area: the United States
  * Collection size: 100K english
  * Intended audience: people with print disabilities

* The Internet Archive
  * Service area: everywhere
  * Focus area: everywhere
  * Collection size: 10M English, at least 1k in every other major language
  * Intended audience: general public

# Sample scenarios

These scenarios assume the libraries listed above are the only ones in the system. For each scenario, I list the libraries I would expect to see in the default view, in the order they should be presented.

* In Manhattan: NYPL, Brooklyn, Internet Archive, NYU Press
* In Brooklyn: Brooklyn, NYPL, Internet Archive, NYU Press
* In Queens: NYPL, Brooklyn, Internet Archive, NYU Press
* In Albany: Albany, NYPL, Brooklyn, Internet Archive, NYU Press
* 200 km from Albany: NYPL, Brooklyn, Internet Archive, NYU Press
* In New Jersey: NYPL, Brooklyn, Internet Archive, NYU Press
* Las Cruces, NM: UNM Press, Internet Archive
* In Albany, Russian speaker: NYPL, Albany, Internet Archive
* In Manhattan, Spanish speaker: NYPL, Internet Archive, UNM Press
* In Manhattan, with accessibility features turned on: NYPL, Brooklyn, BARD, Internet Archive, NYU PRess

These may not reflect actual user preferences, and even if they do we don't have to get this exactly right, but the algorithm we come up with should be able to make reasonable decisions in these cases.

# Proposal

The search algorithm should initially filter out libraries that the user is not eligible for, such as a library with an audience of people with print disabilities for a user who doesn't have accessibility features on. We may also want to filter out libraries when the user is outside their service area, but while we still have a relatively small number of libraries in the system, I think it's okay to show libraries that have a service area near the user.

The search algorithm can then compute scores to rank the remaining libraries in the search results.

The score for each library can increase or decrease based on the following factors:
* Audience: if the user has accessibility features turned on and the library's audience is people with print disabilities, the score is increased.
* Size of collection in the user's language: The score will decrease as the collection size gets smaller, relative to the maximum collection of any library in that language.
* Distance from the focus area: The score will decrease as the user gets farther from the focus area.
* Distance from the service area: The score will decrease as the user gets farther from the service area, but  by a smaller amount than the focus area.
* Size of the focus area: The score will decrease as the library's focus area gets larger.

The influence of each of these factors will need to be adjusted based on testing the algorithm with real data.

Here's some sample code I wrote to play around with this: https://gist.github.com/aslagle/d6f9d8c4cb2d437914572db372c562b5.
It handles most of the scenarios well, except the ones with other languages.