Here's the work we have to do to automatically integrate the circulation manager with the library registry:

# Before launch

This work needs to be done before we can launch library registry.

## Circulation manager

- [Logo uploader](https://github.com/NYPL-Simplified/circulation/issues/566)
- [Store library registry URL](https://github.com/NYPL-Simplified/circulation/issues/576)
- [Manual "Update registration" button](https://github.com/NYPL-Simplified/circulation/issues/577) that kicks off registration procedure

## Library registry

- Register a library or update registration on demand

# After launch

For a while, we can compensate for the absence of these by manually
inserting library info into the registry and manually setting up
shared secrets.

## Circulation manager

- [Service area editor](https://github.com/NYPL-Simplified/circulation/issues/578)
- [Autogenerate public key](https://github.com/NYPL-Simplified/circulation/issues/579)
- [Manual "set shared secret" button](https://github.com/NYPL-Simplified/circulation/issues/580) that negotiates shared secret with library registry.

## Library registry

- Regenerate shared secret on demand.

# After popularity hits

We don't really need this stuff until we start getting a large number
of libraries in the system that don't fit the "geographically localized public library" mold.

## Circulation manager

- [Select audiences for library](https://github.com/NYPL-Simplified/circulation/issues/581)
- [Insert estimated collection sizes into authentication document](https://github.com/NYPL-Simplified/circulation/issues/583)

## Library registry

- Improve presentation of libraries through lenses other than geography