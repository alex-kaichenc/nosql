# Graph Database

+ In Neo4j querying non-existent property return **null**.
+ **Case Sensitive**

## Relationship

**Relationship can be created by simply:**

    CREATE (Keanu)-[:ACTED_IN {roles:['Neo']}]->(TheMatrix)

**In Neo4j, ALL relationships are directed.**

However, you can have the notion of undirected edges at query time. Just remove the direction from your MATCH query:

    MATCH (p:Person)-[r:FRIEND_WITH]-(b:Person) where p.name = "PersonB" 


## CREATE

    CREATE (TheMatrix:Movie {title:'The Matrix', released:1999, tagline:'Welcome to the Real World'})
    CREATE (JoelS:Person {name:'Joel Silver', born:1952})
    CREATE
      (Keanu)-[:ACTED_IN {roles:['Neo']}]->(TheMatrix),
      (LillyW)-[:DIRECTED]->(TheMatrix),
      (JoelS)-[:PRODUCED]->(TheMatrix)

    CREATE (Emil:Person {name:"Emil Eifrem", born:1978})
    CREATE (Emil)-[:ACTED_IN {roles:["Emil"]}]->(TheMatrix)

## WITH

    WITH TomH as a
    MATCH (a)-[:ACTED_IN]->(m)<-[:DIRECTED]-(d) RETURN a,m,d LIMIT 10
    ;

## MATCH

Find the actor named "Tom Hanks"...

    MATCH (tom {name: "Tom Hanks"}) RETURN tom

Find the movie with title "Cloud Atlas"...

    MATCH (cloudAtlas {title: "Cloud Atlas"}) RETURN cloudAtlas

Find 10 people...

    MATCH (people:Person) RETURN people.name LIMIT 10

Find movies released in the 1990s...

    MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title

Finds the titles of the movies reviewed by Jessica Thompson. 

    MATCH (reviewer:Person {name: "Jessica Thompson"})-[:REVIEWED]->(m:Movie) RETURN m.title;


## Graph traversal

List all Tom Hanks movies...

    MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom, tomHanksMovies

Who directed "Cloud Atlas"?

    MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name


Tom Hanks' co-actors...

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name

How people are related to "Cloud Atlas"...

    MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo

What is the name of the person that produced When Harry Met Sally?

    MATCH (p:Person)-[:PRODUCED]->(m:Movie {title: "When Harry Met Sally"}) RETURN p.name;

Solve shortest path, all nodes that are four node away (the * is actually just one asterisk)

Movies and actors up to 4 "hops" away from Kevin Bacon

    MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood) RETURN DISTINCT hollywood;

