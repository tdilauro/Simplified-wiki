# Canonical Contributors
## Problem

- Data from multiple sources is smushed into individual Contributor and Contribution objects. Source of original Contributor data is lost.
- No way to maintain conflicting Contributor data and reference a single person.
- No way to talk about Contributors in the context of a Work.

Right now when we get information about contributors from third-party sources, we’re immediately either processing the data into an existing Contributor or making a brand-new one. This decision is based largely on accident-prone name matching, and it can lead to lost information, overlapping Contributor records for authors with the same name, or duplicate records for authors with slightly different name spellings.

Similar to `Work.presentation_edition`, the following proposal seeks to decrease lost data and support data remediation as algorithms for Contributor representation change. It also supports the representation of a Contributor as a single, canonical person in our data model and lends support to the possibility of considering individual Works canonically in the future.

## Proposal
### Phase 1 - Maintain original data from source
#### Schema Changes
&#42; = new column / table<br/>
~--~ = removed column

**1. contributions**
- id
- &#42;data_source_id (FK datasources.id)
- &#42;identifier_id (FK identifiers.id)
- contributor_id (FK contributors.id)
- role

When we find out from some data source that some person had some role in a book, we create a contribution to represent that assertion. We associate the contribution with an appropriate contributor object, possibly creating that one too.

This is similar to what we have now, but I've added a data_source_id field to track where the data came from, and changed edition_id to identifier_id to allow contributor-only data sources like VIAF to generate contributors and to better support relating contributor objects based on identifier equivalencies.

**2. contributors**
- id
- &#42;data_source_id (FK datasources.id)
- &#42;canonical_id (FK contributors.id)
- sort_name
- display_name
- viaf_id
- etc.

This is similar to before, with the addition of data_source_id and canonical_id.

Whenever we hear about a person from some data source, we create a contributor to represent that person. Some data sources (VIAF) may give us contributors without contributions, which is fine.

Once we have a big pile of contributors and their contributions, we can apply algorithms to figure out which contributors are the same person. For example, if we have two contributors with the same name who wrote a book with the same title, they're probably the same person.

For each person we're able to identify, we create a "canonical contributor" with the best available information. Each 
contributor record links to the corresponding canonical contributor.

**3. editions**
- ~editions.sort_author~ (=> works.sort_author)

**4. works**
- &#42;works.sort_author (<= editions.sort_author)

**5. mv_works_for_lanes**
- works.id as works_id
- editions.id as editions_id
- ~editions.sort_author~
- &#42;works.sort_author
- etc.

#### New Functionality
- A Contributor can return its canonical version (even if it is its own canonical version). If the Contributor is the canonical version, it sends itself.
- The Contributor class can attempt to find a likely canonical Contributor for a given contributor, based on equivalent identifiers and possibly titles and name text matching.

#### Altered Functionality
- [migration] Existing Contributions are given the data_source_id and identifier_id of their related Edition.
- [migration] All existing contributors will be given the “Internal Processing” DataSource, since they’re currently an amalgam of multiple different DataSource objects.
- [migration] Move editions.sort_author to works.sort_author. Drop the editions.sort_author column, as it’s no longer being used.
- server_core.model.py:Edition.author_contributors => Work.author_contributors only returns canonical contributors. (Moving this functionality to Work because contributions now connect via identifiers, not editions.)
- server_core:model.py:Contributor.lookup will require a DataSource for overwrites and/or updates.
metadata_wrangler:viaf.py:VIAFClient.process_contributor creates new Contributor objects using the metadata layer, instead of making changes to an existing Contributor. These contributors will be tied to the identifier sent to the metadata wrangler for resolution.
- server_core:metadata_layer.py:ContributorData requires a DataSource in order to be applied to an identifier.

### Phase 2 - Develop higher-level Text-Author relationship
#### Schema Changes

&#42; = new column / table

**1. &#42;workcontributions**
- &#42;id
- &#42;work_id (FK works.id)
- &#42;contributor_id (FK contributors.id)
- &#42;role

Thus far we've been able to figure out that the 30 people named "Herman Melville" who wrote our 30 editions of "Moby-Dick" are all the same person. But some of those editions of "Moby-Dick" have additional authors, illustrators, audiobook narrators, etc. We need some way of expressing the general fact "Herman Melville wrote Moby-Dick", as distinct from the more specific fact "Frank Muller narrated the 2008 Recorded Books edition of Moby-Dick".

The works table lets us talk about "Moby-Dick" as a work, and the workcontributions table lets us talk about who made the work. This table's schema maps a work to a contributor, and it is generated and updated during Work.calculate_presentation.

We'd have a rule that you can only use a canonical contributor as contributor_id.

#### New Functionality
- The Work is able to calculate and recalculate its contributions. This recalculation should suit contribution profiles on both the metadata wrangler and the circulation manager.

#### Altered Functionality
- circulation:api/lanes.py:ContributorLane.apply_filters needs to be updated to check WorkContributions.

## Advantages

With this system, it's easy to dump data into the database and figure out what it means later. Since we keep all the original data in pristine form, we can recalculate canonical contributors as we improve our algorithms, without losing any data.

It's also possible to show very general information about a work ("Moby-Dick, by Herman Melville") before drilling down to the more specific information associated with specific editions of the work or general data sources. This doesn’t impact the circulation manager at this time (except perhaps for certain open access books), but it could be useful on the metadata wrangler and provides a general flexibility that may prove beneficial in the future.