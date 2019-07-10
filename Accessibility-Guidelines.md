# Accessibility Guidelines
## Web Content Accessibility Standard
Web Content Accessibility Guidelines (WCAG) is an international digital accessibility standard. It is technology agnostic and can be applied to any digital technology from the web to PDFs to native apps. There are three levels A, AA, AAA. Most organizations meet level AA. (For example, this is the standard used by NYPL, Library of Congress and various governments including USA Federal. The standard organized by 4 principles each with a set of guidelines. Guidelines are defined by success criteria, which are concrete. WCAG 2.1 is the most current version.

This wiki page includes WCAG requirements and best practices that can be applied to the work on the SimplyE Web Client. For the most part, it covers relevant HTML patterns supporting accessible functionality. There are also some notes on visual design.

## General Guidelines
### Color Contrast
* Text color should have a contrast ratio of 4.5:1 with background color.
* There are a number of free tools for checking contrast, including the [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) to check contrasts.

### Visible Focus
* Visible focus should not be suppressed.
* Visible focus can be increased by upping the contrast of the styling to a 3:1 ratio with the background.

### Fonts and Legibility
* Sans-serif fonts are usually preferred so that’s covered. Sixteen point font is a good reading size.
* All caps can lessen legibility (we read word shapes rather than individual letters). Instead of using all caps or small caps, consider using title case.

### Metadata
#### Title
* Each webpage should have a unique and descriptive `<title>` element.
* Typically the title will be the same as the page’s `h1`; though with a title, you may want to add branding at the end. For example: Search Results - Open eBooks.

#### Language
* Every webpage should have an HTML `<lang>`. (Otherwise, may get strange screenreader behavior.)

### Images
* Images (`<img>`) must have alt text (`<alt>` attribute). 
* With linked images, the alt text should indicate what is the link target.
* For more on alt text: [WebAIM has a tutorial on alt text.](https://webaim.org/techniques/alttext/)

## Page Structure and Semantics  
### Landmark Regions
* All the content on the page should belong to a landmark regions. 
* Landmark regions identify sections of a page, like the header, navigation, and main. 
* Landmarks are created by HTML5 elements, sometimes supplemented with aria `role` attributes. 
* By adding an `aria-label` attribute to a landmark element, you can add a unique name. When there are multiples of the same region type, always include a name.
* Screenreader use landmarks to navigate the page and skip to sections. When a screenreader user enters a region, landmark type will be announced and the unique name (if exists).
* The [Web Accessibility Tutorials has a good tutorial](https://www.w3.org/WAI/tutorials/page-structure/regions/).

### Navigation
* Navigation should be wrapped in a <nav> element.
* When there are multiple navigation elements on a page, they should be assigned a unique name.

###  Headings
Every page must * have an h1. This should be the title of the page. 
* Headings need to logically organize the page.
* Headings must be semantic (HTML elements h1-h6) rather than simply visual.
* Headings should be sequential. You can follow an h1 with an h2 or an h3 with an h4, but you can never follow an h1 with an h4. (It is however okay to follow an h4 with an h2.)
* Reference: [WAI Tutorial on Page Structure + Headings](https://www.w3.org/WAI/tutorials/page-structure/headings/)
* Reference: [WebAIM article on Semantic Structure](https://webaim.org/techniques/semanticstructure/)

### Lists
* Use semantic lists to organize information.
* Organize metadata in definition lists, `dl`.

### Reading Order
* Make sure that the HTML order is in the reading order you expect. Since this is the order a screenreader user or keyboard navigator will encounter the page. 
* Remember to group things logically. Take an entry for a book as an example. Let's say there is an image, heading (that is the book title), and then some metadata. Even though the image may visually be to the left of the rest of the information, it should follow the heading in page order, since it is organized by the heading.

### Keyboard Navigation
* App should be fully navigable and operational with a keyboard.

### Skip Links
* A skip link should be the first element on each page. It should link to the main content of the page with link text like “Skip navigation” or “Skip to main content”. The text target can be the h1 or the <main> element. Whichever is the target, add a tab-index of “-1”.
* The link text can be invisible, but best practice is for it to be visible on focus; you can find an example on nypl.org by tabbing through the page.

## Dynamic Page Change Guidelines
* When using JavaScript to change the content of a webpage, we need to make sure that information is available and announced to screenreaders. How we achieve this can change based on the functional design of the page. One technique is to use an `aria-live` attribute to announce changes. When using this technique follow best practices and test with screenreader and browser combinations.

## Component Guidelines 
The [United States Web Design System](https://designsystem.digital.gov/) includes patterns for a number of components. It is a fantastic reference for the accessible way of building and labeling components, including form fields. (Note, you can use the mark-up patterns without using any of the rest of the framework, including the visual styles and naming conventions.) Follow these patterns for:
* Inputs
* Dropdowns
* Buttons
* Radio-buttons

Note, when there are multiple buttons with the same label on a page, use an `aria-label` or `aria-labelledby` to create unique, descriptive labels for screenreaders. For example, on a page with multiple “Return” or “Borrow” buttons, use aria attributes to pull in the item title. For instance, "Return The Very Hungry Caterpillar".