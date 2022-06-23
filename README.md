## Password Generator

We create a character level text generator using an encoder-decoder model with the passwords that are leaked
from these [databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases)

### Data
Below are the dumps used to create a master password database. You can find more details in this [notebook](notebooks/data.ipynb)
```
rockyou.txt
000webhost.txt
Ashley-Madison.txt
Lizard-Squad.txt
NordVPN.txt
adobe100.txt
bible.txt
carders.cc.txt
elitehacker.txt
faithwriters.txt
hak5.txt
honeynet.txt
honeynet2.txt
hotmail.txt
izmy.txt
md5decryptor-uk.txt
muslimMatch.txt
myspace.txt
phpbb-cleaned-up.txt
porn-unknown.txt
singles.org.txt
tuscl.txt
youporn2012.txt
```

After merging all above, the total count came to 18308617 passwords (18 million passwords).

Note: We only considered passwords that have length between 3 and 50. 

### Scripts
#### Vocabulary
The vocabulary size is 96. Below are some stats on data
```
Total number of passwords 18308617
Passwords vocab size 95
Max passwords length 50
```
#### Data setup
Sequence to sequence is used to train Tensorflow GRU model.<br>
For example: if password is `12STEVEN` <br>
then Input to model is `12STEVE` <br>
Output from the model is `2STEVEN` <br>
Data is split as 90% for training, 5% for validation, 5% for test.<br>

#### Model
Below is the GRU model used
```
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
embedding_1 (Embedding)      multiple                  9216      
_________________________________________________________________
gru_1 (GRU)                  multiple                  271872    
_________________________________________________________________
dense_1 (Dense)              multiple                  24672     
=================================================================
Total params: 305,760
Trainable params: 305,760
Non-trainable params: 0
_________________________________________________________________
```
Model was trained on char sequence to char sequence.

