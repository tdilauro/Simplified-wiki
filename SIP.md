SIP is a TCP-based protocol for ILSes performing basic functions of a library.

[The documentation for the Evergreen ILS](http://docs.evergreen-ils.org/2.10/_sip_communication.html) gives a rundown of the SIP message types. The only message we're interested in at the moment is [Patron Information](docs.evergreen-ils.org/2.10/_sip_communication.html#sip_63-64_patron_informationl#sip_63-64_patron_information), which verifies a patron's credentials and returns information about them.

Although archaic and seemingly restricted to barcode/PIN based authentication, SIP is ILS-independent and would allow Simplified to present a simple Basic Auth-based authentication system.