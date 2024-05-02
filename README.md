# Tweets2Stance
Code and dataset for Tweets2Stance.

"From Tweets to Stance: An Unsupervised Framework for User Stance Detection on Twitter": https://dl.acm.org/doi/abs/10.1007/978-3-031-45275-8_7

Cite:

    @inproceedings{gambini2023tweets,
      title={From Tweets to Stance: An Unsupervised Framework for User Stance Detection on Twitter},
      author={Gambini, Margherita and Senette, Caterina and Fagni, Tiziano and Tesconi, Maurizio},
      booktitle={International Conference on Discovery Science},
      pages={96--110},
      year={2023},
      organization={Springer}
    }
 
## Installation

Move to the project folder
`cd path/to/Tweets2Stance`

Create the python virtual environment:

`virtualenv venv --python=python3.7`

Activate the virtual environment:

`source ./venv/bin/activate`

Install libraries:

`pip install -r utils/requirements.txt`

## Pipeline to follow:
If you wish to use a language model fine-tuned on tweets, the preprocessing occurs in two ways: one that removes hashtags, emojis, and mentions at the beginning of the tweet (suffix=''), and another that retains them (suffix: '_with_@#emoji').

The parameters of the Tweets2Stance framework are:

* Language Model for zero-shot classification
* Dataset (i months before the election)
* Threshold th: threshold to filter tweets depending on the topic
* Algorithm: one of the 4 implemented algorithms

Steps to follow:

* ensure that the dataframe containing various tweets has at least the fields: _created\_at_, _lang_, _tweet\_id_, _screen\_name_, _tweet_ 
* **preprocessing**: 
  * `$nohup python3 preprocessing.py`
  * produces a pkl file `f'{INST.DATA_PATH_FOLDER}/{election}/{election}_{dataset}_preprocessed{suffix}'`
* **translation**: 
  * `$nohup python3 translation.py`
  * starting from the processed tweets, a pkl file is produced `f'{INST.DATA_PATH_FOLDER}/{election}/{election}_{dataset}_preprocessed_translated{suffix}'`
* **classification\_ZSC**: 
  * `$nohup python3 ZSC.py -i in_ZSC.json >/dev/null 2>&1 &`
  * from the processed and translated tweets, a pkl file is produced `f'{INST.DATA_PATH_FOLDER}/{election}/{model_name.split("/")\[-1\]}/{election}_{dataset}_zsc_tweets{suffix}'`
  * the method _pretty\_save\_df()_ produces a dataframe suitable for subsequent processing. The produced pkl file is
  `f'{INST.DATA_PATH_FOLDER}/{election}/{model_name.split("/")\[-1\]}/{election}_{dataset}_df_zsc_tweets{suffix}'`. 
  It is a dataframe in which each row contains classification information for a pair `(tweet, sentence)`. Remember that
each sentence is associated with only one topic. The _created\_at_ field is important, so you can classify only
the tweets from the largest dataset and build the other datasets starting from this one.
* **tweets2stance**: 
  * `$nohup python3 tweets2stance.py`
  * from the classified tweets, a pkl file is produced `f'{INST.DATA_PATH_FOLDER}/{election}/{model_name.split("/")\[-1\]}/{dataset}/{algorithm}/df_agreements{suffix}'`


## Notes on executing the ZSC
Activate the virtual environment:

`source ./venv/bin/activate`

Execute Zero-Shot Classification (ZSC):

`$nohup python3 ZSC.py -i ./inputs/in_ZSC.json >/dev/null 2>&1 &`

Input file `in_ZSC.json`:

**"begin" field**: for each election, the language models (through a list) to begin classification are indicated. If all models
are still to be used, simply write `"models": "all"`. The field `index_begin_from` is always set to `0`.

**"restart" field**: indicates for which elections and language models it is necessary to restart the classification. In this case,
the field `index_begin_from` indicates the index of the tweet from which to restart. This index can be deduced from how many tweets have
already been classified and saved in the pkl file `f'{INST.DATA_PATH_FOLDER}/{election}/{model_name.split("/")\[-1\]}/{election}_{dataset}_zsc_tweets{suffix}'`

    {
      "begin": {
        "GB19": {
          "models": "all",
          "index_begin_from": 0
        },
        "CA19": {
          "models": "all",
          "index_begin_from": 0
        },
        "AB19": {
          "models": "all",
          "index_begin_from": 0
        },
        "AU19": {
          "models": "all",
          "index_begin_from": 0
        },
        "SK20": {
          "models": "all",
          "index_begin_from": 0
        },
        "BC20": {
          "models": "all",
          "index_begin_from": 0
        },
        "CA21": {
          "models": "all",
          "index_begin_from": 0
        },
        "NS21": {
          "models": "all",
          "index_begin_from": 0
        },
        "NFL21": {
          "models": "all",
          "index_begin_from": 0
        }
      },
      "restart": {}
    }

## Notes on Tweet2Stance's output

The output of the `tweets2stance.py` script is a dataframe like this:

| screen_name  | sentence                                                                                 | topic                               | avg_agreement | agreement_level | threshold_topic | num_tweets |
|--------------|------------------------------------------------------------------------------------------|-------------------------------------|---------------|-----------------|-----------------|------------|
| albertaNDP   | anti-abortion activists should be able to protest in the immediate vicinity of an abortion clinic | anti-abortion protests close to clinic | 0.425605      | 3               | 0.5             | 25         |
| ...          | ...                                                                                      | ...                                 | ...           | ...             | ...             | ...        |


* The `avg_agreement` is set only for `algorithm_1`
* The `num_tweets` is the number of tweets after the _topic filtering_ step