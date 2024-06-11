# Genome-NLP benchmark 

Workflow Diagram :

<img width="586" alt="image" src="https://github.com/CodelyUnicorn/Genome-NLP/assets/139937131/50c0522a-0e90-44d7-8833-2be7f4fb402f">


## Steps

### 1. Setting Up Dataset
- Downloaded the .FASTA file containing genomic sequence from the National Centre for Biotechnology Information (NCBI) FTP site.
- Utilized "Homo sapiens", the latest GRCh38 human genome assembly sequences for evaluation.

### 2. NLP Pre-processing
NLP aids in efficient pre-processing and extracting 'meaning' from human language data, facilitating better understanding and analysis in genomics.
- **Data Cleaning:** Ensured each sequence identifier and its corresponding DNA sequence are correctly formatted. Removed any non-ATCG characters (DNA molecule: adenine (A), cytosine (C), guanine (G), and thymine (T)) or whitespace.
- **Case Conversion:** Converted sequences to upper or lower case.
- **Normalization:** Standardized the sequences to a common format or length if necessary. Removed any redundant or duplicate sequences.
- **Tokenization:** Trained a SentencePiece tokenizer on the sequences.
- **Saving Tokenizer:** Saved the trained tokenizer to a specified file.

### 3. Data Representation
- The dataset was divided into training (90%), testing (5%), and validation (5%).

### 4. Pre-Training BERT Model
- Utilized DNABERT tokenizer.
- Utilized GenomicBERT tokenizer.

For Pre-training GenomicBERT- following lines of codes were changed:

Changes to GenomicBERT pipeline to replace Sequence classification function with Foundation model building (Train.py)
 
1.	Change data collator function in Tyrone's code #138 to do Masking of tokens
from transformers import DataCollatorForLanguageModeling
 
data_collator = DataCollatorForLanguageModeling(
    tokenizer=tokenizer, mlm=True, mlm_probability=0.15
)
 
2.	Change distilbert config; no need to mention num_labels, remove that. Line 223
config = DistilBertConfig(
            vocab_size=vocab_size,
            num_labels=2, #remove this line
            # output_hidden_states=True,
            # output_attentions=True
            )
3.	Change line 230,234 for model init..ideally all DistilBertForSequenceClassification to be replaced with DistilBertForMaskedLM
model = DistilBertForMaskedLM(config=config)

4.	#Change trainer line 302 : Can remove customized compute metrics function 
    trainer = Trainer(
        model_init=_model_init,
        tokenizer=tokeniser,
        args=args_train,
        train_dataset=dataset["train"],
        eval_dataset=dataset["test"],
        compute_metrics=_compute_metrics, #remove this line
        data_collator=data_collator,
    ) 
 
5.	Can add Evaluation function to measure perplexity. Function readily available in Transformers Trainer function.
import math
eval_results = trainer.evaluate()
print(f"Perplexity: {math.exp(eval_results['eval_loss']):.2f}")

6. Also comment out the following lines 161 to 163
    #by default this will be "labels"
    #if type(dataset[i].features[args.label_names[0]]) != ClassLabel:
        #dataset[i] = dataset[i].class_encode_column(args.label_names[0])    




    
### 5. Fine-tuning for Promoter and Non-Promoter Classification
By fine-tuning the pre-trained models on promoter and non-promoter datasets, the models learn to distinguish between these two types of sequences based on their features and patterns.


## Acknowledgements
We extend our sincere gratitude to Dr. Ke Deng, our Academic Supervisor, for providing invaluable guidance and oversight throughout this journey. Additionally, we wish to express our appreciation to Dr. Sonika Tyagi, an Associate Professor of Digital Health and Bioinformatics at the School of Computational Technologies, RMIT University Australia, and an affiliate Machine Learning lead scientist at Central Clinical School, Monash University Australia. Dr. Tyagi's expertise in developing innovative machine learning tools and pipelines, and her application of these methods to address biological and clinical research questions, significantly contributed to the success of our project.
Lastly, we extend our gratitude to all team members for their dedication, collaboration, and contributions to this project.



