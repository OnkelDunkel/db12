# DB assignement 12

## Ex1

sudo docker run -d -v $(pwd):/var/lib/neo4j/import --name neo4j --rm --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=neo4j/fancy99Doorknob neo4j

I used below query to load the data and add mentions lists (uncomment "LIMIT 100" for shorter execution time):

    LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row 
        FIELDTERMINATOR ";"
    WITH row //LIMIT 100
    WHERE NOT row.Latitude = "" AND NOT row.Longitude = ""
    MERGE (t:Tweet
        {
            username:row["User Name"],
            nickname: row.Nickname,
            place: row["Place (as appears on Bio)"],
            latitude: toFloat(row.Latitude),
            longitude: toFloat(row.Longitude),
            mentions: [
                m in filter(m in split(row["Tweet content"]," ") 
                where m starts with "@" and size(m) > 1) | right(m,size(m)-1)
            ],
            content: row["Tweet content"]
        }
    );

## Ex2

### Use the mentions list of each tweet to create a new set of nodes labeled "Tweeters", whith a "Mentions" relation.

    MATCH(t:Tweet) 
    WITH t
    FOREACH (
        m in t.mentions | 
        MERGE (tu:Tweeters { username:t.username} )
        CREATE (tu)-[:MENTIONS]->(t)
    );

### Create a relation "Tweeted" between Tweeters and Tweet.

    MATCH(t:Tweet) 
    WITH t
    MERGE (tu:Tweeters { username:t.username} )
    CREATE (tu)-[:TWEETED]->(t);

## Ex3

It is very unclear to me what is requested from the question. However I've made a query that creates relations between all tweets containing a property with the calculated distance.

    MATCH (t1:Tweet), (t2:Tweet)
    WHERE NOT (t2)-[:DISTANCE]->(t1) AND NOT (t1) = (t2)
    MERGE 
    (t1)-
    [d:DISTANCE 
        {
            distance: toInteger(
                round(distance
                (
                    point({ longitude: t1.longitude, latitude: t1.latitude }),
                    point({ longitude: t2.longitude, latitude: t2.latitude })
                )) / 1000
            )
        }
    ]
    ->(t2)
    RETURN d.distance, t1.username, t1.nickname, t2.username, t2.nickname;





