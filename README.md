# Kaggle Jigsaw Multilingual Toxic Comment Classification 42nd Place Solution  
**Competition Description**  
Jigsaw is a unit within Google whose one of the main areas of focus is machine learning models that can identify toxicity in online conversations, where toxicity is defined as anything rude, disrespectful or otherwise likely to make someone leave a discussion. In this competition we were tasked with identifying toxic comments from given test set containing comments from 6 different languages while the training set was in english only. The goal was to understand how well a multilingual model trained with only english dataset can be generalized to other languages. You can get more details about this competition by clicking [here](https://www.kaggle.com/c/jigsaw-multilingual-toxic-comment-classification).  

**Public Leaderboard Scores**  
Best single model score: 0.9425  
Best score with translated data: 0.9443  
Best Ensemble (including public kernels): 0.9479  
Pesudo-labelling: 0.9486  
Best Submission: 0.9487  

**Private Leaderboard score:** 0.9473

**Data Input & Preprocessing**  
No text cleaning, augmenting or any other preprocessing techniques improved score. Since the data was highly skewed,the  training data was reduced by downsampling the negative examples to get the ratio of around 7:3 negative to positive examples instead of around 99:1. Furthermore, the test set comments were converted into english language with google and yandex translations using textblob and yandex.translate API.  

**Building Multilingual Models**  
There were two different approaches used to build the XLM-R (top performing multilingual pretrained model) models. In first approach, the model was trained and fine tuned on training and validation data using simple XLM-R model and model.fit function. This model was helpful with translated data and gave best score of 0.9433. The other approach was to use model which was first fine tuned with Masked Language Model (MLM) on jigsaw test dataset. This idea was taken from public kernel and though the scores from this model were similar to the former, the MLM model when blended with former model gave the best score of 0.9443.  

**Hyperparameter Tuning**  
The strategy used to fine tune both models involved the use of Adam optimizer, binary cross entropy loss function, and a low learning rate. The mixed precision training for TPUs ('bfloat16') was used to get faster training with higher batch size. Both models were trained using tensorflow since it worked better with TPUs. The first model was trained using default model.fit function while the second model involving MLM head used custom training loop to reduce training time and overfitting.  
Since re-training the model with same hyperparameters gave different scores using TPUs (difference ranging upto 0.005 with AUROC metric), I had to re-train the models with same hyperparameters several times to find the best model with that hyperparameters. This also made hyperparameter tuning more challenging and time consuming.

**CV Approach**  
Initially k-fold CV was used but later it turned out that the given validation set gave more consistent results with public leaderboard (PL) hence the score on validation set along with PL was used to assess model performance.  

**Ensembling Approach**  
Given the public test set, I went with an iterative blending approach, refining the test set predictions across submissions with a weighted average of the previous best submission (using either model strategy) and the current model’s predictions. I began with a simple average, and gradually increased the weight of the previous best submission. Finally, after merging my ensemble with public ensemble kernels I got 0.9479 on public leaderboard.

**Pseudo-labelling (PL)**  
I used test set as a training set to obtain soft labels for the training data and then reversed the scores that mismatched the soft labels with hard labels. Then I used these labels to obtain predictions for test set. This step gave me a score of 0.9483 from 0.9479. Later, I added test data to training data again to train the model and finally obtained predictions for test data which resulted in overall score of 0.9487 giving me 42nd position in private leaderboard.

**What I learned from this competition**  
Post-processing seems to be one of the winning strategies in a lot of top solutions. Therefore it will be one of my main focus for next NLP competition.  
Mono lingual models still outperform multilingual models and therefore should have been part of my solution.


