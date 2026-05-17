# n8n
## Objective: n8n project undertaken for building a RAG pipeline to extract required information from Office documents stored in Google Drive as repository.

# Credentials Required for the setup:
- Google Drive Access Credentials, you also specify the folder that will be used as Office Folder
- Pine Cone Vector Store Credentials
- Model API whichever you want to prefer

# Setup
1. You add two Google Drive File Nodes, one for File Created and the other for File Updated. This means this node will trigger as soon as any new file is added or updated in our office folder repository.
2. By clicking these nodes you further configure them by adding Google Drive OAuth2 API credentials with poll time of 1 minute and specifying the target folder which in our case is named n8n Testing
3. The next node we add in the flow is Download File from Google Drive. By double clicking it we further configure it. We firstly add the Google Drive OAuth2 API credentials, Select Resource as File, select Download in operation field. Fill json.id in file by ID (generally done by dragging from input window and dropping it in the required field.). Further, in options  we select the file name json.name in similar way of dragging and dropping from input window.
4. After connecting the Google Drive Create and Update node with Download Google Drive node we add next node as Pinecone Vector store.
5. We double click PineCone Vector store for its configuration. We firstly go to pinecone vector store site "pinecone.io" We click get started and click on Create index under Database. Firstly we name the index say "company-files"> select custom settings >  Under configurations select Dense and put dimension as 1536 with Cosine selection > Click create index.
6. You put this file name "company-files" in Pinecone Vector Store node under PineCone Index from list field.
Then we add OpenAI Embeddings node that connects PineCone Vector store. We double click to configure this node. We add the required node credentials, select the desired model and also add dimensions as 1536.
We connect default data loader node to Pinecone vector store and configure dataloader
