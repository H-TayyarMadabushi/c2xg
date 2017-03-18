
c2xg 0.2
=============

Computational Construction Grammar, or c2xg, is two things: 

(1) A Python package for the unsupervised learning of CxG representations along with tools for vectorizing these representations for computational tasks

(2) A discovery-device grammar that learns falsifiable and replicable CxGs from observed unannotated text data

Why CxGs? Constructions are grammatical entities that allow the straight-forward quantification of linguistic structure.


Installation
--------------

		pip install c2xg

Environment and Dependencies
----------------------------------

This package is meant to run in Python 3.5 with a number of dependencies. The easiest way to maintain the necessary environment is to use Anaconda Python: https://www.continuum.io/downloads

The package can be installed within Anaconda using the command:

		conda install c2xg
		
This makes it easier to maintain the necessary environment. The package works with the dependency versions listed below. It will likely work with older versions of some packages but has not been tested with them. For example, older versions of numexpr have been known to cause issues withs pandas and may lead to lost candidates.

Dependencies:
		
	Python 3.5
	cytoolz 0.7.4
	gensim 0.12.2
	matplotlib 1.5.0
	numexpr 2.5
	numpy 1.10.4
	pandas 0.18.0
	scipy 0.16.0
	sklearn 0.17

Usage
=====
C2xG has two main classes:

	c2xg.Parameters loads and initializes the settings needed for running C2xG; these values are set in a file
	
	c2xg.Grammar contains the grammar resources used across all stages of the package
	
Initialize the package with the following commands:

	import c2xg
	Parameters = c2xg.Parameters("filename")
	
The parameters class takes as input a string indicating the name of the parameters file. Now, run the API using the following template, where Parameters is an initialized c2xg.Parameter object:

	c2xg.learn_c2xg(Parameters)
	c2xg.learn_mwes(Parameters)
	
All functions in the API take a c2xg.Parameters object as an argument. The c2xg.Grammar object can be passed to each function or, if not passed, loaded from file.
	
API
====

Each function in the API takes a Parameters object and either creates the Grammar object or loads it from the file specified in the parameters.

Automate Pipeline
------------------

learn_c2xg			

		Umbrella function for entire learning pipeline (from learn_mwes to learn_constructions).

Individual Learning Functions
------------------------------

learn_dictionary		

		Use GenSim to create the dictionary of semantic representations needed for c2xg.

learn_rdrpos_model		

		Use RDRPOS Tagger Dependency to learn a new pos-tagging model.

learn_idioms				

		Use c2xg to learn a dictionary of idioms (lexical constructions).

learn_constituents	 	

		Use c2xg to learn a constituency grammar.

learn_constructions 	

		Use c2xg to learn a full Construction Grammar with lexical, MWE, semantic, and constituent representations.

learn_usage				

		Prepare to use TF-IDF weighting during feature extraction.
		
learn_association

		Produce a CSV file of association measures for sequences of a given length and types of representation

Helper Functions
-----------------

annotate_pos			

		Tokenize, pos-tag, mark emojis, and convert to CoNLL format.

get_indexes				

		Get indexes of representation types.

get_candidates			

		Getcandidate sequences from input files (covers MWEs, Constituents, and Constructions).

get_association			

		Get vector of association values for each candidate.

get_vectors				

		Get vector of CxG usage for input files.

Evaluation Functions
----------------------

examples_constituents	

		Get examples of predicted constituents by type. (*Not stable in v 0.2)

examples_constructions	

		Get examples of each predicted construction. (*Not stable in v 0.2)
		
Command-Line Usage
==================

	(1) Begin a Python interpreter

	(2) Import the package:
	
			import c2xg
	
	(3) Initialize the parameters object:
	
			Parameters = c2xg.Parameters("filename")
			
	(4a) Run the API, loading grammar objects from disk:
	
			c2xg.learn_constituents(Parameters)
			
	(4b) Run the API, initializing and then passing grammar objects:
	
			Grammar = c2xg.Grammar()
			c2xg.learn_constituents(Parameters, Grammar)	


Input Formats
===================

This section describes the input formats for the different components.

(1) Creating Semantic Dictionary

	Input: Unannotated text, one sentence per line. Tokenization and emoji identification are performed on each line.
	
(2) Creating Models of Grammar and Usage
	
	Input: Annotated: CoNLL format of tab-separate fields [Word-Form, Lemma, POS, Index]. 
	Use <s:ID> to assign ids to documents.
	
	Input: Unannotated: Plain text with line breaks for documents / sentences as desired. 
	[In both cases, each line is assumed to be a "text" or the containing unit of analysis; instances can be separated by the "|" character for aggregation]
			
(3) Extracting Feature Vectors
	
	Input with Meta-Data: 		Field:Value,Field:Value\tText
	Input without Meta-Data:	Plain text with line breaks (\n) for documents or sentences depending on the level of analysis.
	
	
Feature Extraction
=========================

Given a language-specific CxG, the get_vectors and learn_usage functions convert that grammar into a vector representation of texts or sentences (i.e., one unit per line in the input files). There are two modes and three quantification methods for creating vectors:

	vector_scope = "Full": Constructions and lexical / POS / semantic features
	vector_scope = "Lexical": Only lexical features
	vector_scope = "CxG": Only construction features	
	
	relative_freq == True: Quantify using the relative frequency of the feature in given sentence or text (as negative logarithms)
	relative_freq == False: Quantify using unadjusted raw frequency of the feature
	use_centroid == True: Extract vectors with centriod normalization learned using learn_usage. This is functionally equivalent to TF-IDF scaling
	
	Centroid normalization first finds the probability of a given feature in the background corpus. This is stored after running learn_usage in separate centroid_df models for the full grammar and for the lexical-only features. During extraction, if centroids are used for representation, this is converted into negative logarithms of the inverted joint probability of each feature occuring as many times as it does in a message.
	
	*Note: Feature extraction is stable in v 0.1 but not in v 0.2