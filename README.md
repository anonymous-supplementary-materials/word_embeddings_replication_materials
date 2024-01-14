# word_embeddings_swedish_riksdag_motions
Code for replicating results on political discourse using word embeddings

## Script order and description:

**data_prep.py:**  
Extracts meta data and text from the motions files downloaded from Riksdagens öppna data (ROD) here: https://data.riksdagen.se/data/dokument/ 
For newer data (after 2010 sample) the html files can be used, as they have a consistent format. For older motions data is extracted from the txt files.  
**Input required:** data_folder containing csv files with the metadata (downloaded from ROD) and subfolders containing the motions for each session (for data before 2010 download the txt data from ROD, for 2010 onwards - download the html data from ROD)  
**Output:** Produces two pickle files containing S and M motions for the two studied time periods - 1988-2009 and 2010-2020    
**Example run:** python data_prep.py --data_folder data --output_folder output    

**pre_train_data_prep.py:**  
Extracts data for pre-training. Contains a list of the folders that were used for the data set. Only alphanumeric characters are extracted and appended to a txt file. No party information is needed, as the data is used as one continuous string of text during pretraining.   
**Input required:** data_folder containing subfolders with txt data files downloaded from ROD containing Interpellation, Proposition and Riksdagsskrivelse from the period 1998 - 2021. The list of folders used is defined in the script - some folders were dropped due to data quality issues.   
**Output:** a pre_train_text.txt file with all text examples that will be used for pre-training  
**Example run:** python pre_train_data_prep.py --data_folder data --output_folder output  

**pre_training.py:**  
Trains a gensim Word2Vec model by using data set produced by pre_train_data_prep.py and outputs it to a file.  
**Input required:** A txt file containing the pre-training data  
**Output:** a trained word2vec model consisting of three files - a standard format used in the gensim library  
**Example run:** python pre_training.py --data_file pre_train_text.txt --output_file pre_trained_model  

**fine_tune_riksdag.py:**  
Trains a model on S or M data separately from a starting point the model from pre_training.py. Training is repeated 10 times by resampling the fine-tuning data and learned word embedding vectors are written to file for each iteration.   
**Input required:** time_span - a choice between the three studied time periods, party - a choice between the two studied parties, pre_trained_model - path to the pre-trained model, epochs - number of epochs to train (one epoch means the training data is passed through the model for training once, usually multiple passes are needed for improved results), output_folde - directory for saving results; also uses the two pickle files created from the data_prep script.   
**Output:** fine-tuned models in a gensim format  
**Example run:** pyhton fine_tune_riksdag.py --time_span 1988-2020 --party s --pre_trained_model model --epochs 50 --output_folder output  

**fine_tune_nlpl.py:**  
Trains a model on S or M data separately from a starting point an externally pretrained model. The model is initialized by extracting available embeddings from the pre-trained model, adding randomly initialized embedding vectors for the new terms found in the fine-tuning data and using these vectors to initialize both the weights between input layer and hidden layer and the weights between hidden layer and output layer.
Training is repeated 10 times by resampling the fine-tuning data and learned word embedding vectors are written to file for each iteration.   
**Input required:** time_span - a choice between the three studied time periods, party - a choice between the two studied parties, pre_trained_model - the external pre-trained model - should point to the model.txt file downloaded from http://vectors.nlpl.eu/repository/20/69.zip, epochs - number of epochs to train, output_folde - directory for saving results; also uses the two pickle files created from the data_prep script.   
**Output:** fine-tuned models in a gensim format  
**Example run:** pyhton fine_tune_nlpl.py --time_span 1988-2020 --party s --pre_trained_model model --epochs 50 --output_folder output  

**similarity_lists.py:**  
Given a folder, containing the fine-tuned models, a type of pre-training and a party, goes through the models to extract top 20 most similar terms to the studied words and goes through the data to extract those terms' frequency in the fine-tuning data. Outputs to a '|' separated file. The resulting files contain the following columns:   
* term - the term that is calculated to be close to the studied word (e.g. to "crime");  
* mean similarity score - mean cosine distance between the term and the studied word calculated from the bootstrapped runs;  
* standard deviation - standard deviation of the similarity score;  
* term frequency - how many times the term appears in the training data (for the given party)   
* document frequency - in how many documents the term appears at least once (for the given party)  

**Input required:** model_folder - folder containing the fine-tuned models, time_span - a choice between the three studied time periods, party - a choice between the two studied parties, pre_train - a choice between the two types of pre-training  
**Output:** a txt file with top 20 most similar terms to the studied words for the specified model  
**Example run:** python similarity_lists.py --model_folder nlpl_1988_2009 --time_span 1988-2009 --party s --pre_train nlpl  


-----Versions-----  
Python 3.8.12  
bs4	4.11.1   
re 2.2.1  
csv 1.0  
argparse 1.1   
gensim 3.8.3  
sklearn 1.0.2  
numpy 1.21.5  

-----Computational resources-----  
Run on 16 core Intel(R) Xeon(R) Gold 6338 CPU @ 2GHz, each NLPL model takes on average 45 minutes to fine-tune on this hardware for the full 50 epochs, each Riksdag model takes 35 minutes to fine-tune on the full 50 epochs.





