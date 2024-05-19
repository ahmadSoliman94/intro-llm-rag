
- For new server we may need to install Nvidia drivers if not installed. 
```
  
sudo add-apt-repository ppa:graphics-drivers/ppa --yes  
sudo apt update  
  
sudo apt install nvidia-driver-470 # or nvidia-driver-495
```

- The required python library are listed in the file requirements.txt
o We need to install Python 3.10 or above
o We need to run the command `python3 -m pip install -r requirements.txt`
o Below is the content of the requirements.txt
```
aiohttp==3.9.3
aiosignal==1.3.1
annotated-types==0.6.0
anyio==4.3.0
appnope==0.1.4
asttokens==2.4.1
asyncio==3.4.3
attrs==23.2.0
certifi==2024.2.2
cffi==1.16.0
charset-normalizer==3.3.2
comm==0.2.1
dataclasses-json==0.6.4
debugpy==1.8.1
decorator==5.1.1
deepgram-sdk==3.1.4
distro==1.9.0
executing==2.0.1
frozenlist==1.4.1
gevent==24.2.1
greenlet==3.0.3
groq==0.4.1
h11==0.14.0
httpcore==1.0.4
httpx==0.27.0
idna==3.6
ipykernel==6.29.2
ipython==8.22.1
jedi==0.19.1
jsonpatch==1.33
jsonpointer==2.4
jupyter_client==8.6.0
jupyter_core==5.7.1
langchain==0.1.9
langchain-community==0.0.24
langchain-core==0.1.26
langchain-groq==0.0.1
langchain-openai==0.0.7
langsmith==0.1.6
marshmallow==3.20.2
matplotlib-inline==0.1.6
multidict==6.0.5
mypy-extensions==1.0.0
nest-asyncio==1.6.0
numpy==1.26.4
openai==1.12.0
orjson==3.9.15
packaging==23.2
parso==0.8.3
pexpect==4.9.0
platformdirs==4.2.0
prompt-toolkit==3.0.43
psutil==5.9.8
ptyprocess==0.7.0
pure-eval==0.2.2
PyAudio==0.2.14
pycparser==2.21
pydantic==2.6.2
pydantic_core==2.16.3
pygame==2.5.2
Pygments==2.17.2
python-dateutil==2.8.2
python-dotenv==1.0.1
PyYAML==6.0.1
pyzmq==25.1.2
regex==2023.12.25
requests==2.31.0
six==1.16.0
sniffio==1.3.0
sounddevice==0.4.6
SQLAlchemy==2.0.27
stack-data==0.6.3
tenacity==8.2.3
tiktoken==0.6.0
tornado==6.4
tqdm==4.66.2
traitlets==5.14.1
typing-inspect==0.9.0
typing_extensions==4.9.0
urllib3==2.2.1
verboselogs==1.7
wcwidth==0.2.13
websocket==0.2.1
websocket-client==1.7.0
websockets==12.0
yarl==1.9.4
zope.event==5.0
zope.interface==6.2
```
