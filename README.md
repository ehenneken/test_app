Micro service implementation of ADS object searches. This micro service supports search of the literature for bibliographic records relevant to one or more astronomical objects or for a specified position on the sky. An object name query searches used the SIMBAD and/or NED services to find relevant records in the ADS.

## SIMBAD implementation

The following queries are supported to retrieve information from SIMBAD:

#### Object Search
This search accepts either a list of object names or a list of SIMBAD object identifiers (POST queries). Both queries return JSON where the keys are object names and the values are dictionaries containing the (SIMBAD) canonical object name and the SIMBAD identifier. For example

    {
		"LMC" : {"canonical": "LMC",
		         "id": 12345
				 },
		"Andromeda": {"canonical": "ANDROMEDA",
		              "id": 342536
	             },
		"FooBar": None
	}

where a value of `None` means that the associated object name could not be identified in the SIMBAD database.

1. Retrieve SIMBAD identifiers for a string of object names

Using the Python "requests" module, this works as follows:

    headers = {'Content-type': 'application/json', 'Accept':'text/plain'}
	queryURL = 'http://localhost:4000/'
	payload = {"objects": ["M11", "M13", "Andromeda"]}
	r = requests.post(queryURL, data=json.dumps(payload), headers=headers)

and the results, returned through `r.json()`, are

    {"M11":{"canonical":"M  11", "id":2613692}, "M13":{"canonical": "M  13", "id":2894585}, "Andromeda":{"canonical":"ANDROMEDA", "id":1575544}}

2. Retrieve canonical object names for a list of SIMBAD identifiers

In this case this works as follows:

    headers = {'Content-type': 'application/json', 'Accept':'text/plain'}
	queryURL = 'http://localhost:4000/'
	payload = {"identifiers": [3133169, 1575544, 2419335, 3253618]}
	r = requests.post(queryURL, data=json.dumps(payload), headers=headers)

and the results in this case are

    {"3253618":{"canonical": "SMC", "id":3253618}, "1575544":{"canonical": "M  31", "id":1575544}, "2419335":{"canonical": "GAL CENTER", "id":2419335}, "3133169":{"canonical": "LMC", "id": 3133169}}

#### Positional ("Cone") Search
This search takes a position string and (optionally) a search radius. Position searches locate the papers dealing with celestial objects located within the specified radius of the specified position. The syntax for position searches is: 

    RA ±Dec : radius 

where RA and Dec are right ascention and declination J2000 positions, expressed in decimal degrees or in sexagesimal notation (hours minutes seconds and degrees arcmin and arcsec). The plus or minus sign before the declination is mandatory. The search radius may be given in decimal or sexagesimal degrees. For example, a 10' radius may be written as 0.1667 or 0 10 . The default search radius is 2' (0 2 = 0.033 deg). Some examples of allowed position strings:

    05 23 34.6 -69 45 22:0 6 (or 05h23m34.6s -69d45m22s:0m6s)
	05 23 34.6 -69 45 22:0.166666 (or 05h23m34.6s -69d45m22s:0.166666)
	80.89416667 -69.75611111:0.166666

The query is performed as follows

	pos_string = "80.89416667 -69.75611111:0.166666"
	queryURL = 'http://localhost:4000/pos/%s' % pos_string
	r = requests.get(queryURL)

This search returns the following results:

    {u'data': [u'1998A&A...335L..65H', u'2012yCat.1322....0Z', u'2008AcA....58..293S', u'2010AcA....60....1P', u'1974A&AS...18...47B', u'2011ApJS..197...16W', u'2009ApJS..184..172G', u'1972PASP...84..365H', u'1964IrAJ....6..241A', u'1999A&AS..139..277H', u'2003yCat.2246....0C', u'1988MNRAS.230..215B', u'2007PASP..119...19S', u'2010AcA....60..179P', u'1999AcA....49..521P', u'2011AcA....61..199P', u'2000A&A...355L..27H', u'2008AcA....58..163S', u'2012ApJ...747..107K', u'2008ApJS..178...56F', u'2011AJ....142...48W', u'2003A&A...405..111G', u'2008AJ....136...18W', u'2003yCat.1289....0Z', u'2000A&AS..145...11M', u'2003AcA....53...93S', u'2007AJ....134.1963F', u'2012ApJ...755...40P', u'2002AJ....124.2039K', u'2002LEDA.........0P', u'1976A&AS...24...35S', u'2011A&A...536A..60S', u'2000A&AS..143..391S', u'2009AcA....59....1S', u'1999AJ....117..238B', u'2002AcA....52..143S', u'1963IrAJ....6..127L', u'2006MNRAS.373..521R', u'2007ApJ...663..249D', u'2009AcA....59..239S', u'1988IRASP.C......0J', u'1988NASAR1190....1B', u'2001A&A...377..945C', u'2011AcA....61..103G', u'2008MNRAS.389..678B', u'2002AJ....124.3241S', u'2010A&A...514A...1I']}