Shortest path

    MATCH p=shortestPath(
      (bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
    )
    RETURN p;

## Recommend: Relationship adjacency

Find actors that Tom Hanks hasn't yet worked with, but his co-actors have.

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
          (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors)
    WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors
    RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC

Find someone who can introduce Tom to his potential co-actor.

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
          (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"})
    RETURN tom, m, coActors, m2, cruise

Other application:

Entity: customer, product
Relationship: (customer)-[:PURCHASED]->(product), (customer)-[:CLASSMATE]->(customer);
Recommendation: Find product that someone have not yet purchased, but his / her classmates have.


## Product Catalog

**Load records** (default type: string)

    LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/products.csv" AS row
    CREATE (n:Product)
    SET n = row,
      n.unitPrice = toFloat(row.unitPrice),
      n.unitsInStock = toInteger(row.unitsInStock), n.unitsOnOrder = toInteger(row.unitsOnOrder),
      n.reorderLevel = toInteger(row.reorderLevel), n.discontinued = (row.discontinued <> "0")

Create **indexes** (just regular index)

    CREATE INDEX ON :Product(productID)

Create data relationships (**foreign key reference**, etc)

    MATCH (p:Product),(c:Category)
    WHERE p.categoryID = c.categoryID
    CREATE (p)-[:PART_OF]->(c)

_Note you only need to compare property values like this when first creating relationships_

**Query using patterns**

    MATCH (s:Supplier)-->(:Product)-->(c:Category)
    RETURN s.companyName as Company, collect(distinct c.categoryName) as Categories

    Company 		Categories
    "Lyngbysild"	["Seafood"]
    "G'day"			["Grains/Cereals", "Meat/Poultry", "Produce"]

**Many-to-many relationships: called "join table" (Neo4j), represented as a relationship with properties.**

Here, we'll directly **promote** each OrderDetail record into a relationship in the graph.

    LOAD CSV WITH HEADERS FROM "http://data.neo4j.com/northwind/order-details.csv" AS row
    MATCH (p:Product), (o:Order)
    WHERE p.productID = row.productID AND o.orderID = row.orderID
    CREATE (o)-[details:ORDERS]->(p)
    SET details = row,
      details.quantity = toInteger(row.quantity)

Query using pattern

    MATCH (cust:Customer)-[:PURCHASED]->(:Order)-[o:ORDERS]->(p:Product),
          (p)-[:PART_OF]->(c:Category {categoryName:"Produce"})
    RETURN DISTINCT cust.contactName as CustomerName, SUM(o.quantity) AS TotalProductsPurchased

Find the names of the products that were ordered on November 11, 1996

    MATCH (o:Order)-[r:ORDERS]->(p:Product) RETURN o;

    MATCH (o:Order)-[r:ORDERS]->(p:Product) 
    WHERE o.orderDate = "1997-01-14 00:00:00.000"

    MATCH (o:Order)-[r:ORDERS]->(p:Product) 
    WHERE o.orderDate = "1996-11-11 00:00:00.000"
    RETURN p.productName;


## Lab 4

    CREATE (Dan: People {firstName:'Dan'})
    CREATE (Dave: People {firstName:'Dave'})
    CREATE (Doug: People {firstName:'Doug'})
    CREATE (Dale: People {firstName:'Dale'})
    CREATE (Darren: People {firstName:'Darren'})
    CREATE (Dane: People {firstName:'Dane'})

    CREATE
      (Dan)-[:FRIEND]->(Dave),
      (Dave)-[:FRIEND]->(Dan),

      (Dave)-[:FRIEND]->(Doug),
      (Doug)-[:FRIEND]->(Dave),

      (Darren)-[:FRIEND]->(Dave),
      (Dave)-[:FRIEND]->(Darren),

      (Darren)-[:FRIEND]->(Dane),
      (Dane)-[:FRIEND]->(Darren),

      (Dane)-[:FRIEND]->(Dale),
      (Dale)-[:FRIEND]->(Dane),

      (Dale)-[:FRIEND]->(Darren),
      (Darren)-[:FRIEND]->(Dale)
    ;

    MATCH (p1)-[:FRIEND]-(p2)
    RETURN p1.firstName AS subject, p2.firstName AS fof;

    MATCH (p1)-[:FRIEND]->(p2)-[:FRIEND]->(p3)
    RETURN DISTINCT p1.firstName AS subject, p3.firstName AS fof;

**Errors**: in cycle, p3 may be p2 of a p1

    MATCH (p1)-[:FRIEND]->(p2)-[:FRIEND]->(p3)
    WHERE p1 <> p3
    RETURN DISTINCT p1.firstName AS subject, p3.firstName AS fof;

Won't work: the query allow **repeated** path

    MATCH (p1)-[\*1..2]->(p3)
    WHERE p1 <> p3
    RETURN DISTINCT p1.firstName AS subject, p3.firstName AS fof;

Good: but this will return both p1:p3 and p3:p1

    MATCH (p1)-[:FRIEND]->(p2)-[:FRIEND]->(p3)
    WHERE p1 <> p3
        AND (NOT (p1)-[:FRIEND]->(p3))
    RETURN DISTINCT p1.firstName AS subject, p3.firstName AS fof;

Cannot prevent that...

    MATCH (p1)-[:FRIEND]-(p2)-[:FRIEND]-(p3)
    WHERE p1 <> p3
        AND (NOT (p1)-[:FRIEND]-(p3))
    RETURN DISTINCT p1.firstName AS subject, p3.firstName AS fof;


