# Setup Virtuoso RDF Store for the paper ``Example-Driven Exploratory Analytics over Knowledge Graphs''


## Install and setup docker image of empty database 


```bash

docker pull openlink/virtuoso-opensource-7:7.2.10-alpine

mkdir -p database
cp  virtuoso.ini.example database/virtuoso.ini

mkdir -p import

docker run --name vos -d -v `pwd`/database:/opt/virtuoso-opensource/database \
           -v `pwd`/import:/import \
           -t -p 1111:1111 -p 8890:8890 -i openlink/virtuoso-opensource-7:7.2.10-alpine 
```

**Test system is running**

```bash
docker exec -it vos /opt/virtuoso-opensource/bin/isql  exec="SPARQL SELECT COUNT(*) WHERE {?s ?p ?o };"
```


## Load the various Databases


### BONSAI/EXIOBASE production database

#### Download and uncompress RDF files


```bash

wget -O exiobase.tar.gz https://zenodo.org/record/8205610/files/exiobase.tar.gz?download=1
wget -O ontology.tar.gz https://zenodo.org/record/8205610/files/ontology.tar.gz?download=1
wget -O rdf.tar.gz https://zenodo.org/record/8205610/files/rdf.tar.gz?download=1

wget -O load-ontology.sql https://zenodo.org/record/8205610/files/load-ontology.sql?download=1
wget -O load-exiobase-flows.sql https://zenodo.org/record/8205610/files/load-exiobase-flows.sql?download=1
wget -O load-rdf.sql https://zenodo.org/record/8205610/files/load-rdf.sql?download=1

```

**Uncompress and move into `import` folder**

```bash
tar -xvf exiobase.tar.gz 
tar -xvf ontology.tar.gz
tar -xvf rdf.tar.gz

mv exiobase import/
mv ontology import/
mv rdf import/

rm *.tar.gz

mv *.sql import/

```

#### Import Data 

```bash

docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-ontology.sql"
docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-rdf.sql" 
docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-exiobase-flows.sql"

```

#### Test successful import


**Run count query**

```bash

docker exec -it vos \
/opt/virtuoso-opensource/bin/isql  \
exec="SPARQL SELECT ?g (COUNT(*) as ?nTriple) WHERE { GRAPH ?g { ?s ?p ?o } } GROUP BY ?g;"

```


**Expected output**


|----------------------------------------------------------|-----------|
| g                                                        | nTriple   |
|----------------------------------------------------------|-----------|
| http://rdf.bonsai.uno/data/exiobase3_3_17/huse           | 223964323 |
| http://rdf.bonsai.uno/data/exiobase3_3_17/hsup           | 166763    |
| http://rdf.bonsai.uno/prov/ystafdb                       | 49872     |
| http://rdf.bonsai.uno/activitytype/ystafdb               | 6584      |
| http://rdf.bonsai.uno/location/ystafdb                   | 5072      |
| http://rdf.bonsai.uno/time                               | 3985      |
| http://rdf.bonsai.uno/flowobject/exiobase3_3_17          | 614       |
| http://rdf.bonsai.uno/flowobject/us_epa_elem             | 231       |
| http://rdf.bonsai.uno/activitytype/entsoe                | 74        |
| http://rdf.bonsai.uno/unit                               | 47        |
| http://rdf.bonsai.uno/prov/exiobase3_3_17                | 30        |
| http://rdf.bonsai.uno/foaf/exiobase3_3_17                | 21        |
| http://rdf.bonsai.uno/activitytype/lcia/climate_change   | 20        |
| http://rdf.bonsai.uno/flowobject/lcia/climate_change     | 20        |
| http://rdf.bonsai.uno/flowobject/core/electricity_grid   | 17        |
| http://ontology.bonsai.uno/core                          | 132       |
|----------------------------------------------------------|-----------|


### Eurostat `migr_asyappctzm` QB4OLAP dataset

#### Download and uncompress RDF files


```bash

wget -O eurostat_schema_QB4OLAP_v1.3.ttl https://zenodo.org/record/8211126/files/eurostat_schema_QB4OLAP_v1.3.ttl?download=1
wget -O migr_asyappctzm_data.ttl.gz https://zenodo.org/record/8211126/files/migr_asyappctzm_data.ttl.gz?download=1
wget -O migr_asyappctzm_instances.ttl.gz https://zenodo.org/record/8211126/files/migr_asyappctzm_instances.ttl?download=1

wget -O load-eurostat.sql https://zenodo.org/record/8211126/files/load-eurostat.sql?download=1

```

**Move into `import` folder**

```bash
mdkir -p import

mv eurostat_schema_QB4OLAP_v1.3.ttl import/
mv migr_asyappctzm_data.ttl.gz import/
mv migr_asyappctzm_instances.ttl.gz import/

mv *.sql import/

```

#### Import Data 

```bash

docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-eurostat.sql"

```

#### Test successful import


**Run count query**

```bash

docker exec -it vos \
/opt/virtuoso-opensource/bin/isql  \
exec="SPARQL SELECT ?g (COUNT(*) as ?nTriple) WHERE { GRAPH ?g { ?s ?p ?o } } GROUP BY ?g;"

```

**Expected output**


|------------------------------------------------------------|-----------|
| g                                                          | nTriple   |
|------------------------------------------------------------|-----------|
| http://www.fing.edu.uy/inco/cubes/schemas/migr_asyapp      | 178       |
| http://eurostat.linked-statistics.org/data/migr_asyappctzm | 150603161 |
|------------------------------------------------------------|-----------|






