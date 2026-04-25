## Inspect the most frequent fields of work

* In the case of this population, we observe that although the main fields of work are physics and astronomy, there are many other fiels and it would be interesting to inspect specificities related to them.

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?career ?careerLabel (COUNT(*) as ?eff)
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?archaeologist
        WHERE {
        {?archaeologist wdt:P106 wd:Q3621491} # archaeologist
            UNION
            {?archaeologist wdt:P101 wd:Q23498} # archaeology field 
            ?archaeologist wdt:P31 wd:Q5;
                            wdt:P569 ?birthDate; # It must necessarily have a birth date property
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
            }
			}
        ### The property P101 associates fields of work to persons
        ?archaeologist wdt:P101 ?career.
        ?career rdfs:label ?careerLabel.
        FILTER(LANG(?careerLabel) = 'en')
} 
GROUP BY ?career ?careerLabel 
ORDER BY DESC(?eff)
LIMIT 100
```




### Most frequent fields of work

| career                                    | careerLabel                        | number |
| ----------------------------------------- | ---------------------------------- | ------ |
| http://www.wikidata.org/entity/Q23498     | archaeology                        | 4036   |
| http://www.wikidata.org/entity/Q309       | history                            | 1000   |
| http://www.wikidata.org/entity/Q2415966   | prehistoric archaeology            | 447    |
| http://www.wikidata.org/entity/Q50637     | art history                        | 428    |
| http://www.wikidata.org/entity/Q1669589   | classical archaeology              | 380    |
| http://www.wikidata.org/entity/Q23404     | anthropology                       | 285    |
| http://www.wikidata.org/entity/Q1671076   | medieval archaeology               | 249    |
| http://www.wikidata.org/entity/Q959782    | archaeological excavation          | 198    |
| http://www.wikidata.org/entity/Q460162    | museology                          | 170    |
| http://www.wikidata.org/entity/Q11756     | prehistory                         | 149    |
| http://www.wikidata.org/entity/Q145903    | Egyptology                         | 147    |
| http://www.wikidata.org/entity/Q132151    | ethnography                        | 142    |
| http://www.wikidata.org/entity/Q631286    | numismatics                        | 133    |
| http://www.wikidata.org/entity/Q41493     | ancient history                    | 132    |
| http://www.wikidata.org/entity/Q113884225 | ancient archeology                 | 131    |
| http://www.wikidata.org/entity/Q495527    | classical philology                | 97     |
| http://www.wikidata.org/entity/Q181260    | epigraphy                          | 92     |
| http://www.wikidata.org/entity/Q12271     | architecture                       | 88     |
| http://www.wikidata.org/entity/Q476294    | oriental studies                   | 87     |
| http://www.wikidata.org/entity/Q1760972   | Christian archaeology              | 85     |
| http://www.wikidata.org/entity/Q12554     | Middle Ages                        | 81     |
| http://www.wikidata.org/entity/Q63386008  | curating                           | 80     |
| http://www.wikidata.org/entity/Q119190    | medieval studies                   | 74     |
| http://www.wikidata.org/entity/Q11761     | Bronze Age                         | 70     |
| http://www.wikidata.org/entity/Q43455     | ethnology                          | 69     |
| http://www.wikidata.org/entity/Q914856    | historic preservation              | 69     |
| http://www.wikidata.org/entity/Q36422     | Neolithic                          | 68     |
| http://www.wikidata.org/entity/Q3621486   | landscape archaeology              | 68     |
| http://www.wikidata.org/entity/Q1069      | geology                            | 65     |
| http://www.wikidata.org/entity/Q1254302   | archaeology of the Roman provinces | 62     |

&nbsp;

### Query to get the data and import them into the database

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT DISTINCT ?archaeologist ?field_uri ?field_label
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?archaeologist
        WHERE {
        
            {?archaeologist wdt:P106 wd:Q3621491} # archaeologist
            UNION
            {?archaeologist wdt:P101 wd:Q23498} # archaeology field 
            ?archaeologist wdt:P31 wd:Q5;
                            wdt:P569 ?birthDate; # It must necessarily have a birth date property
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)
            }
        } 
        ### The property P101 associates fields of work to persons
        ?person_uri wdt:P101 ?field_uri.
        ?field_uri rdfs:label ?field_label.
        FILTER(LANG(?field_label) = 'en')
}  
LIMIT 100

```



#### Experimental : DO NOT USE
```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT DISTINCT ?person_uri ?field ?fieldLabel ?parentField ?parentFieldLabel ?parentClass ?parentClassLabel
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?person_uri
        WHERE {
        ?person_uri wdt:P31 wd:Q5; 
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?person_uri wdt:P106 wd:Q11063}
            UNION
            {?person_uri wdt:P101 wd:Q333} 
            UNION
            {?person_uri wdt:P106 wd:Q169470}
            UNION
            {?person_uri wdt:P101 wd:Q413} 
            }
        } 
        ### The property P101 associates fields of work to persons
        ?person_uri wdt:P101 ?field.
        ?field rdfs:label ?fieldLabel.
        FILTER(LANG(?fieldLabel) = 'en')
        OPTIONAL {
                # is instance of
                ?field wdt:P31 ?parentClass.
                        ?parentClass rdfs:label ?parentClassLabel.
                FILTER(LANG(?parentClassLabel) = 'en')
            }
        OPTIONAL {
                # is subclass of
                ?field wdt:P279 ?parentField.
                        ?parentField rdfs:label ?parentFieldLabel.
                FILTER(LANG(?parentFieldLabel) = 'en')
            }  
}  
# LIMIT 100

```




### Create a new table

* Download the result of this query as a CSV file
* Import it as a new table into the SQLite database
  * rename the column names during the import process 
* Inspect the imported data using the SQL scripts in the file [] 




## Occupations

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?person_uri ?occupation_label ?occupation_uri
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?person_uri
        WHERE {
        ?person_uri wdt:P31 wd:Q5; 
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?person_uri wdt:P106 wd:Q11063}
            UNION
            {?person_uri wdt:P101 wd:Q333} 
            UNION
            {?person_uri wdt:P106 wd:Q169470}
            UNION
            {?person_uri wdt:P101 wd:Q413}  
            }
        } 
	
        ### The property P106 associates occupations to persons
        ?person_uri wdt:P106 ?occupation_uri.
        ?occupation_uri rdfs:label ?occupation_label.
        FILTER(LANG(?occupation_label) = 'en')
}  
ORDER BY ?person_uri ?occupation_uri
LIMIT 10
```