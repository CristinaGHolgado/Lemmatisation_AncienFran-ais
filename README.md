# OLD FRENCH LEMMATIZATION
<img src="https://i.ibb.co/Tqwv77b/battle-of-roncevaux-from-bl-royal-16-g-vi-f-178-8046ca-1024.jpg" width="1050" height="200">
This repository includes a set of scripts for the lemmatization of medieval French using 4 different tools : <a href="https://www.cis.uni-muenchen.de/~schmid/tools/TreeTagger/">TreeTagger</a>, <a href="http://stella.atilf.fr/LGeRM/">LGeRM</a>, <a href="https://cran.r-project.org/web/packages/udpipe/index.html">UDPipe (CRAN R packgage)</a> and <a href="https://pypi.org/project/nlp-pie/">NLP Pie</a>. Two different combination systems for lemmatizacion from these tools as provided as well. The goal is to assess the lemmatization as well as analizing the advantages (and limits) these tools may offer, especially for each part-of-speech and unknown words.
The work contained in this repository was carried out within the framework of a master's interpship (M2 Linguistics-NLP, Strasbourg University) at the <a href="https://www.atilf.fr/">ATILF</a> laboratory as part of the projet ANR <a href="https://www.lattice.cnrs.fr/projets/projet-anr-profiterole/"> Profiterole </a>(PRocessing Old French Instrumented TExts for the Representation Of Language Evolution, ANR-16-CE38-0010). PROFITEROLE is a project financed by the French National Research Agency (ANR) focused into the syntactic aspects of Old french.

##### Additional information
Repository of the project : https://gitlab.huma-num.fr/lemmatisation-fro/bfm-lem  
TALN 2021 article : https://gitlab.huma-num.fr/lemmatisation-fro/bfm-lem/-/blob/master/doc/taln-recital2021.pdf

## Data source and corpus features  
The annotated texts used for training and evaluation are part of the BFMGOLDLEM corpus and gather a total of 431 144 tagged and lemmatized forms. It is part of the <a href="http://bfm.ens-lyon.fr/">BFM</a> (Base de Français Médiéval), an online database of french medieval texts covering the period from the 9th to the 15th century whose total number of word occurrences amounts to 4.7 million. 

The corpus used for this project is composed of two sources of which a predominant one belongs to only one author (Chrétien de Troyes). It is thus an important corpus, but not very diversified (a single author, a single manuscript, a single genre). It has its own reference system of lemmas, which correspond for the most part to the entries of the dictionary Tobler-Lommatzsch (TL) dictionary, which favors older forms. The rest of the corpus has been lemmatized in the framework of the BFM and is much more diversified. Files use CONLL-U format and include tokenized inflected forms, morphological labels based on <a href="http://bfm.ens-lyon.fr/spip.php?article176">Cattex 2009</a> and lemmas.

## External libraries  
pandas (1.0.1)
scikit-learn (0.23.0)
numpy (1.18.1)

## Lemma standarization  
Before trainning and tagging, the source lemmas (DECT lemmas) have been converted into DMF lemmas using the <a href="https://github.com/sheiden/Medieval-French-Language-Toolkit/blob/master/clfrolex-3.0.tsv">FROLEX</a> lexicon (table of equivalences between lemmas) according to the following procedure:

→ **DECT** lemmas are converted to **DMF** lemmas.  
→ If not avaliable equivalences in the lexicon, they are converted to **TL** lemmas.  
→ Otherwise, they are converted to **GDF** or, ultimately, to **BFM** lemmas.  

Not converted lemmas are saved into an external file. Previos version old the lemmas (those converted) are kept in the last column of the CONLL-U file which contains the source lemma information (e.g.: *XmlId=w_CharretteKu_1|LemmaSrc=DECT* if none (text source and id, lexicon source), *XmlId=w_CharretteKu_7|LemmaSrc=DMF|LemmaDECT=voloir* if converted (text source and id, lexicon source updated, previous lemma)).  
In the standardized corpus, the 424 836 lemmas are thus DMF (98.54%), 4 512 DECT (1.04%), 965 BFM (0.22%), 801 TL (0.18%) and 30 GDF (0.01%).  

## Folder's description

