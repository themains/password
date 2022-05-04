
# Character level text(password) generation

  

This report is about charcter level text generation using Recurrent Neural Networks. We will use it to generate passwords. 

  

## Packages 

Python 3.7 and above 
 - Pytorch
 - pytorch_lightning
 - pandas

  

## Dataset

Data used to train this model is avaliable under [data](https://github.com/themains/password/tree/main/data "data") folder   


## Training

###  Data loading and preprocessing 
We preprocess all the passwords and remove non ascii charcters for this problem so totally 100 different type of charcters are possible along with a padding  token and end of sentence token total 102.

###  Model and parameters
As a recurrent network, we will use LSTM with 2 layers and 128 dimm embedding layer and 256 dimm hidden layer.

| | |
|--|--|
| Optimizer | Adam  |
| Learning rate| 5e-4|
| Batch| 1024|
| Epochs| 3|

Each epoch is taking about 5 hrs on rtx 3090 gpu with 1024 batch size.

### Metric 

BELU metric is calculated using the following way and the score is 0.368

 - Randomly sample 10000 passwords from whole dataset
-   In each of 10000 password take first character of each password and generate the a new passowrd
-   calculate bleu score on 10000 samples passwords as reference and 10000 generated passswords as     hypothesis


For the training inference and metric scripts please lookinto [passwordgen_rnn.ipynb](https://github.com/themains/password/blob/main/passwordgen_rnn.ipynb "passwordgen_rnn.ipynb")
