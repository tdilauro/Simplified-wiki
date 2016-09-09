Evergreen is an open source ILS.

## Authentication

Authentication can be handled two ways: either with a [[SIP]] integration ([Evergreen SIP documentation](docs.evergreen-ils.org/2.10/_sip_communication.html)) or through [OpenSRF](http://journal.code4lib.org/articles/3284). Both techniques assume that patrons are authenticated with username and password.

It looks like the [open-ils.auth](https://wiki.evergreen-ils.org/doku.php?id=open-ils_auth_api:open-ils.auth) service can give a yes-no answer given username and password.

## Patron record

A patron record can be fetched through SIP.

Presumably the open-ils.actor service can give more detailed information about a patron, but I can't find documentation on it.

