Many content sources used by libraries cannot be integrated into the Library Simplified ecosystem because they don't have an API that meets our requirements.

## Biblioboard
## Gale Virtual Reference
## Grey House
## Capstone
## Marshall Cavendish
## Salem Press

## IndieFlix

Videos. No custom app.

## Zinio

Magazines. Custom app, no API.

## Comics Plus

Comics. Custom app, no API.

### BookFlix/Science Flix/TrueFlix

from Scholastic.


## No API, custom app

A service that provides a custom app must have some kind of API--that's how the app talks to the server. In a pinch we might be able to negotiate the circulation manager's access to this same API.

## hoopla

[hoopla](https://www.hoopladigital.com/) offers several media formats: ebooks, movies, music etc. They stream content to the patron's device via a custom app.

Patrons choose a library that offers hoopla service, then verify their membership in that library by providing barcode and (optional) PIN. Presumably this is done through an ILS integration. Apart from this ILS integration, hoopla offers no integrations to a library.

## Interactive educational software

These services won't fit into an OPDS-based app because the content they deliver is not in a container that we can reasonably render.

### Lynda.com

Lynda.com offers [an API](https://www.lynda.com/integration) that includes authorization hooks and collection information delivered through MARC records. So we could probably show lists of courses in OPDS, and send the patron to a web view once they chose a course.

### Transparent Languages

Custom app, no apparent API.

### Learning Express

Available at NYPL. Authentication happens through and content is served through EZProxy. No visible or advertised API.

### Mango Languages

Available at NYPL. Authentication happens through and content is served through EZProxy. No visible or advertised API.

### Universal Class

No visible or advertised API. [Sales page](http://company.universalclass.com/for-libraries.htm) has few details.