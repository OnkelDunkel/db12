# DB assignement 12

Ex1

sudo docker run -d -v $(pwd):/var/lib/neo4j/import --name neo4j --rm --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=neo4j/fancy99Doorknob neo4j

I used below query to load the data and add mentions lists (remove "LIMIT 100" to load the full data):

    LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row 
        FIELDTERMINATOR ";"
    WITH row LIMIT 100
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










MATCH (n)
DETACH DELETE n;


LOAD CSV WITH HEADERS FROM "file:///some2016UKgeotweets.csv" AS row 
    FIELDTERMINATOR ";"
WITH row LIMIT 10
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
     

match(t:Tweet) return t.mentions[0];

match(t:Tweeters) return t;




MATCH(t:Tweet) 
WITH t
FOREACH (
    m in t.mentions | 
    MERGE (tu:Tweeters { username:t.username} )
    CREATE (tu)-[:TWEETED]->(t)
);


MATCH(t:Tweet) 
WITH t
FOREACH (
    m in t.mentions | 
    MERGE (tu:Tweeters { username:t.username} )
    CREATE (tu)-[:TWEETED]->(t)
);



