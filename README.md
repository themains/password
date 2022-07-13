## Pass-Fail: How Quickly Can Your Password Be Cracked?

Despite most modern browsers providing password generator and management tools, many of the users do not pick long, randomly generated passwords. With the aim of building a tool that offers advice on a) strength of passwords (that goes beyond using password length), and b) how to make stronger passwords, we built a ML model. Using ~816M leaked passwords from [various databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases), we built a [character level password generator](https://github.com/themains/password/blob/main/notebooks/03_tf_seq_to_seq_training_881M.ipynb) using an encoder-decoder model. (We also built a [separate model using a database of 18M passwords](https://github.com/themains/password/blob/main/notebooks/03_tf_seq_to_seq_tranining_18M.ipynb).) We then validate the model against the [HaveIBeenPwned (HIBP)](https://haveibeenpwned.com/) database. We find that of the roughly 10,000 model generated 5-character-long passwords, 58% match a password in the HIBP database compared to .1% of the 10,000 randomly generated passwords. As a more stringest test, we estimated the HIBP match rate for model generated passwords that are not in the training corpus. For passwords starting with 10 of the most common characters, the match rate is about 10%. 

We use model to understand one of the correlates of a strong password---the starting character. In our data, the starting characters of passwords have a sharp skew, with about 30 characters covering about 83% of the passwords. And, understandably, passwords starting with more common characters can be guessed by model more easily (the correlation is 84% for 5 char. passwords on our data). This suggests that there is an opportunity to create passwords using less commonly used starting characters (see the distribution of first characters in our notebooks linked to above). 

Our model can also be used to estimate strength of a password. Admittedly, the job is computationally heavy. And approximate inference based on, e.g., maximum percentage matched in the first 100 tries, may be useful. For illustration, [our generator](https://github.com/themains/password/blob/main/notebooks/03_tf_seq_to_seq_tranining_18M.ipynb) can recover the password 'Password1' in ~ 1000 tries when the search space for 9 character password with 95 tokens is in the quadrillions.

**Note:** We are aware that the model can be used by hackers to crack passwords but we believe the computational cost of running the model for such inference is likely too high when compared to brute force or dictionary based methods or methods that exploit common patterns in passwords.

### Table of Contents

* [Data](#data)
* [Model](#model)
* [Metrics](#metrics)
* [Practical Validation](#validation)
* [Password Embeddings](#embeddings)

### Data

We created the master password database by merging data from the following [leaked password databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases). (Note: We are aware that bible is merely a collection of all the words in bible---or so it seems---and have chosen to keep it as a dictionary.) You can find more details in this [notebook](notebooks/01_data.ipynb).

```
passwords_db/000webhost.txt      passwords_db/izmy.txt
passwords_db/adobe100.txt        passwords_db/Lizard-Squad.txt
passwords_db/Ashley-Madison.txt  passwords_db/md5decryptor-uk.txt
passwords_db/bible.txt           passwords_db/muslimMatch.txt
passwords_db/carders.cc.txt      passwords_db/myspace.txt
passwords_db/comb.txt            passwords_db/NordVPN.txt
passwords_db/elitehacker.txt     passwords_db/phpbb-cleaned-up.txt
passwords_db/faithwriters.txt    passwords_db/porn-unknown.txt
passwords_db/hak5.txt            passwords_db/rockyou.txt
passwords_db/honeynet2.txt       passwords_db/singles.org.txt
passwords_db/honeynet.txt        passwords_db/tuscl.txt
passwords_db/hotmail.txt         passwords_db/youporn2012.txt
```

After merging the above, the total count came to 881623002 passwords (881 million passwords).

We only consider passwords that are between 3 and 50 characters long. Only a tiny fraction of passwords are over 50 character long. And the 3 character lower limit eliminates many of the short words from `bible`, etc.

### Model

The vocabulary size is 95. 

#### Data Setup

Sequence to sequence is used to train Tensorflow GRU model.<br>
For example: if the password is `12STEVEN` <br>
then the input to the model is `12STEVE` <br>
And the prediction label is `2STEVEN` <br>
Data is split as follows: 95% for training, 2.5% for validation, 2.5% for test.<br>
You can find more details on how the data is split in this [notebook](notebooks/01_data.ipynb).<br>
Since loading all train data to memory requires more than 200+ GB RAM, tf records are created, you can refer to this [notebook](notebooks/02_tf_records.ipynb) on how this is done.

Below is the GRU model that we used
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

### Metrics

We used the [bleu](https://www.nltk.org/_modules/nltk/translate/bleu_score.html) score to measure the model performance. <br>
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

The bleu score on the test dataset is `0.02` <br>
After training the model for 10 epochs the bleu score on the test dataset increased to `0.96`

### Validation

How quickly can we guess the password `Password1` (of length 9) when we feed `P` to the model? The model takes 1026 attempts to find the password (compared to 315 quadrillion attempts on average using random password generation with 95 tokens) 

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
[('k', 0.030112092511078912),
 ('t', 0.03019875298476047),
 ('l', 0.03200024618607084),
 ('d', 0.034974911596357544),
 ('c', 0.03615092217770743),
 ('b', 0.036440602438896755),
 ('m', 0.04673549531688035),
 ('a', 0.04689793719340077),
 ('1', 0.04736622677456893),
 ('s', 0.04979685375899136)]
```
As we can see, passwords most frequently start with `m`, followed by `s`, `0`, and so on. 

In the table below, <br>
`dist` - start character distribution across all passwords<br>
`rand_pred_prob` - Percent of passwords found in [https://haveibeenpwned.com/](https://haveibeenpwned.com/)  when passwords are generated randomly. <br>
`model_pred_prob` - Percent of passwords that were found in [https://haveibeenpwned.com/](https://haveibeenpwned.com/)  when passwords were generated using trained model. The model predictablility rate depends on start char distribution. If a start char is common, the model is able to predict more leaked passwords.  


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

Please refer to the [full notebook](notebooks/03_tf_seq_to_seq_training_881M.ipynb) for the steps involved in creating the model. 

### Embeddings

* [password vectors trained with 18M records](embeddings/password_tf_vectors_18M.tsv)
* [password metadata trained with 18M records](embeddings/password_tf_metadata_18M.tsv)
* [password vectors trained with 881M records](embeddings/password_tf_vectors_881M.tsv)
* [password metadata trained with 881M records](embeddings/password_tf_metadata_881M.tsv)

You can use http://projector.tensorflow.org/ to visualize the embeddings.

### Authors

Rajashekar Chintalapati and Gaurav Sood
