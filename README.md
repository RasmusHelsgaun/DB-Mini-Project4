# DB-Mini-Project2

### Assignment link
[Mini Project 2: NoSQL Databases](https://datsoftlyngby.github.io/soft2020spring/resources/b27d96f6-MP2.pdf)

### Group:
* Pernille Lørup
* Adam Lass
* Rasmus Helsgaun

## Database choices:
* Neo4j
* MongoDB

## Dataset
The dataset being used in this assignment is an IMDB movie dataset downloaded from [kaggle](https://www.kaggle.com/stefanoleone992/imdb-extensive-dataset#IMDb%20movies.csv), and is a csv file that contains a little over 81.000 movies.  

## Assignment

*Your task is to select two or more databases of different NoSQL types and to compare their features
and performance in storing, scaling, providing, and processing big data.*

### Your solution includes

### 1) preparing a large data source and loading it into both databases

We chose to implement a python script that loads both datasets into our chosen databases. 

**Neo4j**:  

```python
for k,v in movies.iterrows():
    id = v.imdb_title_id
    title = v.title
    year = v.year
    original_title = v.original_title
    date_published = v.date_published
    genre = v.genre
    try:
        actor_names = v.actors.split(",")
    except:
        actor_names = []
        
    result = session.run("CREATE(m:Movie { title:$title, year:$year, original_title:$original_title , date_published:$date_published, genre:$genre }) RETURN m",
                        title=title, year=year, original_title=original_title, date_published=date_published, genre=genre)
    result = result.single()[0]
    
    for actor_name in actor_names:
        actor_name = actor_name.strip()
        match = session.run("match (a) WHERE a.name=$name return a",name=actor_name)
        if match.peek() == None:
            match = session.write_transaction(lambda tx, name: tx.run("CREATE(a:Name { name:$name } )RETURN a", name=name), actor_name)
        
        match = match.single()[0]
        result1 = session.run("""
            MATCH (name: Name{ name:$name })
            MATCH (movie: Movie{ title: $title })
            CREATE (movie)-[r1:FEATURES]->(name)
        """, name=actor_name, title=title)
 
``` 

Because the movie dataset contained several redundant columns, we made the decision to only include the columns: id, title, year, original title, date published, genre, and actor names.  
After deciding which columns we wanted to include, we also chose to only load 5000 rows because Neo4j is not loading large datasets very fast. With that being said, when the data has been loaded it works very well and is easy to use. 

After approximately 17 minutes of loading the dataset it had the size of 190,46 MB. 
One of the great features of using Neo4j is the visualization of the data. 

The visualization below shows 25 movies and their featured actors. 

![small](./images/small.png.)

We also tried to visualize 3000 movies and their featured actors which looked like this: 

![3000](./images/3000.png.)


**MongoDB**:  

```python
for k,v in movies.iterrows():
    id = v.imdb_title_id
    title = v.title
    year = v.year
    original_title = v.original_title
    date_published = v.date_published
    genre = v.genre
    try:
        actor_names = v.actors.split(",")
        actor_names = list(map(lambda an: an.strip(), actor_names))
    except:
        actor_names = []
    
    movie.insert_one({
        "id":id,
        "title":title,
        "year": year,
        "original_title": original_title,
        "date_published": date_published,
        "genre": genre,
        "actor_names": actor_names
    })
    
    for actor_name in actor_names:
        actor_name = actor_name.strip()


```
 
It only took 3.62 seconds to CREATE the dataset in MongoDB which was a big improvement to the Neo4j solution!

MongoDB doesn't have the same kind of visualization as Neo4j, but the data is listed as below in the Compass GUI. 

![mongodb](./images/mongodata.png.)



#### 2) selecting relevant database operations, which can be used to compare the databases

* **Get most featured actor:**

*Neo4j* 

```
MATCH (a)-[:FEATURES]->(m)
RETURN m, COLLECT(a) as actor
ORDER BY SIZE(actor) DESC LIMIT 1
```

This query took **4.55 ms** to run but a few seconds to get visualized.  

![henryoneill](./images/henryoneill.png.)


*MongoDB*  

```sql
ff
```


- selecting appropriate criteria for comparison, such as access time, storage space,
complexity, versioning, security, or similar
- creating demo code for testing the selected database operations against the selected
comparison criteria
- reporting the results and conclusions.



