Many content sources used by libraries cannot be integrated into the Library Simplified ecosystem because they don't have an API that meets our requirements.

## Biblioboard

Working on an ODL-based interface, but it's not ready yet.

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

All three of these sites are from Scholastic and combine ebooks with videos through a web view.

BookFlix is available at NYPL. Authentication is handled and content is delivered through EZProxy.

## hoopla

[hoopla](https://www.hoopladigital.com/) offers several media formats: ebooks, movies, music etc. They stream content to the patron's device via a custom app.

Patrons choose a library that offers hoopla service, then verify their membership in that library by providing barcode and (optional) PIN. Presumably this is done through an ILS integration. Apart from this ILS integration, hoopla offers no integrations to a library system.

## Interactive educational resources

Even if we can get the content, the experience these services provide probably won't fit into the Simplified app because the content they deliver is not in a container that we can reasonably render. In most cases the best we can possibly do is open a web view and transparently handle authentication.

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