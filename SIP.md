SIP is a TCP-based protocol for ILSes performing basic functions of a library.

[The official documentation is here.](http://multimedia.3m.com/mws/media/355361O/sip2-protocol.pdf) [The documentation for the Evergreen ILS](http://docs.evergreen-ils.org/2.10/_sip_communication.html) gives a simplified rundown of the SIP message types. The only message we're interested in at the moment is [Patron Information](docs.evergreen-ils.org/2.10/_sip_communication.html#sip_63-64_patron_informationl#sip_63-64_patron_information), which verifies a patron's credentials and returns information about them.

Although archaic and restricted to barcode/PIN based authentication, SIP is ILS-independent and allows Simplified to present a simple authentication system through HTTP Basic Auth.

## Implementations

Evergreen and Koha both use [SIPServer](https://github.com/atz/SIPServer) as their SIP server implementation.