# Objective: n8n project undertaken for building a RAG pipeline to extract required information from Office documents stored in Google Drive as repository.

## Credentials Required for the setup:
- Google Drive Access Credentials, you also specify the folder that will be used as Office Folder
- Pine Cone Vector Store Credentials
- Model API (whichever you want to prefer)
**Note:** Make sure you have added required set of documents in the google drive folder which we want to make as office repository
# Setup
## Workflow 1: The first workflow talks about the storage of google drive files into vector database. In this process firstly sentences in the documents are broken down into chunks, then they are embedded into vector form and then stored pinecone DB.
1. You add two Google Drive File Nodes, one for File Created and the other for File Updated. This means this node will trigger as soon as any new file is added or updated in our office folder repository.
2. By clicking these nodes you further configure them by adding Google Drive OAuth2 API credentials with poll time of 1 minute and specifying the target folder which in our case is named n8n Testing
3. The next node we add in the flow is Download File from Google Drive. By double clicking it we further configure it. We firstly add the Google Drive OAuth2 API credentials, Select Resource as File, select Download in operation field. Fill json.id in file by ID (generally done by dragging from input window and dropping it in the required field.). Further, in options  we select the file name json.name in similar way of dragging and dropping from input window.
4. After connecting the Google Drive Create and Update node with Download Google Drive node we add next node as Pinecone Vector store.
5. We double click PineCone Vector store for its configuration. We firstly go to pinecone vector store site "pinecone.io" We click get started and click on Create index under Database. Firstly we name the index say "company-files"> select custom settings >  Under configurations select Dense and put dimension as 1536 with Cosine selection > Click create index.
6. You put this file name "company-files" in Pinecone Vector Store node under PineCone Index from list field.
Then we add OpenAI Embeddings node that connects PineCone Vector store. We double click to configure this node. We add the required node credentials, select the desired model and also add **dimensions as 1536**.
We connect default data loader node to Pinecone vector store and configure dataloader. Select type data as binary in the configuration section. Rest details can be kept default.
Next connect Recursive Character Text Splitter node with Default Data loader and put Chunk Size as 1000 and Chunk Overlap as 100.
This completes our first workflow.

## Workflow 2: The second workflow is about sending the query in vector form to pinevector DB. Extracting the required information and converting back into text with the help of retreiver and sending it back to the user by processing the required information via our AI model.
_**Note:** Most of the times you will find people making use of "When chat message received" functionality node available in n8n. However, I feel that making use of it restricts the overall functionality to n8n only. I wanted to interact with my office repository externally. Therefore, I made use of Webhook and a separate chatbot window created using html/css and java._
1. We add a webhook trigger node . This node we need to configure thoroughly, firstly we need to change HTTP method as "POST". Name the path in our own words, like I preferred to name it as mychatbot. This will make mychatbot appear in the end of the POST https link url. Keep Authentication as none. Change respond option to Using "Respond to webhook" node. Next add option as Allowed Origins (CORS)
2. Next add Edit Field node. Now this is really important node in case you are going for configuration with external webapp. For configuraion in mode you add two fields: first named as "chatinput" against the expression type String with details as ={{ $json.body.chatInput }} and second named as sessionId with expression type String with details {{ $json.body.sessionId }}. This is done because mostly the chat input comes through our chatwebapp via webhook is coming in json body. Whereas internally the AI model and other components look for json.chatInput and json.sessionId field. Next you also need to toggle on Include other input fields because it is like passing on the batton to the next player in the relay race. Where batton in this case represents the complete information we are passing to the next node and they are successfully able to execute the required operations flawlesly.
3. Next we connect to AI agent node. We can attach the chat  model of our choice. We also add system message as per our requirement. Source of prompt is selected as "Defined below" and for Prompt User Message field we add {{ $json.chatInput }}  
4. This time we also connect a Window buffer memory just to ensure that the agent doesn't loose the context of the previous conversations. For configuration we add {{ $json.sessionId }} under key field. We also select Session ID as Defined below type from the drop down. Context window length we are considering as 5.
5. Next we add Vector Store tool give the data name, like I gave it as compan_documents_tool. We add the description and add the limit as 4.
6. Then we add the PineStore Vector Retrieval node and link it to the Pine Vector Store tool. In Retrieval node we add the Pincone API details and also provide the Pine Code index from list as company-files.
7. Next we create and connect Embeddings OpenAI node with Pinecone vector store (Retrieval) node.
8. We also connect one additional OpenAI model with Vector Store tool node as a default requirement.
9. The output of AI agent is connected to the Respond to Webhook node. This will give back the reply to the chatbot.
10. This way we complete the second workflow.

**Finally:**
- We execute both the workflows then we ask specific questions in the chatbot window created on our local (on the same system we created the  n8n workflow) the webhook node in the second workflow gets triggered.
- The n8n workflow start responding to questions as desired.
- In case you want the chatbot to have a public url then you can post the chatbot code on intellify.com












