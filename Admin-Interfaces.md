# Overview
Of the three core elements, the Circulation Manager, the Metadata Wrangler, and the Content Server, there will need to be three separate, web-based admin UI's for running the program by non-developer/operations staff.


# Content Manager
## List Manager
### First Release
While we aspire to the delightfully powerful system specified below, here's what'll suffice for the beta NYPL release:

* Create/Destroy lists
* Name lists
* Lists powered by third party import (i.e. NYT Bestsellers, Bibliocommons Lists from NYPL librarians)
* Publish/Unpublish list
* Promote one list to homescreen of app for promotional purposes

### General Case / Later Releases
There are 2 general kinds of lists:

1. Dynamically imported & Read-only (selection & order managed by a third party feed)
2. Manually curated (selected and ordered by the library staff)

<table>
<tr>
<th>See:</th>
<th>Do:</th>
</tr>

<tr><td>List name</td><td>Set/edit name of list</td></tr>
<tr><td>Books</td><td>Add/Remove books</td></tr>
<tr><td>Annotation per book (optional)</td><td>Add, edit, remove annotation per book</td></tr>
<tr><td>Private comment per book (optional - only visible to admin staff)</td><td>Add, edit, remove comment per book</td></tr>
<tr><td>List Description (Optional - text)</td><td>Add, edit remove list description</td></tr>
<tr><td>Active/Inactive (Published) status of list</td><td>Toggle published status of list</td></tr>
<tr><td>Active/Inactive status of individual books on list</td><td>Toggle published status of individual books</td></tr>
<tr><td>List promoted to specific areas of the app (e.g. homescreen/inside lanes)</td><td>Specify which areas of the app a particular list is promoted/attached to</td></tr>
<tr><td>Active start/end time for the visibility of a list (optional)</td><td>Specify start(now/future) active time for a lists visibility</td></tr>
<tr><td>Language</td><td>manually set language/languages</td></tr>
<tr><td></td><td>Search to add books to a list</td></tr>
<tr><td></td><td>Admin can re-sort list by... (recency, date, alphabetical, manual, etc.)</td></tr>
</table>


# Metadata Wrangler

# Circulation Server