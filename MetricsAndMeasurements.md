## Our goals and objectives

### Project Objectives
* Grow Users
* Increase Circulation of collection

#### Other Goals 
* Maximize the time patrons spend reading books they like - this is a proxy to directing them to content they want
* Minimize the number of unused loans - this can help circulation though greater availability
* Expand the user profile for ebooks - this can grow users

#### Usability Goals - how we will evaluate the app
The app is intuitive: How easy is it for users to accomplish basic tasks the first time they encounter the design?
The app is Efficient: Once users have learned the design, how quickly can they perform tasks?
The app is Memorable: When users return to the design after a period of not using it, how easily can they reestablish proficiency?
The app avoids user Errors: How many errors do users make, how severe are these errors, and how easily can they recover from the errors?
The has high Satisfaction: How pleasant is it to use the design?

## How licenses work

This is Leonard's impression of how licensing works:

* For most titles, we buy licenses that are both time-limited and use limited. That is, the license will expire after 26 loans or one year, whichever comes first. (_What are the actual numbers?_)
* We acquire licenses to some titles (_which ones?_) on a pay-per-read basis.
* For some titles, we acquire licenses for one title at a time. Other titles we acquire in bundle deals with the ebook provider. Our librarians may not even know which titles are available in these bundles.
* Overdrive and 3M have purchasing consoles that encourage the library to buy more licenses. This is done on 
the basis of the hold/license ratio (see below).

## Measurements

### Hold/license ratio

The ratio of active holds to active licenses. This is the metric used by 3M and Overdrive (_both of them?_) to drive license sales. When this ratio gets above 9 (_is that right? for both vendors?_) they tell us we should buy another license.

This is a terrible metric because it encourages buying a lot of licenses for things that are currently popular, licenses that are likely to expire before we get enough usage out of them to justify their cost.

### Unused loans

When a license expires, the number of times we could have loaned it out if it hadn't expired is the number of unused loans. Unused loans represent wasted money. It may have been better if we hadn't bought that license at all. Given that we did buy the license, we should have tried harder to lend out the book.

### Wait time

Wait time is the time between a patron entering the hold queue for a book, and being given the opportunity to check it out.

Minimizing wait time is important because large wait time leads to failures to lend (q.v.).

It's also very important that wait time be _predictable_, so that we can give patrons an accurate picture of how long it will take them to get a book.

### Reservation time

Reservation time is the time between a book being made available to someone and the time they decide to either check it out or pass it up.

If the user decides to pass it up, that's the worst. They wanted the book badly enough to get in the hold queue, but they don't want it now that it's available. Maybe they already read it some other way, maybe they're no longer interested, maybe they're still interested but they don't have time right now. However, it happened, they've bumped up the wait time for everyone behind them in the queue, and for nothing.

Minimizing reservation time is important because your reservation time contributes to the wait time of everyone in the queue behind you. If there is no one in the queue behind you, it's okay to give you more time to make up your mind.

### Loan time

The time between a patron checking out a book and checking it in. For genre fiction this may be very short--measured in hours. For literary fiction and serious fiction it is longer. 

Not to be confused with the _maximum_ loan time, which is the time (set by the ebook vendor) at which the DRM license will expire.

In general, we don't want to try to change loan time, because we want people to read at their own pace. The one exception is when the loan time is the same as the maximum loan time. This probably means that the patron didn't finish the book. They might not have even _started_ the book, in which case it would have been better if we hadn't loaned them the book at all. It cost us money, it drastically increased the wait time of everyone in the hold queue, and the patron didn't get anything out of it.

For public domain books, "loan time" is not a coherent concept. However, we still don't want to loan out books that a patron will never read, because it uses bandwidth and makes the patron feel guilty.

### Idle time

Idle time is the time between one patron checking a book in and the next patron reserving it or checking it out.

For a book with active holds, idle time is zero. The book is reserved for the next person in the queue immediately after it's checked in.

Minimizing idle time is important because we want someone to be reading the book all the time. The only books with large idle times should be books with niche audiences.

### Failure to lend

A failure to lend is when someone joins the queue for a book and then leaves the queue without checking out the book. They either decided they didn't want the book, or they read the book some other way. We don't count people who are still in the queue, or people who left the queue via checking out the book.

On 3M as of 2014-04-21:

* 7225 books had no failures to lend.
* 6301 books had failures to lend.
* Among books with failures to lend, the median failure rate was 18%. That is, 18% of the people who ever joined the queue have now left the queue without checking out the book.

This includes people who entered the queue, went all the way through the queue, were notified that the book was available, and decided not to check it out. Because they made the decision in response to a reservation notification, these patrons bump up the failure-to-lend rate _and_ the reservation time.

We can minimize failure to lend by buying more licenses (the vendors' prefered solution), and by decreasing wait time (serving patrons more quickly). 

Giving accurate up-front estimates of wait time will bring this measurement down, because it will discourage patrons from joining a queue they can't wait through. However, someone who wants the book and doesn't join the queue because it's too long is still technically a "failure to lend". The benefits to the system of giving an up-front estimate of wait time are:

* We've front-loaded the patron's disappointment, instead of making them frustrated at a long wait, then bothering them with a notification long after they're forgotten the book or read it some other way.
* The patron's decision not to join the queue decreases the wait time (by the length of the reservation time) for everyone who joins the queue in the future.

### Public domain loans

Loans of public domain books from Gutenberg are free, and the patron gets to keep the book. Loans of public domain books from 3M and Overdrive are none of these things.

#### How many loans are for public domain books?

I did a very basic title/author match to try to find books we own in Overdrive and 3M that are also in Gutenberg.

On Overdrive we own between 130 and 900 licenses for between 120 and 380 public domain books. There are also a few books (e.g. Frankenstein) for which we own thousands of licenses, which kind of simulates the public domain thing. This is out of about 16000 titles.

On 3M we own between 100 and 450 licenses for between 90 and 350 public domain books. This is out of a library size of about 14500 titles.

So the overlap is real, but it's not huge. I haven't determined how often these books get loaned out.
