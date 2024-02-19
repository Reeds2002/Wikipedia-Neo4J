## Technology Stack and Project Design

A project in which I organised the data from a CSV file containing data of over 5 million lines Wikipedia categories and subcategories in a graph.

The chosen technology was Neo4J along with the APOC library. For the actual application I used a bash script which imports the data and creates relationships from the CSV file through cypher-shell commands. With Neo4J being a graph database. Each category is represented as a node in the graph and that node points to it's subcategories (children).  

-------------

## Implementation Process

The import query looks like this: 

```cypher
CREATE CONSTRAINT import_constraint ON (c:Category) ASSERT c.Name IS UNIQUE;

CALL apoc.periodic.iterate(
  'LOAD CSV FROM "file:///taxonomy_iw.csv" AS line RETURN line',
  'WITH line[0] AS cat1Name, line[1] AS cat2Name
   MERGE (n:Category {Name: cat1Name})
   MERGE (m:Category {Name: cat2Name})
   MERGE (n)-[:Super_Category]->(m)',
  {batchSize: 500, parallel: true}
)
```

To decrease the import duration a unique constraint was added to the name.

Then using the APOC library I am calling the command with a batch of 500 lines of the CSV at at a time and doing this in parallel. This reduces load on the memory and helps import the data much faster. Without this I was importing the data at a rate of 12 hours and now it can be done in a couple of minutes.  

I have 11 unique queries to test the speed at which data can be interacted with in the database. Here is an example of how the queries were set up:

```bash
if [[ "$action" == 1 || "$action" == 2 || "$action" == 3 || "$action" == 4 || "$action" == 5 || "$action" == 6 || "$action" == 11 ]]; then
		echo "Enter the node you would like to perform the action on:"
		read given

fi

if [[ "$action" == 1 ]]; then
		output=$(echo "MATCH (parent:Category {Name: '$given'})
		MATCH (parent)-[:Super_Category]->(child)
		RETURN child.Name;" | time -f "\nThe action took %e seconds and the answer is the following:\n" cypher-shell -u neo4j -p password)
		
		node_names=$(echo "$output" | awk 'NR>1 {gsub(/"/,"",$0); print $0}')
		echo "$node_names"
fi
```

Once the user has entered which query they would like to execute the script will check if that goal requires a node to be chosen, the user will be asked for it with the first ```if``` statement. Then I use the preceding ```if``` statements to check which goal the user would like to commence with. Once that is done, I execute the Cypher query and store the output in a variable which is then edit with an AWK command to make it look more user-friendly. I am also using the ```time``` command included with ```bash``` to measure the execution time of the cypher query.


-------------

## Installation Guide

During the installation we will download and install the following software:

1. Neo4J 4.1
2. Curl
3. Cypher-shell
4. Java 11
5. APOC Library for Neo4J

If you already have these installed please read through the short install.sh script to make sure it will not affect current environments.

**Steps for installation:**  

1. Run the install.sh file with the following command while in the project directory: ```sudo bash install.sh.``` 
This will install Neo4J and its dependencies and start the Neo4J server.  

2. Enter the following command once the Neo4J server has started: ```cypher-shell``` . The current username is 'neo4j' and password is 'neo4j'. It will start the shell and prompt you to change the password. Please change the password to 'password'.  

3. Now, while in the project directory, run the application by entering the following command: ```bash app.sh``` .

4. Wait for the database to load and choose the option from 0 to 12 you wish to execute, in order to evaluate the time taken and reproduce the Result section.

---------------

## Results

All tests were performed on an "Acer Aspire A515-54" with the following specifications:

CPU: Intel® Core™ i5-10210U CPU @ 1.60GHz × 8  
Memory: 16GB  
Storage: 1TB SSD  
OS: Ubuntu 22.04.2 LTS 64-bit  

To measure the time I am using the ```time``` command that is included with bash as shown in the introduction.  
  

| Goals | Execution time [s] |
|----------|----------|
|    Goal 1 |    1.03 |
|    Goal 2 |    0.84 |
|    Goal 3 |    1.06 |
|    Goal 4 |    0.85 |
|    Goal 5 |    1.02 |
|    Goal 6 |    0.87 |
|    Goal 7 |    2.40 |
|    Goal 8 |    0.99 |
|    Goal 9 |    4.35 |
|   Goal 10 |   3.89 |
|   Goal 11 |   0.88 |
|   Data Import |   170.33 |  


-------------

