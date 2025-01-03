# Distributed Search

There are 3 main component in a search system:
  * A Crawler : Fetches content and creates documents(json or similar).
  * An Indexer : Creates a searchable index. 
  * A Searcher : It responds to search queries by running the search query on indexes.

 ![ Component of Distributed Search System](assests/search1.png) 

## Functional Requirement:
 * Search

## Non-functional Requirement

 * Availability
 * Scalability
 * Fast Search on Big Data
 * Reduced Cost

The response time to a search query depends on a few factors:
 * The data organization strategy in the database.
 * The size of the data.
 * The processing speed and the RAM of the machine that’s used to build the index and process the search query.

Running the search queries on billions of documents that are **document-level indexed** will be a slow process. It may take minuts or even hours. 

### Inverted Index

 * Its HashMap like data structure that employs a document-term matrix.
 * Instead of storing the complete document as it is, it split the documents into individual words , discards some low importance words and then create indexes for frquently occurring words.
 * For each term, the index computes the following information:
   * List of document in which term appeared.
   * The Frequency with which the term appears in each document.
   * The position of the term in each document.
 * Reduces the time of counting the occurance of a word in each document at run time because we have mapping against each term.
 * There is storage overhead  for maintaining the inverted index along with the actual document.
 * Maintenance overhead (while adding or deleting the document we need to update the inverted index)

## High-level design:

 There are 2 phases of such systems 
  1) **Online Phase** : consist of searching for results against the search query by the user.
  2) **Offline Phase** : It involves data crawling and indexing

 ![High Level Design of Distributed Search System ](assests/search2.png) 

  * **Crawler** : It collect the content from intended resources. From the extracted content from resources it generates the JSON document and store it into the storage.
  * **Indexer**: It fetches the document from a distributed storage and index them using MapReduce.
  * Distributed storage is used to store the document and its idexes.
  * The **Searcher** parses the search string and searches for mapping from the index that are stored in distributed storage.

### Distributed indexing and searching 

To develop an index in a distributed fashion, we employ large number of low-cost machines and partition or divide the documents based on resourses they have. 
This process requires us to partition or split the input data among these nodes.  How do we perform these partitioning ?

 * Document Partioning
 * Term Partitioning 


#### Indexing Phase
 * The *clusture manager* splits the input documents into N number of partition. Clusture Manager monitor the health of each node through the perioidic heartbeat.
 * After making the partition clusture manager runs the indexing algorithm for all partitions and creates tiny inverted index, which is stored on local storage of nodes. 
 
#### Searching Phase
 * When a user query comes in, we run parallel searches on each tiny inverted index.
 * The search result from each inverted tiny index is a list of results which is merged by the merger.
 * Merger sorst the list of documents from the aggregated mapping list(created on previous step) based on the frequncy of the term in each document.
 * Sorted list of document is returned to the user.
We can make replicas of the indexing nodes that produce inverted index for assigned partition. We continue using the same architecture but instead of having only one group of nodes , we can have R group of nodes.The number of replicas can expand or shrink. 

#### Problem with this design ?
 1) Colocated indexing and searching
 2) Index recomputation 