*Normalisation_lemmes/* folder include all files from this step :  

#### files
-----
- `DECT-TL_vers_DMF.py` : script for lemma conversion from a prior corpus  
- *fpath_normalisation.txt* : input data paths (corpus folder and path to lemmas equivalences file(s))  
- `count_DECT.py` : counting of the number of lemmas converted by referential and number of remaining lemmas (not converted)  

#### folders
--------
- *corpus_normalise* : output files (nomalized files)  
- *correspondances_lemmes* : folder containing conversion reference files  
- *dect_restants* : remanining not converted lemmas  
- *intermeridaires* : intermediate files  


## Preprocessing corpus for train & tagging
After lemma standardization, corpus files are divided into a 10 tests&train split and pre-processed. More detailed info about the produre and texts features are avaliable at the following site :  <a href="https://gitlab.huma-num.fr/lemmatisation-fro/bfm-lem/-/blob/master/doc/Protocole%20de%20test.pdf"> Protocole de tests </a> and <a href="https://gitlab.huma-num.fr/lemmatisation-fro/bfm-lem/-/blob/master/doc/taln-recital2021.pdf"> section 3.2. </a>

### Train data
A different preprocessing step is followed regarding the tool specifications in the input data. For instance, each tool requires the following training data structure :  
  
- **TreeTagger :** Tab file composed by forms, POS and lemma(s)
    ||||||
    |-|-|-|-|-|
    |form1|POS|lemma1|-|-|
    |form2|POS|lemma2|POS2|lemma2|  
    |...|||||
    
    Forms corresponding to multiple lemmas are separated in the corpus by a bar ("|").  
    However, when some forms from the corpus correspond to lemmas fom different parts of speech, <a href="https://gite.lirmm.fr/advanse/sentiment-analysis-webpage/blob/master/resources_on_server/TreeTagger/cmd/make-lex.perl">`make-lex.perl`</a> can be helpful to build the lexicon (traning data for TreeTagger).     This script has been used to build our lexicon as it includes multiple homographs.  
    In addition to the training data, an openclass file can be used.
    
- **UDPipe :** Conventional CoNLL-U structure (Requires at least sentence id, sentence and tokenized&lemmatized sentence)  
  
    \# text = "Cil qui fist d'Erec [...] ."  
    \# sent_id = 1
 
  |||||||||  
  |---|------|-------|-------|--------|--------------|-----|---------------------------------------------------|  
  | 1 | Cil  | cil   | PRON  | PROdem | PronType=Dem | ... | XmlId=w_CligesKu_1\|LemmaSrc=DMF\|LemmaDECT=cel   |  
  | 2 | qui  | qui   | PRON  | PROrel | PronType=Rel | ... | XmlId=w_CligesKu_2\|LemmaSrc=DMF\|LemmaDECT=qui   |  
  | 3 | fist | faire | VERB  | VERcjg | VerbForm=Fin | ... | XmlId=w_CligesKu_3\|LemmaSrc=DMF\|LemmaDECT=faire |  
  | 4 | d'   | de    | ADP   | PRE    | _            | ... | XmlId=w_CligesKu_4\|LemmaSrc=DMF\|LemmaDECT=de    |  
  | 5 | Erec | Erec  | PROPN | NOMpro | _            | ... | XmlId=w_CligesKu_5\|LemmaSrc=DECT                 |  
  | 71 | .    | .     | PUNCT | PONfrt | _            | ... | XmlId=w_CligesKu_71\|LemmaSrc=DMF\|LemmaDECT=.    |  

- **NLP Pie** : Lemmas and POS tags are trained independiently. Two files are generated for the training.
  |FORM|POS| 
  |-|-|
  |token|pos|  
  
  |FORM|lemma|
  |-|-|
  |token|lemma|  
  

### Test data
Since the corpus files are already tokenized, we skip tokenization step. For illustrative purposes, the structure is the following :  
|||||||||
|---|------|-------|-------|--------|--------------|-----|---------------------------------------------------|  
| 1 | Cil  | cil   | PRON  | PROdem | PronType=Dem | ... | XmlId=w_CligesKu_1\|LemmaSrc=DMF\|LemmaDECT=cel   |
|...|...|...|...|...|...|...|...|

- TreeTagger, UDPipe and Pie : A file per test is genetared contaning the forms column. 
- LGeRM : Uses iso-8859-1 as default input data encoding. Forms are previously converted into this encoding and re-encoded to uft-8 after tagging and lemmatization.  

## 4. Training & Annotation

- **TreeTagger** : `train_annotate_treetagger.py` launchs training (*par* files are created forch each test) and annotation of each test forms file.
- **LGeRM** : `annotate_lgerm.py` launch annotations and converstion of output files to utf-8. `lgerm.bash` is required and avaliable by contacting the author.
- **UDPipe** : `train_udpipe.R` launchs training and annotation. A more recent script is avaliable with an improved lemmatization : https://colab.research.google.com/drive/1RfLAWYHelhp8iVuLeLe-tfGKfCmFnEIJ?usp=sharing  
- **Pie** : run `!pie train default_settings.json` after setting the parameters (at least input path and model name) in the json file or use <a href="https://colab.research.google.com/drive/1lTXwt55hTxRhyP-HRXm-MUzSCcSt_rbb?usp=sharing">train Pie notebook</a>. Function `setParam()` only works with Pie (v0.3.6). Since tokenization is skipped, funtion `noTokenizing()` is essential.
