---
title: "the-marvel universe in agensgraph - part1 "
date: 2019-02-01 10:09:28 -0400
categories:
  - agensgraph
  - marvel
  - graphdatabase
tags:
  - categories
  - edge case
---

The Marvel Universe is American comic book series such as the Avengers, the Iron Man and others. It consisting of thousands of separate universes.
I like the Iron Man and the Guardians of the Galaxy series.

Well, I explain how to import the Marvel Universe dataset into agensgraph database. I referrenced these datasets from [Kaggle](https://www.kaggle.com/csanhueza/the-marvel-universe-social-network)

![marvel](https://raw.githubusercontent.com/illmatik12/illmatik12.github.io/master/_screenshots/marvel_universe.jpg)



### Requirements
* Agensgraph
* The Marvel universe datasets. [(the-marvel-universe-social-network.zip)](https://www.kaggle.com/csanhueza/the-marvel-universe-social-network/downloads/the-marvel-universe-social-network.zip/1)

### data structure

| Files  | Columns  |Description
| ------------ | ------------ |
|edge.csv | hero / comic |  Heroes and the comic in which they appear. |
|hero-network.csv | hero1 / hero2 |  Edges between heroes that appear in the same comic.|
|edges.csv | node/ type | Node name and type |


### Graph Modeling 
#### simple graph model
![marvel graph](https://raw.githubusercontent.com/illmatik12/illmatik12.github.io/master/_screenshots/marvel_universe_graph_model.jpg)

* Vertex Label : hero {name} / comic {title}
* Edge Label : appeared_in / knows

##### this means 
* **hero** appeared in **comic**.
* **hero** knows **hero**.



### Data Import

#### unzip dataset files.
unzip files into /path/to/
in my case, path is /home/agens/marvel_universe

#### import query
##### scripts using agens cli command.
```sql
CREATE EXTENSION file_fdw;
CREATE SERVER import_server FOREIGN DATA WRAPPER file_fdw;
CREATE FOREIGN TABLE nodes
(
node TEXT,
type TEXT
)
SERVER import_server
OPTIONS( FORMAT 'csv', HEADER 'true', FILENAME '/home/agens/marvel_universe/nodes.csv', delimiter ',');

CREATE FOREIGN TABLE hero_network
(
hero1 TEXT,
hero2 TEXT
)
SERVER import_server
OPTIONS( FORMAT 'csv', HEADER 'true', FILENAME '/home/agens/marvel_universe/hero-network.csv', delimiter ',');

CREATE FOREIGN TABLE edges
(
hero TEXT,
comic TEXT
)
SERVER import_server
OPTIONS( FORMAT 'csv', HEADER 'true', FILENAME '/home/agens/marvel_universe/edges.csv', delimiter ',');


create graph marvel_graph;

set graph_path = marvel_graph;

# hero vertex create 
LOAD FROM nodes AS source 
WITH   { name : to_jsonb( source.node ), type : to_jsonb( source.type ) } as nodes 
WHERE nodes.type = 'hero'
CREATE (a:hero {name: nodes.name } )
;

# comic vertex create 
LOAD FROM nodes AS source 
WITH   { title : to_jsonb( source.node ), type : to_jsonb( source.type ) } as nodes 
WHERE nodes.type = 'comic'
CREATE (a:comic { title: nodes.title } )
;

# appered_in edge create 
LOAD FROM edges AS source 
MATCH (h:hero {name : to_jsonb(source.hero) })
MATCH (c:comic {title : to_jsonb(source.comic) })
CREATE (h)-[:appered_in]->(c)
;

# knows edge create 
CREATE ELABEL KNOWS;
LOAD FROM hero_network AS source 
MATCH (h1:hero {name : to_jsonb(source.hero1) })
MATCH (h2:hero {name : to_jsonb(source.hero2) })
MERGE (h1)-[:knows]->(h2)
;

# check import result
MATCH (a:hero )-[e:knows]->(b:hero)
WHERE a.name = 'CAPTAIN AMERICA'
RETURN * 
LIMIT 20
;

```
##### Result
```bash
agens=# 
agens=# create graph marvel_graph;
CREATE GRAPH
agens=# set graph_path = marvel_graph;
SET
agens=# LOAD FROM nodes AS source 
agens-# WITH   { name : to_jsonb( source.node ), type : to_jsonb( source.type ) } as nodes 
agens-# WHERE nodes.type = 'hero'
agens-# CREATE (a:hero {name: nodes.name } )
agens-# ;
GRAPH WRITE (INSERT VERTEX 6439, INSERT EDGE 0)
agens=# LOAD FROM nodes AS source 
agens-# WITH   { title : to_jsonb( source.node ), type : to_jsonb( source.type ) } as nodes 
agens-# WHERE nodes.type = 'comic'
agens-# CREATE (a:comic { title: nodes.title } )
agens-# ;
GRAPH WRITE (INSERT VERTEX 12651, INSERT EDGE 0)
agens=# LOAD FROM edges AS source 
agens-# MATCH (h:hero {name : to_jsonb(source.hero) })
agens-# MATCH (c:comic {title : to_jsonb(source.comic) })
agens-# CREATE (h)-[:appered_in]->(c)
agens-# ;
GRAPH WRITE (INSERT VERTEX 0, INSERT EDGE 94527)
agens=# 
agens=# CREATE ELABEL KNOWS;
CREATE ELABEL
agens=# LOAD FROM hero_network AS source 
agens-# MATCH (h1:hero {name : to_jsonb(source.hero1) })
agens-# MATCH (h2:hero {name : to_jsonb(source.hero2) })
agens-# MERGE (h1)-[:knows]->(h2)
agens-# ;
GRAPH WRITE (INSERT VERTEX 0, INSERT EDGE 183645)
agens=# 
agens=# MATCH (a:hero )-[e:knows]->(b:hero)
agens-# WHERE a.name = 'CAPTAIN AMERICA'
agens-# RETURN a.name, labels(e), b.name
agens-# LIMIT 20
agens-# ;
                   a                    |              e               |                      b                      
----------------------------------------+------------------------------+---------------------------------------------
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.66][3.858,3.2]{}     | hero[3.2]{"name": "3-D MAN/CHARLES CHAN"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.222][3.858,3.8]{}    | hero[3.8]{"name": "ABOMINATION/EMIL BLO"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.504][3.858,3.13]{}   | hero[3.13]{"name": "ABSORBING MAN/CARL C"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.574][3.858,3.16]{}   | hero[3.16]{"name": "ACHEBE, REVEREND DOC"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.617][3.858,3.18]{}   | hero[3.18]{"name": "ACHILLES II/HELMUT"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.685][3.858,3.21]{}   | hero[3.21]{"name": "ADAMS, CINDY"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.741][3.858,3.25]{}   | hero[3.25]{"name": "ADAMS, NICOLE NIKKI"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.993][3.858,3.41]{}   | hero[3.41]{"name": "AGAMEMNON III/"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1103][3.858,3.48]{}  | hero[3.48]{"name": "AGENT AXIS/"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1511][3.858,3.71]{}  | hero[3.71]{"name": "AKUTAGAWA, OSAMU"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1520][3.858,3.72]{}  | hero[3.72]{"name": "ALANYA"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1594][3.858,3.78]{}  | hero[3.78]{"name": "ALDEN, PROF. MEREDIT"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1700][3.858,3.81]{}  | hero[3.81]{"name": "ALEXANDER, CARRIE"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1842][3.858,3.92]{}  | hero[3.92]{"name": "ALVAREZ, FELIX"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1942][3.858,3.97]{}  | hero[3.97]{"name": "AMERICAN EAGLE III/J"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.1958][3.858,3.100]{} | hero[3.100]{"name": "AMERICOP/"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.2018][3.858,3.105]{} | hero[3.105]{"name": "AMPHIBIAN/KINGLEY RI"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.2110][3.858,3.107]{} | hero[3.107]{"name": "ANACONDA/BLANCHE SIT"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.2540][3.858,3.130]{} | hero[3.130]{"name": "ANELLE"}
 hero[3.858]{"name": "CAPTAIN AMERICA"} | knows[6.2665][3.858,3.133]{} | hero[3.133]{"name": "ANGEL DOPPELGANGER"}
(20 rows)

agens=# 

```

### Conclusion

graph data/model is a such a complicated. but graph database can solve the problem step by step . 
As you can see above, Agensgraph can store csv data with graph model (ex: social network data) 
More information you can visit website [https://bitnine.net](https://bitnine.net)

### References

[1] [https://en.wikipedia.org/wiki/Marvel_Universe](https://en.wikipedia.org/wiki/Marvel_Universe)

[2] [https://en.wikipedia.org/wiki/Marvel_Universe](https://www.kaggle.com/csanhueza/the-marvel-universe-social-network)
