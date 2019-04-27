# DB assignement 12

sudo docker run -d -v $(pwd):/var/lib/neo4j/import --name neo4j --rm --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=neo4j/fancy99Doorknob neo4j
