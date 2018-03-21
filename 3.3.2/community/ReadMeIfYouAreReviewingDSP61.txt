#This file contains commands in order to facilitate reviewing and testing the solution for DSP-61.
#To test this solution for DSP-61, please execute the following commands. 
#Lines escaped with a "#" ellaborate on these commands.

#Start a container locally with neo4j and specific entrypoint
sudo docker run 
-v ~/docker-neo4j-publish/3.3.2/community/docker-entrypoint.sh:/docker-entrypoint.sh 
--entrypoint '/docker-entrypoint.sh' 
-e NEO4J_USER_NAME=andrea 
-e NEO4J_GROUP_NAME=andreas_group
-e NEO4J_USER_ID=1234 -e NEO4J_GROUP_ID=1234 
-p 7474:7474 
--ulimit nofile=40000:40000 neo4j:3.3.2 neo4j
#Here is the previous command in one line: 
#sudo docker run -v ~/docker-neo4j-publish/3.3.2/community/docker-entrypoint.sh:/docker-entrypoint.sh --entrypoint '/docker-entrypoint.sh' -e NEO4J_USER_NAME=andrea -e NEO4J_GROUP_NAME=andreas_group -e NEO4J_USER_ID=1234 -e NEO4J_GROUP_ID=1234 -p 7474:7474 --ulimit nofile=40000:40000 neo4j:3.3.2 neo4j

#In the container (preferably in another terminal) create a test.csv and verify the permissions for this file. 
#Copy CONTAINER_ID and CONTAINER_NAME from the next command's output
sudo docker ps
sudo docker exec -it $CONTAINER_ID touch import/test.csv
sudo docker exec -it $CONTAINER_ID ls -l import

#Log into neo4j. Enter username and password for this. 
sudo docker exec -ti $CONTAINER_NAME /var/lib/neo4j/bin/cypher-shell
#Default username is "neo4j" and default password is "neo4j".

#Import the test.csv to neo4j
LOAD CSV FROM "file:///test.csv" AS line
    RETURN count(*);

#Expected output: 
#	+----------+
#	| count(*) |
#	+----------+
#	| 0        |
#	+----------+

#Exit neo4j with 
:exit 

#In the container change the permissions for test.csv and verify this change. 
sudo docker exec -it $CONTAINER_ID chmod 700 import/test.csv
sudo docker exec -it $CONTAINER_ID ls -l import
 

#Log into neo4j. Enter username and password for this. 
sudo docker exec -ti $CONTAINER_NAME /var/lib/neo4j/bin/cypher-shell
#Default username is "neo4j" and default password is "neo4j".

#Import the test.csv to neo4j
LOAD CSV FROM "file:///test.csv" AS line
    RETURN count(*);

#Expected output (error):
#	"Couldn't load the external resource at: file:/var/lib/neo4j/import/test.csv"
