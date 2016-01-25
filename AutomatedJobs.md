# Circulation manager

## Jobs that need to be running all the time or bad things will happen

* bin/threem_circulation_sweep (If you have a 3M Cloud Library collection)
* bin/threem_circulation_monitor (If you have a 3M Cloud Library collection)
* bin/overdrive_monitor_full (If you have an Overdrive collection)
* bin/overdrive_monitor_recent (If you have an Overdrive collection)
* bin/overdrive_reaper (If you have an Overdrive collection)
* bin/axis_monitor (If you have an Overdrive collection)

These jobs are how the circulation manager learns about new books in a collection. These jobs also help the circulation manager find out about changes to circulation in something close to real time. 

These jobs should always be running. If one of them completes or stops running it needs to be restarted immediately.

If the Axis job or one of the 3M jobs breaks or stops running, you can start it up again and catch up. If one of the Overdrive jobs breaks or stops running, you lose circulation information that we can never get back. 

# Jobs that need to run regularly or bad things will happen

* bin/refresh_materialized_views (Once a night, at 3 AM. Requires postgres 9.4)
* bin/update_nyt_best_seller_lists (Once a week)
* bin/cache_opds_blocks (Every 5 minutes)
* bin/cache_opds_lane_facets (Currently disabled)
* bin/make_presentation_ready (Every 10 minutes)
* bin/make_identifiers_without_work_presentation_ready (Every 10 minutes)
* bin/content_server_monitor (Once a day)

## Jobs that should be run as needed

These are utility jobs for whacking some part of the system to get it unstuck, or doing large-scale data revisions to accommodate a change in code.

Note that some of these jobs can take a _very_ long time to complete, the longest being `bin/metadata_refresh`.

* bin/metadata_refresh
* bin/subjects_prepare (Assign Subjects to Genres)
* bin/refresh_permanent_work_id
* bin/opds_entries_cache (Update cached OPDS entries)
* bin/search_index_update (Force refresh of search index)
* bin/make_identifiers_without_edition_presentation_ready

# Metadata wrangler

## Jobs that need to run regularly or bad things will happen

* bin/identifiers_resolve (Every 10 minutes)
* bin/make_presentation_ready (Every 10 minutes)
* bin/content_server_monitor (Once a day)
* bin/refresh_materialized_views (Once a night, at 3 AM. Requires postgres 9.4)

## Jobs that should be run as needed

* bin/opds_entries_cache
* bin/subjects_prepare
* bin/works_reflassify

# Content server

## Jobs that need to run regularly or bad things will happen

* bin/update_main_mirror (Once a week)
* bin/update_generated_mirror (Once a week)
* bin/gutenberg_monitor (Once a week)
* bin/standard_ebooks_monitor (Once a week)
* bin/unglue_it_monitor (Once a week)
* bin/make_presentation_ready (Once a day) 
* bin/refresh_materialized_views (Once a night, at 3 AM. Requires postgres 9.4)

## Jobs that should be run as needed

* bin/opds_entries_cache (Refresh cached OPDS entries)