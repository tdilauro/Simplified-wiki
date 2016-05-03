To populate a local database with ~2000 Axis 360 books:

* `psql`
* `CREATE DATABASE simplified_circulation_dev`
* `GRANT ALL PRIVILEGES ON DATABASE simplified_circulation_dev TO [user]`
* `\q`
* edit `circulation.json` and set `integrations.Postgres.production_url` to `postgres://[user]:[password]@localhost:5432/simplified_circulation_dev`
* `bin/axis_monitor`
* `bin/metadata_wrangler_coverage`
* `bin/refresh_materialized_views`