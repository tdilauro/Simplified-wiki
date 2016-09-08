The Millenium Patron API is a simple HTTP-based API that returns HTML documents. It offers two useful endpoints:

* `pintest`: Verify that a barcode/PIN combination is valid. Example URL: `https://my-ils.com/PATRONAPI/{barcode}/{pin}/pintest`

* `dump`: Return an HTML representation of a patron's ILS record. Example URL: `https://my-ils.com/PATRONAPI/{barcode}/dump`