## What is LlamaParse ?

LlamaParse is a proprietary parsing service that is incredibly good at parsing PDFs with complex tables into a well-structured markdown format.

This service is available in a ****public preview mode**:** available to everyone, but with a usage limit (1k pages per day) with 7,000 free pages per week. Then $0.003 per page ($3 per 1,000 pages).It operates as a standalone service that can also be plugged into the managed ingestion and retrieval API

```

from llama_parse import LlamaParse  
  
parser = LlamaParse(  
api_key="llx-...", # can also be set in your env as LLAMA_CLOUD_API_KEY  
result_type="markdown", # "markdown" and "text" are available  
verbose=True  
)

```

Currently LlamaParse primarily support PDFs with tables, but they are also building out better support for figures, and and an expanded set of the most popular document types: .docx, .pptx, .html as a part of next enhancements.

**rich table support**

Since we first released LlamaParse it has featured [industry-leading table extraction]([https://github.com/run-llama/llama_parse/blob/main/examples/demo_advanced.ipynb](https://github.com/run-llama/llama_parse/blob/main/examples/demo_advanced.ipynb) ) capabilities. Under the hood, this has been using LLM intelligence since the start. It seamlessly integrates with the advanced indexing/retrieval capabilities that the open-source framework offers, enabling users to build state-of-the-art document RAG. Now with JSON mode (see below) and parsing instructions, you can take this even further.

  

  
![enter image description here](https://www.llamaindex.ai/_next/image?url=https://cdn.sanity.io/images/7m9jw85w/production/b5404df39ca9c68da96a69a72cb877ec6c22ab1a-6426x1688.png?fit=max&auto=format&w=3840&q=75)
source: https://www.llamaindex.ai/blog/launching-the-first-genai-native-document-parsing-platform


  

**Example 2: parsing comic books**

Parsing translated manga presents a particular challenge for a parser since a regular parser interprets the panels as cells in a table, and the reading order is right-to-left even though the book is in English, as shown in this extract from "The manga guide to calculus", by Hiroyuki Kojima:

![enter image description here](https://www.llamaindex.ai/_next/image?url=https://cdn.sanity.io/images/7m9jw85w/production/cf511e4ac12c8b0d48cbed2e9914298e030c6479-1772x1680.png?fit=max&auto=format&w=1080&q=75)

source: https://www.llamaindex.ai/blog/launching-the-first-genai-native-document-parsing-platform

Using LlamaParse, you can give the parser plain, English-language instructions on what to do:

```

The provided document is a manga comic book.  
Most pages do NOT have title. It does not contain tables.  
Try to reconstruct the dialogue happening in a cohesive way.

```

(You can see the full code in our [demonstration notebook]([https://colab.research.google.com/drive/1dO2cwDCXjj9pS9yQDZ2vjg-0b5sRXQYo](https://colab.research.google.com/drive/1dO2cwDCXjj9pS9yQDZ2vjg-0b5sRXQYo) ), including what it looks like to parse this without the instructions)

The result is a perfect parse!

```

# The Asagake Times  
  
Sanda-Cho Distributor  
  
A newspaper distributor?  
  
Do I have the wrong map?

```

**Example 3: mathematical equations**

Another challenging format for parsing is complex mathematical equations (by coincidence, the manga we picked as an example is all about how to do mathematics):

![enter image description here](https://www.llamaindex.ai/_next/image?url=https://cdn.sanity.io/images/7m9jw85w/production/ac99476c1f6729fa05b9221caea8199a3b96fce6-1676x702.png?fit=max&auto=format&w=1080&q=75)

To parse this, we take the same instructions as before and add one sentence: `Output any math equation in LATEX markdown (between $$)` . The result of parsing is clear LaTeX instructions, which render the equations perfectly:
![enter image description here](https://www.llamaindex.ai/_next/image?url=https://cdn.sanity.io/images/7m9jw85w/production/f07d6913452db6e24eefff6b2779d94b8b0692b8-1988x614.png?fit=max&auto=format&w=1080&q=75)

for local deployment using docker: https://blog.gopenai.com/running-pdf-parsers-in-docker-containers-5e7a7ed829c8

**Code Implementation**

The code is implemented in Google Colab (cpu)

Install required dependencies

```

%%writefile requirements.txt  
langchain  
langchain-community  
llama-parse  
fastembed  
chromadb  
python-dotenv  
langchain-groq  
chainlit  
fastembed  
unstructured[md]  
  
!pip install -r requirements.txt

```

Set up the environment variables

```

from google.colab import userdata  
  
llamaparse_api_key = userdata.get('LLAMA_CLOUD_API_KEY')  
groq_api_key = userdata.get("GROQ_API_KEY")  
  
Import required dependencies  
  
##### LLAMAPARSE #####  
from llama_parse import LlamaParsefrom langchain.text_splitter import RecursiveCharacterTextSplitterfrom langchain_community.embeddings.fastembed import FastEmbedEmbeddingsfrom langchain_community.vectorstores import Chromafrom langchain_community.document_loaders import DirectoryLoaderfrom langchain_community.document_loaders import UnstructuredMarkdownLoaderfrom langchain.prompts import PromptTemplatefrom langchain.chains import RetrievalQA#  
from groq import Groqfrom langchain_groq import ChatGroq#  
import joblibimport osimport nest_asyncio # noqa: E402  
nest_asyncio.apply()

```

LlamaParse Parameters

```

* api_key: str = Field(  
default="",  
  
description="The API key for the LlamaParse API.",  
  
)  
  
* base_url: str = Field(  
default=DEFAULT_BASE_URL,  
description="The base URL of the Llama Parsing API.",  
)  
* result_type: ResultType = Field(  
default=ResultType.TXT, description="The result type for the parser."  
)  
* num_workers: int = Field(  
default=4,  
gt=0,  
lt=10,  
description="The number of workers to use sending API requests for parsing."  
)  
* check_interval: int = Field(  
default=1,  
description="The interval in seconds to check if the parsing is done.",  
)  
* max_timeout: int = Field(  
default=2000,  
description="The maximum timeout in seconds to wait for the parsing to finish.",  
)  
* verbose: bool = Field(  
default=True, description="Whether to print the progress of the parsing."  
)  
* language: Language = Field(  
default=Language.ENGLISH, description="The language of the text to parse."  
)  
* parsing_instruction: Optional[str] = Field(  
default="",  
description="The parsing instruction for the parser."  
)

```

Helper function to load and parse the input data

```

!mkdir data  
#  
def  load_or_parse_data():  
data_file = "./data/parsed_data.pkl"  
  
if os.path.exists(data_file):# Load the parsed data from the file  
parsed_data = joblib.load(data_file)  
else:  
# Perform the parsing step and store the result in llama_parse_documentsparsingInstructionUber10k = """The provided document is a quarterly report filed by Uber Technologies,  
Inc. with the Securities and Exchange Commission (SEC).  
This form provides detailed financial information about the company's performance for a specific quarter.  
It includes unaudited financial statements, management discussion and analysis, and other relevant disclosures required by the SEC.  
It contains many tables.  
Try to be precise while answering the questions"""  
parser = LlamaParse(api_key=llamaparse_api_key,  
result_type="markdown",  
parsing_instruction=parsingInstructionUber10k,  
max_timeout=5000,)  
llama_parse_documents = parser.load_data("./data/uber_10q_march_2022 (1).pdf")  
  
  
# Save the parsed data to a file  
print("Saving the parse results in .pkl format ..........")  
joblib.dump(llama_parse_documents, data_file)  
  
# Set the parsed data to the variable  
parsed_data = llama_parse_documents  
  
return parsed_data

```

Helper function to load chunks into vectorstore.

```

# Create vector database

def  create_vector_database():  
"""  
Creates a vector database using document loaders and embeddings.  
  
This function loads urls,  
splits the loaded documents into chunks, transforms them into embeddings using OllamaEmbeddings,  
and finally persists the embeddings into a Chroma vector database.  
  
"""  
# Call the function to either load or parse the data  
llama_parse_documents = load_or_parse_data()  
print(llama_parse_documents[0].text[:300])  
  
with open('data/output.md', 'a') as f: # Open the file in append mode ('a')  
for doc in llama_parse_documents:  
f.write(doc.text + '\n')  
  
markdown_path = "/content/data/output.md"  
loader = UnstructuredMarkdownLoader(markdown_path)  
  
#loader = DirectoryLoader('data/', glob="**/*.md", show_progress=True)  
documents = loader.load()  
# Split loaded documents into chunks  
text_splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=100)  
docs = text_splitter.split_documents(documents)  
  
#len(docs)  
print(f"length of documents loaded: {len(documents)}")  
print(f"total number of document chunks generated :{len(docs)}")  
#docs[0]  
  
# Initialize Embeddings  
embed_model = FastEmbedEmbeddings(model_name="BAAI/bge-base-en-v1.5")  
  
# Create and persist a Chroma vector database from the chunked documents  
vs = Chroma.from_documents(  
documents=docs,  
embedding=embed_model,  
persist_directory="chroma_db_llamaparse1", # Local mode with in-memory storage only  
collection_name="rag"  
)  
  
#query it  
#query = "what is the agend of Financial Statements for 2022 ?"  
#found_doc = qdrant.similarity_search(query, k=3)  
#print(found_doc[0][:100])  
#print(qdrant.get())  
  
print('Vector DB created successfully !')  
return vs,embed_model

```

Process the data and create Vector Store

```

vs,embed_model = create_vector_database()

```

Instantiate LLM

```

  
chat_model = ChatGroq(temperature=0,  
model_name="mixtral-8x7b-32768",  
api_key=userdata.get("GROQ_API_KEY"),)

```

The above code does the following:

- Creates a new ChatGroq object named chat_model

- Sets the temperature parameter to 0, indicating that the responses should be more predictable

- Sets the model_name parameter to “mixtral-8x7b-32768“, specifying the language model to use

Instantiate Vectorstore

```

vectorstore = Chroma(embedding_function=embed_model,  
persist_directory="chroma_db_llamaparse1",  
collection_name="rag")  
#  
retriever=vectorstore.as_retriever(search_kwargs={'k': 3})

```

Create a Custom Prompt Template

```

custom_prompt_template = """Use the following pieces of information to answer the user's question.  
If you don't know the answer, just say that you don't know, don't try to make up an answer.  
  
Context: {context}  
Question: {question}  
  
Only return the helpful answer below and nothing else.  
Helpful answer:  
"""

```

Helper Function to format the prompt

```

def  set_custom_prompt():  
"""  
Prompt template for QA retrieval for each vectorstore  
"""  
prompt = PromptTemplate(template=custom_prompt_template,  
input_variables=['context', 'question'])  
return prompt#  
prompt = set_custom_prompt()  
prompt

########################### RESPONSE ###########################

PromptTemplate(input_variables=['context', 'question'], template="Use the following pieces of information to answer the user's question.\nIf you don't know the answer, just say that you don't know, don't try to make up an answer.\n\nContext: {context}\nQuestion: {question}\n\nOnly return the helpful answer below and nothing else.\nHelpful answer:\n")

```

Instantiate the Retrieval Question Answering Chain

```

qa = RetrievalQA.from_chain_type(llm=chat_model,  
chain_type="stuff",  
retriever=retriever,  
return_source_documents=True,  
chain_type_kwargs={"prompt": prompt})

```

Invoke the Retriveal QA Chain

```

response = qa.invoke({"query": "what is the Balance of UBER TECHNOLOGIES, INC.as of December 31, 2021?"})

```
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*0uUDSBEDL2u8E8LWhuQ1lA.png)
Response Synthesized

```

response['result']

########################### RESPONSE ###########################

Based on the provided balance sheet of Uber Technologies, Inc. as of December 31, 2021, the total assets are $38,774 million, total liabilities are $23,425 million, and total equity is $9,613 million.

```

Question 2
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*8_rsNmf8M41X2ge5H6PLGg.png)
```

response = qa.invoke({"query": "What is the Cash flows from operating activities associated with bad expense specified in the document ?"})  
response['result']

######################## RESPONSE ###############################

The Cash flows from operating activities associated with bad debt expense is 23 for the year 2021 and 18 for the year 2022.

```

Question 3

```

response = qa.invoke({"query": "what is Loss (income) from equity method investments, net ?"})  
response["result"]

############################### RESPONSE #############################

The loss from equity method investments, net, is calculated as the sum of the impairment of equity method investment and the revaluation of MLU B.V. call option, which amounted to $182 million and $181 million, respectively. This results in a total loss from equity method investments, net, of $363 million. This loss is included in the net loss attributable to Uber Technologies, Inc. of $5.9 billion.

```

Question 4
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*DSKip_YdgdpTquyUgGRUkQ.png)
```

response = qa.invoke({"query": "What is the Total cash and cash equivalents, and restricted cash and cash equivalents for reconciliation ?"})  
response['result']

######################## RESPONSE ####################################

The total cash and cash equivalents, and restricted cash and cash equivalents for reconciliation is $6,607 million. This amount is obtained by adding the cash and cash equivalents of $4,836 million and the restricted cash and cash equivalents - current of $247 million, and the restricted cash and cash equivalents - non-current of $1,524 million.

```

Question 5
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*R07nG_JbXdEN-XB9nqsPuQ.png)
```

response = qa.invoke({"query":"Based on the CONDENSED CONSOLIDATED STATEMENTS OF REDEEMABLE NON-CONTROLLING INTERESTS AND EQUITY what is the Balance as of March 31, 2021?"})  
print(response['result'])

############# RESPONSE ##################

The balance as of March 31, 2021 was $473 for Redeemable Non-Controlling Interests, 1,867,369 shares for Common Stock, $— for Additional Paid-In Capital, $36,182 for Other Comprehensive Income (Loss), $654 for Non-Controlling Interests, and $654 for Total Equity.

```

Question 6
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Evb-ZK7JB6IJaGMlyCAesQ.png)
```

response = qa.invoke({"query":"Based on the condensed consolidated statements of comprehensive Income(loss) what is the Comprehensive income (loss) attributable to Uber Technologies, Inc.for the three months ended March 31, 2022"})  
response['result']

######################### RESPONSE####################################

The Comprehensive income (loss) attributable to Uber Technologies, Inc. for the three months ended March 31, 2022 was $(5,911) million. This information can be found on the Uber Technologies, Inc. - Condensed Consolidated Statements of Comprehensive Income (Loss) provided in the quarterly report.

```

Question 7

```

response = qa.invoke({"query":"Based on the condensed consolidated statements of comprehensive Income(loss) what is the Comprehensive income (loss) attributable to Uber Technologies?"})  
response['result']

##################### RESPONSE #################################

The Comprehensive income (loss) attributable to Uber Technologies, Inc. for the three months ended March 31, 2021 is $1,081 million, and for the three months ended March 31, 2022 is -$5,911 million.

```

Question 8

```

response = qa.invoke({"query":"Based on the condensed consolidated statements of comprehensive Income(loss) what is the Net loss including non-controlling interests"})  
response['result']

################ RESPONSE #######################################

The Net loss including non-controlling interests is $(122) million for the three months ended March 31, 2021 and $(5,918) million for the three months ended March 31, 2022.

```

Question 9
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*JZJmFME2uiNwRJtKgcVudA.png)
```

response = qa.invoke({"query":"what is the Net cash used in operating activities for Mrach 31,2021? "})  
response['result']

############## RESPONSE ###############################

Net cash used in operating activities for March 31, 2021 was $611 million.

```

Question 10
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ScTvxQA3nap8AUs9q1heAA.png)
```

query = "Based on the CONDENSED CONSOLIDATED STATEMENTS OF CASH FLOWS What is the value of Purchases of property and equipment ?"  
response = qa.invoke({"query":query})  
response['result']

####################### RESPONSE #####################################

The value of purchases of property and equipment for the three months ended March 31, 2021 and 2022 can be found in the 'Cash flows from investing activities' section of the condensed consolidated statements of cash flows.

For the three months ended March 31, 2021: $71 million

For the three months ended March 31, 2022: $62 million

```

Question 11

```

query = "Based on the CONDENSED CONSOLIDATED STATEMENTS OF CASH FLOWS what is the Purchases of property and equipment for the year 2022?"  
response = qa.invoke({"query":query})  
response['result']

########### RESPONSE #####################################

The purchases of property and equipment for the year 2022 based on the CONDENSED CONSOLIDATED STATEMENTS OF CASH FLOWS is -62.

```

From the above implementation we can assume that LLamaParse is comparatively good at parsing complex pdf documents. Although we have to experiment more with different tabular structures. Here ia comparison of LlamaParse with PyPDF
![enter image description here](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ZFKIWGuuVK2DUbZoybBpmg.png)
ref:

[https://www.llamaindex.ai/blog/launching-the-first-genai-native-document-parsing-platform](https://www.llamaindex.ai/blog/launching-the-first-genai-native-document-parsing-platform)

[https://medium.com/the-ai-forum/rag-on-complex-pdf-using-llamaparse-langchain-and-groq-5b132bd1f9f3](https://medium.com/the-ai-forum/rag-on-complex-pdf-using-llamaparse-langchain-and-groq-5b132bd1f9f3)

[https://wow.groq.com/retrieval-augmented-generation-with-groq-api/](https://wow.groq.com/retrieval-augmented-generation-with-groq-api/)

[https://wow.groq.com/retrieval-augmented-generation-with-groq-api/](https://wow.groq.com/retrieval-augmented-generation-with-groq-api/)
