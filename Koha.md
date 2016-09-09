[Koha](http://www.koha.org/) is an open source ILS.

## Authentication and Patron information

It looks like the only way to integrate with Koha's authentication system is through [[SIP]]. There are a number of proposed APIs that will do what we want, but none of them are a complete part of the standard product. There are a number of active APIs, but only SIP does what we want.

* [The koha-restful project](http://git.biblibre.com/biblibre/koha-restful/) is not yet part of Koha.
* [Several other attempts are recorded here.](https://wiki.koha-community.org/wiki/New_REST_API_RFC)
* The [/svc/ HTTP API](https://wiki.koha-community.org/wiki/Koha_/svc/_HTTP_API) seems oriented towards authenticating administrators, not patrons.



