# ppc64le

Notes on installing PyTorch and GPT-NeoX on ppc64le architecture

The following notes were provided by Eric Hallahan as a starting point (https://github.com/EleutherAI/gpt-neox/pull/456):  

## GPT-NeoX Requirements
pybind11>=2.6.0  
six  
regex  
numpy>=1.20.0  
git+git://github.com/EleutherAI/DeeperSpeed.git@eb7f5cff36678625d23db8a8fe78b4a93e5d2c75#egg=deepspeed  
* PyTorch --> Must install from source (hard)  
* Triton --> Must install from source (easy)  
* 
transformers>=4.5.0  
tokenizers>=0.10.0  
lm_dataformat>=0.0.19  
ftfy>=6.0.0  
lm_eval  
* Eigen
* DyNet --> Must install from source (hard)  
* PyArrow
