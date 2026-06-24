## Get the employers

* The property _P108 employer_ is quite frequent and adds an interesting relation to organisations that we will also use for graph analysis
* See [this document](../explore-employer.md) for a distribution of the most frequent employers



## Get the relations to the employers

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?person_uri ?education_uri 
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?person_uri
        WHERE {
         {?person_uri wdt:P106 wd:Q3621491} # archaeologist
            UNION
            {?person_uri wdt:P101 wd:Q23498} # archaeology field 
            ?person_uri wdt:P31 wd:Q5;
                            wdt:P569 ?birthDate; # It must necessarily have a birth date property
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)  
            }
        } 
		
      ?person_uri wdt:P69 ?education_uri.
}  
ORDER BY ?person_uri
# LIMIT 30
```
* execute the query on Wikidata or [QLever](https://qlever.dev/wikidata)
* download the result as a CSV file
* import the file as a 'import_person_employer' table into the database using DBeaver




## Get the list of the organisations

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?person_uri ?education_uri 
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?person_uri
        WHERE {
         {?person_uri wdt:P106 wd:Q3621491} # archaeologist
            UNION
            {?person_uri wdt:P101 wd:Q23498} # archaeology field 
            ?person_uri wdt:P31 wd:Q5;
                            wdt:P569 ?birthDate; # It must necessarily have a birth date property
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)  
            }
        } 
		
      ?person_uri wdt:P69 ?education_uri.
        ?organisation_uri rdfs:label ?organisation_label.
        FILTER(LANG(?organisation_label) = 'en')
}  
# LIMIT 30
```
* execute the query on Wikidata or [QLever](https://qlever.dev/wikidata)
* download the result as a CSV file
* import the file as an 'import_organisation' table into the database using DBeaver
  




## The 30 most frequent places of education


| education_uri                            | education_label                                | count |
| ---------------------------------------- | ---------------------------------------------- | ----- |
| http://www.wikidata.org/entity/Q1576779  | Italian School of Archaeology at Athens        | 309   |
| http://www.wikidata.org/entity/Q390287   | Eötvös Loránd University                       | 183   |
| http://www.wikidata.org/entity/Q83259    | École Normale Supérieure                       | 150   |
| http://www.wikidata.org/entity/Q13371    | Harvard University                             | 148   |
| http://www.wikidata.org/entity/Q165980   | University of Vienna                           | 139   |
| http://www.wikidata.org/entity/Q35794    | University of Cambridge                        | 113   |
| http://www.wikidata.org/entity/Q144488   | University of Warsaw                           | 96    |
| http://www.wikidata.org/entity/Q152171   | University of Bonn                             | 94    |
| http://www.wikidata.org/entity/Q193196   | University College London                      | 90    |
| http://www.wikidata.org/entity/Q273631   | École pratique des hautes études               | 86    |
| http://www.wikidata.org/entity/Q152087   | Humboldt-Universität zu Berlin                 | 86    |
| http://www.wikidata.org/entity/Q131252   | University of Chicago                          | 84    |
| http://www.wikidata.org/entity/Q174158   | Hebrew University of Jerusalem                 | 82    |
| http://www.wikidata.org/entity/Q3563550  | Faculty of Arts, Charles University in Prague  | 79    |
| http://www.wikidata.org/entity/Q209344   | Sapienza University of Rome                    | 79    |
| http://www.wikidata.org/entity/Q12016640 | Masaryk University Faculty of Arts             | 77    |
| http://www.wikidata.org/entity/Q4204528  | MSU Faculty of History                         | 77    |
| http://www.wikidata.org/entity/Q27621    | Saint Petersburg State University              | 76    |
| http://www.wikidata.org/entity/Q999763   | University of Paris 1 Pantheon-Sorbonne        | 75    |
| http://www.wikidata.org/entity/Q189441   | Jagiellonian University                        | 74    |
| http://www.wikidata.org/entity/Q55044    | Ludwig-Maximilians-Universität München         | 73    |
| http://www.wikidata.org/entity/Q34433    | University of Oxford                           | 72    |
| http://www.wikidata.org/entity/Q49117    | University of Pennsylvania                     | 69    |
| http://www.wikidata.org/entity/Q49088    | Columbia University                            | 67    |
| http://www.wikidata.org/entity/Q209842   | University of Paris                            | 65    |
| http://www.wikidata.org/entity/Q49112    | Yale University                                | 65    |
| http://www.wikidata.org/entity/Q230492   | University of Michigan                         | 65    |
| http://www.wikidata.org/entity/Q547867   | National and Kapodistrian University of Athens | 62    |
| http://www.wikidata.org/entity/Q154804   | Leipzig University                             | 61    |
| http://www.wikidata.org/entity/Q153978   | University of Tübingen                         | 58    |



## Get the list of the organisations - not relevant for me

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT DISTINCT ?organisation_uri ?organisation_label 
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
		
        ?person_uri wdt:P108 ?organisation_uri.
        ?organisation_uri rdfs:label ?organisation_label.
        FILTER(LANG(?organisation_label) = 'en')
}  
# LIMIT 30
```
* execute the query on Wikidata or [QLever](https://qlever.dev/wikidata)
* download the result as a CSV file
* import the file as an 'import_organisation' table into the database using DBeaver
  

## Get the list of the organisations' classes - not relevant for me

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT DISTINCT ?organisation_uri  ?organisation_class_uri ?organisation_class_label
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
		
        ?person_uri wdt:P108 ?organisation_uri.
        ?organisation_uri wdt:P31 ?organisation_class_uri.
        ?organisation_class_uri rdfs:label ?organisation_class_label.
        FILTER(LANG(?organisation_class_label) = 'en')

}  
LIMIT 30
```
* execute the query on Wikidata or [QLever](https://qlever.dev/wikidata)
* download the result as a CSV file: import_organisations_and_classes.csv
* import the file as an 'import_organisations_classes' table into the database using DBeaver
  

  

### The first 20 most frequent employer classes - not relevant for me

| class                                     | classLabel                                          | eff  |
| ----------------------------------------- | --------------------------------------------------- | ---- |
| http://www.wikidata.org/entity/Q875538    | public university                                   | 7636 |
| http://www.wikidata.org/entity/Q3918      | university                                          | 6258 |
| http://www.wikidata.org/entity/Q45400320  | open-access publisher                               | 5570 |
| http://www.wikidata.org/entity/Q62078547  | public research university                          | 4529 |
| http://www.wikidata.org/entity/Q31855     | research institute                                  | 4176 |
| http://www.wikidata.org/entity/Q23002054  | private not-for-profit educational institution      | 3358 |
| http://www.wikidata.org/entity/Q23002039  | public educational institution of the United States | 2622 |
| http://www.wikidata.org/entity/Q15936437  | research university                                 | 2456 |
| http://www.wikidata.org/entity/Q43229     | organization                                        | 2314 |
| http://www.wikidata.org/entity/Q902104    | private university                                  | 2302 |
| http://www.wikidata.org/entity/Q1767829   | comprehensive university                            | 2107 |
| http://www.wikidata.org/entity/Q1371037   | institute of technology                             | 1629 |
| http://www.wikidata.org/entity/Q615150    | land-grant university                               | 1505 |
| http://www.wikidata.org/entity/Q96888669  | academic publisher                                  | 1434 |
| http://www.wikidata.org/entity/Q115427560 | University of Excellence                            | 1307 |
| http://www.wikidata.org/entity/Q5341295   | educational organization                            | 1271 |
| http://www.wikidata.org/entity/Q1254933   | astronomical observatory                            | 1196 |
| http://www.wikidata.org/entity/Q1188663   | Colonial Colleges                                   | 1051 |
| http://www.wikidata.org/entity/Q163740    | nonprofit organization                              | 949  |
| http://www.wikidata.org/entity/Q38723     | higher education institution                        | 936  |




<br/>


### Examples of private companies - not relevant for me

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?employer ?employerLabel ?classLabel (COUNT(*) as ?eff)
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?item
        WHERE {
        ?item wdt:P31 wd:Q5; 
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?item wdt:P106 wd:Q11063}
            UNION
            {?item wdt:P101 wd:Q333} 
            UNION
            {?item wdt:P106 wd:Q169470}
            UNION
            {?item wdt:P101 wd:Q413}  
            }
        } 
		
      ?item wdt:P108 ?employer.
        ?employer rdfs:label ?employerLabel.
        FILTER(LANG(?employerLabel) = 'en')
		?employer wdt:P31 ?class.
        ?class rdfs:label ?classLabel.
        FILTER(LANG(?classLabel) = 'en')
		FILTER regex(?classLabel, '.*business.*|.*enterprise.*|.*company.*') 
}  
GROUP BY ?employer ?employerLabel ?class ?classLabel 
ORDER BY DESC(?eff) ?employer 
LIMIT 20
```


| employer                                | employerLabel                                                | classLabel             | eff |
| --------------------------------------- | ------------------------------------------------------------ | ---------------------- | --- |
| http://www.wikidata.org/entity/Q217365  | Bell Labs                                                    | privately held company | 128 |
| http://www.wikidata.org/entity/Q49112   | Yale University                                              | production company     | 122 |
| http://www.wikidata.org/entity/Q238101  | University of Minnesota                                      | production company     | 71  |
| http://www.wikidata.org/entity/Q127990  | Australian National University                               | production company     | 53  |
| http://www.wikidata.org/entity/Q37156   | IBM                                                          | software company       | 48  |
| http://www.wikidata.org/entity/Q37156   | IBM                                                          | enterprise             | 48  |
| http://www.wikidata.org/entity/Q37156   | IBM                                                          | business               | 48  |
| http://www.wikidata.org/entity/Q37156   | IBM                                                          | public company         | 48  |
| http://www.wikidata.org/entity/Q37156   | IBM                                                          | technology company     | 48  |
| http://www.wikidata.org/entity/Q126824  | Institute of Physics and Power Engineering JSC               | business               | 47  |
| http://www.wikidata.org/entity/Q658192  | Vilnius University                                           | business               | 28  |
| http://www.wikidata.org/entity/Q1117048 | Commonwealth Scientific and Industrial Research Organisation | production company     | 27  |
| http://www.wikidata.org/entity/Q54173   | General Electric                                             | public company         | 23  |
| http://www.wikidata.org/entity/Q54173   | General Electric                                             | enterprise             | 23  |
| http://www.wikidata.org/entity/Q54173   | General Electric                                             | business               | 23  |
| http://www.wikidata.org/entity/Q734764  | University of New South Wales                                | production company     | 17  |
| http://www.wikidata.org/entity/Q170416  | Koninklijke Philips NV                                       | public company         | 15  |
| http://www.wikidata.org/entity/Q170416  | Koninklijke Philips NV                                       | enterprise             | 15  |
| http://www.wikidata.org/entity/Q170416  | Koninklijke Philips NV                                       | business               | 15  |
| http://www.wikidata.org/entity/Q632404  | Westinghouse Electric Corporation                            | business               | 13  |