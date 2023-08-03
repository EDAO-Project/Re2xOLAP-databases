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


## Download and uncompress RDF files


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



## Import Data 

```bash

docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-ontology.sql"
docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-rdf.sql" 
docker exec -it vos /opt/virtuoso-opensource/bin/isql exec="LOAD /import/load-exiobase-flows.sql"

```

## Test successful import


```bash

docker exec -it vos \
/opt/virtuoso-opensource/bin/isql  \
exec="SPARQL SELECT ?g (COUNT(*) as ?nTriple) WHERE { GRAPH ?g { ?s ?p ?o } } GROUP BY ?g;"

```
