# Just some SPARQL to try at the [Wikidata SPARQL endpoint](https://query.wikidata.org/)

## Composers until Monteverdi, a timeline

```
#defaultView:Timeline
SELECT ?composer ?composerLabel ?dateOD ?placeODLabel (SAMPLE(?img) AS ?image) (COUNT(?work) as ?CW)
WHERE {
  ?work wdt:P86 ?composer .                 # wdt:P86 composer
  ?composer wdt:P570 ?dateOD ;              # wdt:P570 date of death (Monteverdi died in 1643)
            wdt:P20 ?placeOD                # wdt:P20 place of death
            FILTER(YEAR(?dateOD) > 1000 
                   && YEAR(?dateOD) < 1644) . 
  ?placeOD wdt:P17 ?country .
  # positive list is ugly, I don't know better yet.
  VALUES ?countries {
      wd:Q28 wd:Q29 wd:Q31 wd:Q34 wd:Q36 wd:Q38 wd:Q39 wd:Q40 wd:Q41 wd:Q45 wd:Q55 wd:Q142 wd:Q145 wd:Q183 wd:Q212 wd:Q213 wd:Q219 wd:Q38872
      #  Mexico https://de.wikipedia.org/wiki/Gaspar_Fernandes, currently only one of his works is registered, so having CW>1 will loose him
      wd:Q96 } .
  FILTER(?country IN (?countries)) .
  OPTIONAL { ?composer wdt:P18 ?img . }     # wdt:P18 image
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
}
GROUP BY ?composer ?composerLabel ?dateOD ?placeODLabel
HAVING (?CW >= 2)
LIMIT 180
```

## Composers - influence on each other

```
#defaultView:Graph
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
# I had to provide pictures for both sides, there seems to be some magic I don't understand yet.
SELECT  ?composerB ?composerBLabel ?imgB ?composerA ?composerALabel ?imgA
WHERE {
  ?composerA wdt:P106 wd:Q36834 .
  ?composerB wdt:P106 wd:Q36834 .
  ?composerA wdt:P737 ?composerB . # :influencedBy ?influencer
  ?composerB wdt:P570 ?dateOD ;              # wdt:P570 date of death (Monteverdi died in 1643)
            wdt:P20 ?placeOD                # wdt:P20 place of death
            FILTER(YEAR(?dateOD) > 1000 
                   && YEAR(?dateOD) < 1945) . 
#  {select (count(?composerBB) as ?infCount) where {?composerA wdt:P737 ?composerBB .} group by ?composerA }
  OPTIONAL { ?composerB wdt:P18 ?imgB . }
  OPTIONAL { ?composerA wdt:P18 ?imgA . }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
}
limit 200
```

## Botany

### Fern families
```
SELECT DISTINCT ?fernOrder ?fernOrderLabel (COUNT(DISTINCT(?fernGenus)) as ?genera)
WHERE
{
  { ?fern wdt:P171* wd:Q373615 . } UNION {?fern wdt:P171* wd:Q178286 .} 
  ?fern wdt:P171* ?fernGenus .
  ?fernGenus wdt:P105 wd:Q34740 . # wd:Q767728 wd:Q7432 . # Q34740 .
  ?fernGenus wdt:P171* ?fernFamily .
  ?fernFamily wdt:P105 wd:Q35409 . # wd:Q767728 wd:Q7432 . # Q34740 .
  ?fernFamily wdt:P171* ?fernOrder .
  ?fernOrder wdt:P105 wd:Q36602 .
  
  SERVICE wikibase:label {
    bd:serviceParam wikibase:language "[AUTO_LANGUAGE],de,el,en,es"
  }
}
GROUP BY ?fernOrder ?fernOrderLabel
ORDER BY ?fernOrderLabel
LIMIT 180
```

