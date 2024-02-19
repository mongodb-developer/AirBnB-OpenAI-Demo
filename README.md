## AirBnB-OpenAI-Demo ⚛️🧠🏠
This demo allows you to create sentence embeddings in place and then query from the prompt in python - Only requirements are a working Atlas connection and an OpenAI key

First lets install pymongo
```
pip install pymongo
```

Then lets install numpy
```
pip install numpy
```
and lastly we are going to need to install OpenAI
```
pip install openai
```
Once these packages are installed lets create a new file called vectorizer.py 
make sure to add your Atlas connection string - and your openAI key 

```
import pymongo
import openai

#Set up openai key
openai.api_key = "OPENAI API KEY"

# Connect to MongoDB
client = pymongo.MongoClient('MONGODB CONNECTION STRING')
db = client['sample_airbnb']
collection = db['listingsAndReviews']

# Retrieve documents from the collection
documents = collection.find()

# Iterate over the documents
index = 0
for document in documents:
    # print(document)

    #append various data fields in the database entry, assuming they are not null, to the input string used to generate the embedding
    embedding_input_string = ""
    if document['name']!=None:
        embedding_input_string+=document["name"]+". "
    if document['summary']!=None:
        embedding_input_string+=document['summary']+". "
    if document['space']!=None:
        embedding_input_string+=document['space']+". "
    if document['description']!=None:
        embedding_input_string+=document['description']+". "
    if document['transit']!=None:
        embedding_input_string+=document['transit']+". "
    if document['price']!=None:
        embedding_input_string+="Price per night: "+str(document['price'])+'. '
    
    #print the current input string used to generate the embedding
    print(embedding_input_string)

    #generate openai embedding based on input string
    embedding = openai.Embedding.create(input = [embedding_input_string], model="text-embedding-ada-002")['data'][0]['embedding']

    #Set corresponding openai_embedding into each document in the database
    document['openai_embedding'] = embedding

    #keep track of index to display progress in terminal
    print("Current index: {}".format(index)) #to keep track of progress
    index+=1


    # Update the document in the collection
    collection.update_one({'_id': document['_id']}, {'$set': document})

# Close the MongoDB connection
client.close()
```
When you kick this off you will see lots of sentences scrolling past that means the sentence embeddings are being created - will pause every once in a while
to show you index entries - 

Now you must go to Atlas search and create a new search index that looks like this

```
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "openai_embedding": {
        "dimensions": 1536,
        "similarity": "cosine",
        "type": "knnVector"
      }
    }
  }
}
```
Now you are ready to query the data! use AirBnB-VectorSearch.py in this repo to do that next!