#### Metrics
[nltp bleu](https://www.nltk.org/_modules/nltk/translate/bleu_score.html) score is used to measure the model performance. <br>
Below are some examples - <br>
**if all are equal**
```python
ref = [['a', 'b', 'c'], ['d','e'], ['f'], [' ']]
hyp = [['a', 'b', 'c'], ['d','e'], ['f'], [' ']]
corpus_bleu(ref, hyp, weights=[0.25])
```
output - `1.0` <br>
**if some are equal**
```python
ref = [['a', 'b', 'c'], ['d','e'], ['f'], [' ']]
hyp = [['a', 'b', 'f'], ['d','e'], ['f'], [' ']]
corpus_bleu(ref, hyp, weights=[0.25])
```
output - `0.9621954581957615` <br>
**if non are equal**
```python
ref = [['a', 'b', 'c'], ['d','e'], ['f'], [' ']]
hyp = [['p', 'q', 'r'], ['s','t'], ['u'], ['w']]
corpus_bleu(ref, hyp, weights=[0.25])
```
output - `0`<br>

The bleu score on complete test dataset came to `0.478` <br>
After training the model for 4 epochs the bleu score on test dataset increased to `0.746`

#### Learnings
Given a password `Password1` (of length 9), when we fed `P` to the model, model took 1026 attempts to find the password. 
```
1 - PANDOWCOM
2 - Politim12
3 - P1Z5Z36Q1
4 - Phils#312
5 - Peneme160
6 - PaLiVasda
7 - Panisabas
8 - Pedrostin
9 - PRESO514#
10 - PersiaM80
11 - PALOPER55
12 - PAGAUNECI
13 - Pom#2lu80
14 - Papa62873
15 - PONDBOOZZ
16 - PecKhouse
17 - Peec65alm
18 - POODSONEK
19 - PORNILLOS
20 - Pinoyano7
21 - Pamk30000
22 - Padololin
23 - PpA!2002C
24 - Pgc716558
25 - Pordot001
...
1024 - PENTONC13
1025 - PURPLEJAV
1026 - Password1
Model took 1026 attempts to find password - Password1
```

Passwords starting with each char distribution is like below. Showing top 10 char distribution in ascending order. 
```
[('d', 0.03461719691880605),
 ('j', 0.03637090666105474),
 ('l', 0.037107718185376865),
 ('b', 0.04236693574397236),
 ('c', 0.04299341670646122),
 ('1', 0.04496276261609492),
 ('a', 0.04987919076574708),
 ('0', 0.05549201231310918),
 ('s', 0.05710338470677496),
 ('m', 0.057332020217583886)]
```
With above we can see that most number of passwords start with `m`, `s`, `0` so on. 

In below table, <br>
`dist` - start character distribution across all passwords<br>
`rand_pred_prob` - Percent of passwords found in [https://haveibeenpwned.com/](https://haveibeenpwned.com/)  when passwords are generated randomly. <br>
`model_pred_prob` - Percent of passwords that were found in [https://haveibeenpwned.com/](https://haveibeenpwned.com/)  when passwords were generated using trained model. The model predictablility rate depends on start char distribution, if a start char has more distribution then model is able to predict more leaked passwords.   


|	|char|	dist	|rand_pred_prob	|model_pred_prob|
|---|--- |---       |---    |---    |
|94	|m	|0.057332	|0.00	|0.77 |
|93	|s	|0.057103	|0.00	|0.76 |
|92	|0	|0.055492	|0.00	|0.90 |
|91	|a	|0.049879	|0.00	|0.77 |
|90	|1	|0.044963	|0.01	|0.71 |
|89	|c	|0.042993	|0.02	|0.75 |
|88	|b	|0.042367	|0.01	|0.79 |
|87	|l	|0.037108	|0.00	|0.77 |
|86	|j	|0.036371	|0.01	|0.71 |
|85	|d	|0.034617	|0.01	|0.72 |
|84	|t	|0.034498	|0.00	|0.69 |
|83	|p	|0.033237	|0.00	|0.77 |
|82	|k	|0.031070	|0.00	|0.72 |
|81	|2	|0.028482	|0.00	|0.75 |
|80	|r	|0.028280	|0.00	|0.71 |
|79	|n	|0.023127	|0.00	|0.66 |
|78	|g	|0.022110	|0.00	|0.69 |
|77	|h	|0.021496	|0.00	|0.71 |
|76	|f	|0.020230	|0.00	|0.67 |
|75	|e	|0.019975	|0.00	|0.64 |
|74	|i	|0.018939	|0.00	|0.59 |
|73	|3	|0.015904	|0.00	|0.77 |
|72	|4	|0.015702	|0.00	|0.80 |
|71	|9	|0.015432	|0.00	|0.82 |
|70	|5	|0.015039	|0.00	|0.79 |
|69	|w	|0.013814	|0.00	|0.52 |
|68	|8	|0.012629	|0.01	|0.78 |
|67	|7	|0.012027	|0.00	|0.72 |
|66	|6	|0.011991	|0.00	|0.70 |
|65	|v	|0.010075	|0.00	|0.63 |
|...	|...|	...	...	|...      |
|29	|#	|0.000473	|0.00	|0.05 |
|28	|$	|0.000462	|0.00	|0.09 |
|27	|(	|0.000430	|0.00	|0.01 |
|26	|.	|0.000373	|0.00	|0.07 |
|25	|-	|0.000236	|0.00	|0.04 |
|24	|_	|0.000214	|0.00	|0.08 |
|23	|~	|0.000149	|0.00	|0.02 |
|22	|[	|0.000148	|0.00	|0.00 |
|21	|<	|0.000129	|0.00	|0.01 |
|20	|+	|0.000112	|0.00	|0.01 |
|19	|/	|0.000107	|0.00	|0.02 |
|18	|,	|0.000104	|0.00	|0.00 |
|17	|;	|0.000073	|0.00	|0.00 |
|16	|=	|0.000065	|0.00	|0.01 |
|15	|"	|0.000065	|0.00	|0.02 |
|14	|?	|0.000064	|0.00	|0.02 |
|13	|`	|0.000055	|0.00	|0.03 |
|12	|&	|0.000052	|0.00	|0.03 |
|11	|%	|0.000051	|0.00	|0.01 |
|10	|	|0.000039	|0.00	|0.05 |
|9	|:	|0.000039	|0.00	|0.04 |
|8	|^	|0.000036	|0.00	|0.01 |
|7	|\	|0.000031	|0.00	|0.02 |
|6	|{	|0.000028	|0.00	|0.00 |
|5	|'	|0.000025	|0.00	|0.01 |
|4	|)	|0.000024	|0.00	|0.01 |
|3	|]	|0.000021	|0.00	|0.00 |
|2	|>	|0.000012	|0.00	|0.01 |
|1	|pipe	|0.000009	|0.00	|0.00 |
|0	|}	|0.000003	|0.00	|0.00 |

Please refer [full notebook](notebooks/train_tf_seq_to_seq.ipynb) for complete steps involved in creating this model. 

#### Password Embeddings
Please refer below files for the password embeddings. <br>
- [password vectors](embeddings/password_tf_vectors.tsv)
- [password metadata](embeddings/password_tf_metadata.tsv)

You can use http://projector.tensorflow.org/ to visualize the embeddings.

### Authors

Rajashekar Chintalapati and Gaurav Sood 