## Documentation about SPARQL queries performed in Wikidata

### SPARQL endpoints

* Official SPARQL Endpoint: https://query.wikidata.org
* [QLever Wikidata](https://qlever.dev/wikidata)
  * This is an alternative Wikidata Query service which is much faster (because it uses the [QLever triplestore technology](https://github.com/ad-freiburg/qlever)) but the data is not necessarily up to date.

### Documentation about Queries in Wikidata

* [SPARQL query service/queries](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/queries): documentation.
* Library Carpentry Wikidata: [Introduction to querying](https://librarycarpentry.github.io/lc-wikidata/05-intro_to_querying.html#top)
* [Structure of statements in Wikidata](https://www.wikidata.org/wiki/Help:Statements)
* Query builder: https://query.wikidata.org/querybuilder/?uselang=en
* [Wikidata Dates](https://www.wikidata.org/wiki/Help:Dates)

### SPARQL Tutorial and Standard (with examples)

* [Wikidata SPARQL Tutorial](https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial)
* [W3C SPARQL Standard (and tutorial)](https://www.w3.org/TR/sparql11-query/)

&nbsp;

## Inspection of available information

The objective of this stage of the workflow is to analyse the available information to answer our research questions about the population in our prosopography.

To begin with, we select a few people from the population chosen for the research and inspect their entries in Wikidata. This allows us to identify the properties that will enable us to find the population and determine what information is available.

* [Joyce White](http://www.wikidata.org/entity/Q164396](https://www.wikidata.org/wiki/Q103974661])(https://www.wikidata.org/wiki/Q18386003))
  * On this page or 'card' we can inspect the RDF triples available about this person in the Wikidata knowledge graph
  * Carefullyl inspect the properties ‘employer’ and ‘position held’
    * Cf. the ['card' of the same person in DBpedia](https://dbpedia.org/resource/Viktor_Ambartsumian)
    * Note the difference between a property-centered ontology and an assertion-centered ontology, which de facto contains implicit temporalities.

### URI vs URL

* URL -> https://www.wikidata.org/wiki/Q164396
* URI -> &lt;http://www.wikidata.org/entity/Q164396&gt;

**Dereferencing** is the technical mechanism that allows to insert a **URI** in the browser and be redirected to the **URL** of the page where the entity is described (if it exists). In the case of Wikidata and our population, it is the 'card' about the persons, with all the triples (in fact *statements*) presented in a readable form.

## Querying Wikidata to find the population

For archaeologists, the following properties appear to be an effective way of identifying the population:

* [Human](https://m.wikidata.org/wiki/Property:P31)
* [field of work](https://m.wikidata.org/wiki/Property:P101)

### Number of persons with 'occupation' and/or 'field of work' in archaeology

Figures as of March 09, 2026.: **26'020**

```
SELECT (COUNT(*) as ?eff) #?eff is a variable selected to store the COUNT(*), ie. the number of archaeoligists are found
WHERE {

    ## we use this clause in order to get only humans and not
    ## pages or any other entity wrongly associated to the occupation or field of work

    ?archaeologist wdt:P31 wd:Q5;  # Any instance of a human.
  
                   wdt:P106 wd:Q3621491  # archaeologist #indented and the previous line ends with ;, so that the program knows that it's still working with the variable ?archaeologists.
SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en".} #filters for enteties in Englisch
}  

```

### Combine 'occupation' with 'field of work'

We use here the **UNION** clause which allows to express an **OR** condition and merge two populations. The example of the combination of astronomers and physicians has been left, since a combination of two fields is not relevant for my project, but the referencere may be useful for future projects.

#### Astronomers

14327 as of February 16, 2026.

```
SELECT (COUNT(*) as ?eff)
WHERE {
    ?item wdt:P31 wd:Q5.
    {?item wdt:P106 wd:Q11063}
    UNION
    {?item wdt:P101 wd:Q333}  
}  
```

### Physicists

41629 as of February 16, 2026.

```
SELECT (COUNT(*) as ?eff)
WHERE {
    ?item wdt:P31 wd:Q5;  # Any instance of a human.
    {?item wdt:P106 wd:Q169470}
    UNION
    {?item wdt:P101 wd:Q413}  
}  
```

#### Both sup-populations

55956 as of February 16, 2026.

But be careful: it's actually the sum of the two, so a person could appear more then once.

```
SELECT (COUNT(*) as ?eff)
WHERE {
    ?item wdt:P31 wd:Q5;  # Any instance of a human.

    {?item wdt:P106 wd:Q11063}
    UNION
    {?item wdt:P101 wd:Q333} 
    UNION
    {?item wdt:P106 wd:Q169470}
    UNION
    {?item wdt:P101 wd:Q413}  
}  
```

### Actual number of people

48094 as of February 16, 2026.

There is an overlap of approximately 7,800 individuals who are both astronomers and physicists.

Please note that **SPARQL operates in a layered manner**: the innermost layer is executed first and the result set is then sent to the next layer up.

```
SELECT (COUNT(*) as ?eff)
WHERE {
    ### subquery adding the distinct clause
    {
        SELECT DISTINCT ?item
        WHERE {
        ?item wdt:P31 wd:Q5;  # Any instance of a human.
        {?item wdt:P106 wd:Q11063}
        UNION
        {?item wdt:P101 wd:Q333} 
        UNION
        {?item wdt:P106 wd:Q169470}
        UNION
        {?item wdt:P101 wd:Q413}  
        }
    }
}  
```

### Add a filter on the birth year

17709 on March 09 2026

```
SELECT (COUNT(*) as ?eff)
WHERE
    {
        {
        SELECT DISTINCT ?archaeologist
        WHERE {
        ?archaeologist wdt:P31 wd:Q5; 
                       wdt:P569 ?birthDate;
                       wdt:P106 wd:Q3621491.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981) 
            }
        } 
SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en".} 
    }  

```

**SELECT DISTINCT:** using this function prevents dupplicates being counted individually. In this case each archaeologist can only have a single entry for their birth date counted.

```
BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
```

This piece of code first turns the located birth date of an archaeologist, which is initially stored as an integer and turns it into a string. Now the program can locate the string of numbers that is 4 characters long: the year of birth. This value is then stored in the variable ?year. This process extracts the year of birth from the full birth date allowing for a better comparison, since the month and day are less relevant and are not available in all cases.

```
FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
```

This piece of code first turns the ?year variable into an integer, after which values between 1780 and 1981 are found.

### Inspect individuals

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?archaeologist ?archaeologistLabel ?year
WHERE {
  
           ?archaeologist wdt:P31 wd:Q5;
                          wdt:P106 wd:Q3621491;
                          wdt:P569 ?birthDate.
  BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
  
    ### Two ways of getting labels
    # SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }

    ## This is useful for query from external tool
    ?archaeologist rdfs:label ?archaeologistLabel.
    FILTER(LANG(?itemLabel) = 'en')
    }  
LIMIT 100
```

This code gives you a list of 100 archaeologists that have recorded birthdates and their recorded birthdates using only the English lables retrieved from outside Wikidata, specifically from the site indicated in the prefix to be saved under **rdfs**.

### Count population with English labels

17192 on March 09 2026

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT (COUNT(*) as ?eff)
WHERE
    {
    ### subquery adding the distinct clause
        {
SELECT DISTINCT ?archaeologist ?archaeologistLabel ?year

#Since this is not just filtering using ?archaeologists a single individual could be counted multiple times, so long as the combination of the three variable values is unique. This is just like if your primary key in a database is a unique combination between multiple entries on the table rather than a single entry.

WHERE {
  
           ?archaeologist wdt:P31 wd:Q5;
                          wdt:P106 wd:Q3621491;
                          wdt:P569 ?birthDate.
  BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
  
    ?archaeologist rdfs:label ?archaeologistLabel.
    FILTER(LANG(?archaeologistLabel) = 'en')
    }
            }
        } 

```

### Number of individuals without English labels

614 on March 09 2026

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT (COUNT(*) as ?eff)
WHERE
    {
        {
        SELECT DISTINCT ?archaeologist ?archaeologistLabel ?year
        WHERE {
        ?archaeologist wdt:P31 wd:Q5;
                       wdt:P106 wd:Q3621491;
                       wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981) 
        MINUS {?archaeologist rdfs:label ?archaeologistLabel.
            FILTER(LANG(?archaeologistLabel) = 'en') # MINUS: filters out whatever is specified after it.
            }
            }
        }  
    }  

```

### Individuals without English label

Inspect individuals' cards and observe their properties. The code determines what languages do the entries of archaeologists that have no English entry have.

```
PREFIX rdfs: [http://www.w3.org/2000/01/rdf-schema#](http://www.w3.org/2000/01/rdf-schema#)
PREFIX xsd: [http://www.w3.org/2001/XMLSchema#](http://www.w3.org/2001/XMLSchema#)
PREFIX wd: [http://www.wikidata.org/entity/](http://www.wikidata.org/entity/)
PREFIX wdt: [http://www.wikidata.org/prop/direct/](http://www.wikidata.org/prop/direct/)

SELECT ?archaeologist ?year
(group_concat(?iso_lang ; separator = ',') AS ?langs) #concatinates the language shortenings with commas in between and saves the string as ?langs.
(max(?archaeologistLabel) AS ?nameLabel)
WHERE {
        {SELECT DISTINCT ?archaeologist ?year
        #this second select needs to be separately packaged. All this does is find all the archaeologists without an English entry.
            WHERE {?archaeologist wdt:P31 wd:Q5 ;
                                  wdt:P106 wd:Q3621491 ;
                                  wdt:P569 ?birthDate .
                   BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
                   FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
                   MINUS {?archaeologist rdfs:label ?archaeologistLabel.
                   FILTER(LANG(?archaeologistLabel) = 'en')}
                   }
         }
        ?archaeologist rdfs:label ?archaeologistLabel .
        BIND(LANG(?archaeologistLabel) AS ?iso_lang) #What languages are the entries for archaeologists without English entries.
}

GROUP BY ?archaeologist ?year #Groups by archaeologist and the birth years.
ORDER BY ?archaeologist # Alphapetically/numerically sorted based on the id of the archaeologist.
LIMIT 100
```

## List the available properties and their numbers.

### Outgoing

Cf. [on this page](./Wikidata-liste-proprietes-population.md) the list of properties resulting from this query.

```
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?p ?propLabel ?eff ('' as ?notes)
WHERE {
{
    SELECT DISTINCT  ?p  (count(*) as ?eff)
    WHERE {
        ?archaeologist wdt:P31 wd:Q5;
                       wdt:P106 wd:Q3621491 ; 
                       wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
        ?archaeologist ?p ?o.
        }
		GROUP BY ?p
    }

    ## we need this (?p ?o) construct to get the label of the property
    ## properties are also entities in Wikidata,
    ## but only in the entities' namespace

    ?prop wikibase:directClaim ?p .
    ?prop rdfs:label ?propLabel.

    # ?p which is represented be the wtd:ZYX doesn’t have a human readable label attached to it.
    # so this bit links wtd:XYZ with wd:ZYX which does have a readable label that it then shows you at the end.

    FILTER(LANG(?propLabel) = 'en')
    }  
ORDER BY DESC(?eff) 
```

**NB**. Please note that timeout issues may occur when executing the query on the Wikidata SPARQL endpoint. This is because the query takes too long to process and an error message appears.
&nbsp;
In this case, you must restrict the period of time considered or limit the number of UNION clauses and break the query down into different parts.

Or you can switch to the QLever SPARQL Endpoint (cf. above) that normally works fine.

This list is then exported and transformed to a table in order to document the sequence of operations. This is done as follows:

* execute the query then download the result in csv format into your projet's repository. Cf. the file in this directory: data_queries/Wikidata/wdt_population_outgoing_properties_20260302.csv
* open the CSV in VSCode as text and convert it to a Markdown Table using the plugin 'CSV to Markdown Table (phoihos)'
* copy the whole table and paste it a new Markdown document, cf. [Wikidata-list-of-properties-archaeologists.md](Wikidata-list-of-properties-archaeologists.mdWikidata-liste-proprietes-population.md)
* close the CSV file

In the column 'notes' of the property table you can add links to the pages where you document the treatement of the corresponding information.

### Incoming

```
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?p ?propLabel ?eff  ('' as ?notes)
WHERE {
{
    SELECT DISTINCT  ?p  (count(*) as ?eff)
    WHERE {
        ?archaeologist wdt:P31 wd:Q5;
                       wdt:P106 wd:Q3621491 ; 
                       wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            ## inversed triple
			?s ?p ?archaeologist.
        }
		GROUP BY ?p
    }
    ?prop wikibase:directClaim ?p .

    ?prop rdfs:label ?propLabel.
        FILTER(LANG(?propLabel) = 'en')
    }  
ORDER BY DESC(?eff)
Limit 500
```

Relevant incoming properties:

| p                                          | propLabel                                            | eff   | notes |
| ------------------------------------------ | ---------------------------------------------------- | ----- | ----- |
| http://www.wikidata.org/prop/direct/P50    | author                                               | 73652 |       |
| http://www.wikidata.org/prop/direct/P61    | discoverer or inventor                               | 3362  |       |
| http://www.wikidata.org/prop/direct/P921   | main subject                                         | 2889  |       |
| http://www.wikidata.org/prop/direct/P184   | doctoral advisor                                     | 1679  |       |
| http://www.wikidata.org/prop/direct/P170   | creator                                              | 1480  |       |
| http://www.wikidata.org/prop/direct/P98    | editor                                               | 1456  |       |
| http://www.wikidata.org/prop/direct/P1066  | student of                                           | 861   |       |
| http://www.wikidata.org/prop/direct/P185   | doctoral student                                     | 837   |       |
| http://www.wikidata.org/prop/direct/P4345  | director of archaeological fieldwork                 | 604   |       |
| http://www.wikidata.org/prop/direct/P802   | student                                              | 498   |       |
| http://www.wikidata.org/prop/direct/P710   | participant                                          | 441   |       |
| http://www.wikidata.org/prop/direct/P1037  | director / manager                                   | 287   |       |
| http://www.wikidata.org/prop/direct/P737   | influenced by                                        | 115   |       |
| http://www.wikidata.org/prop/direct/P57    | director                                             | 70    |       |
| http://www.wikidata.org/prop/direct/P1346  | winner                                               | 69    |       |
     

This is just a portion of the resulting downloaded CSV. If you have more you should also create a dedicated page for the incoming properties.

## Define subpopulations (optional)

More complex query that adds a code to both parts of the populations, astronomers and physicists.

Only use if relevant for your project.

### Add the subpopulation code

```
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?p ?propLabel ?eff ?itemType
WHERE {
{
    SELECT DISTINCT  ?p  (count(*) as ?eff) ?itemType
    WHERE {
        ?item wdt:P31 wd:Q5; 
             wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?item wdt:P106 wd:Q11063.
			BIND ('astronomer' as ?itemType)}
            UNION
            {?item wdt:P101 wd:Q333.
			BIND ('astronomer' as ?itemType).} 
            UNION
            {?item wdt:P106 wd:Q169470.
			BIND ('physicist' as ?itemType)}
            UNION
            {?item wdt:P101 wd:Q413.
			BIND ('physicist' as ?itemType)}
			.
			?item ?p ?o.
        }
		GROUP BY ?p ?itemType
        ## limit to more frequent properties
		HAVING(?eff >= 100)
    }
    ?prop wikibase:directClaim ?p .

    ?prop rdfs:label ?propLabel.
        FILTER(LANG(?propLabel) = 'en')
    }  
ORDER BY ?propLabel ?itemType 
```

## Same query but grouped

```
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?p ?propLabel (max(?eff) as ?max_eff) 
(group_concat(concat(str(?eff), ' ', ?itemType); separator=" | ") as ?eff_type)
WHERE {


SELECT ?p ?propLabel ?eff ?itemType
WHERE {
{
    SELECT DISTINCT  ?p  (count(*) as ?eff) ?itemType
    WHERE {
        ?item wdt:P31 wd:Q5; 
             wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?item wdt:P106 wd:Q11063.
			BIND ('astronomer' as ?itemType)}
            UNION
            {?item wdt:P101 wd:Q333.
			BIND ('astronomer' as ?itemType).} 
            UNION
            {?item wdt:P106 wd:Q169470.
			BIND ('physicist' as ?itemType)}
            UNION
            {?item wdt:P101 wd:Q413.
			BIND ('physicist' as ?itemType)}
			.
			?item ?p ?o.
        }
		GROUP BY ?p ?itemType
		HAVING(?eff >= 100)
    }
    ?prop wikibase:directClaim ?p .

    ?prop rdfs:label ?propLabel.
        FILTER(LANG(?propLabel) = 'en')
    }  
	}
	GROUP BY ?p ?propLabel
     ORDER BY desc(?max_eff)

```
