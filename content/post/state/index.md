+++
title = "Exploring state of Union Addresses Data"
subtitle = "Analysis of state of Union Addresses"

date = 2019-03-13T00:00:00
lastmod = 2019-03-13T00:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["admin"]

tags = ["NLP","NLTK", "Bigrams", "Tokenize","Glutenberg","StopWords","Lemmatization","Words"]
summary = "Used NLP techniques to analyze Annual US addresses given by the President "

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = "Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""

  # Show image only in page previews?
  preview_only = false

# Set captions for image gallery.

[[gallery_item]]
album = "gallery"
image = "theme-default.png"
caption = "Default"

[[gallery_item]]
album = "gallery"
image = "theme-ocean.png"
caption = "Ocean"

[[gallery_item]]
album = "gallery"
image = "theme-forest.png"
caption = "Forest"

[[gallery_item]]
album = "gallery"
image = "theme-dark.png"
caption = "Dark"

[[gallery_item]]
album = "gallery"
image = "theme-apogee.png"
caption = "Apogee"

[[gallery_item]]
album = "gallery"
image = "theme-1950s.png"
caption = "1950s"

[[gallery_item]]
album = "gallery"
image = "theme-coffee-playfair.png"
caption = "Coffee theme with Playfair font"

[[gallery_item]]
album = "gallery"
image = "theme-cupcake.png"
caption = "Cupcake"
+++

## State Of Union Data Set
The State of the Union Addresses dataset is a collection of annual speeches delivered by the presidents of the United States. It ranges from George Washington to Barack Obama, to a joint session of the United States Congress for the span of 1790-2016. This dataset contains the two texts combined, which are small subsets of the Project Gutenberg Ebook corpus. Here is the Project Gutenberg Ebook [web page](http://www.gutenberg.org/)

You can find the data [here](https://github.com/Guneev9/State-of-the-Union-Addresses-NLP-)

James Linden’ has composed both the files. Every message has sections indicating to whom it is being cited. Naming convention is President Name followed by Title “State of Union Address” followed by the Date of address. The messages have been sorted on the basis of year it was delivered. Part 1 contains messages from year 1790 to 1860 and Part 2 contains messages from 1946 to 2016. First file contains 71 messages whereas File 2 contains 970 messages.

## Loading the Libraries

Please import following python packages :
```{python}
import nltk as n
import re
from nltk.collocations import *

```
## Data Extraction

We create the below function to extract the text from file. I have used Plain Text Corpus Reader of NLTK to access text files and treat them as regular corpora. We access the name of the files by fileId()[0] and then reading all the text of the corpus.

```{python}
def Get_Corpus(Text):
    #reading
    from nltk.corpus import PlaintextCorpusReader
    corpus_read= PlaintextCorpusReader('.','.*\.txt')
    Part= corpus_read.raw(Text)
    Part_file= corpus_read.fileids()[0]
    Part_string = corpus_read.raw(Part_file)
    return Part_string
```
## Data Tokenization

We use word tokenizer to divide a string into substrings by splitting on the specified string.

```{python}
def Tokenize_Corpus(Words):
    Corpus_Tokens = n.word_tokenize(Words) 
    #strip() is used to remove extra whitespaces from the text
    Tokens = [Token.strip() for Token in Corpus_Tokens]
    return Tokens
```
## Making the text more meaningful

Now we filter the tokens by removing non-aplhabetical ([,],*,41). d) text using simple regex.

```{python}

def Get_aplha_words(words):
# pattern to match a word of non-alphabetical characters
    pattern = re.compile('^[^a-z]+$')
    if (pattern.match(words)):
        return True
    else:
        return False
```

## Let's Process it further :

Our next step involves [lemmatisation](https://en.wikipedia.org/wiki/Lemmatisation). Its aim is to remove conjugational endings, and to return the base form of a word, known as the lemma. For example: Conjugational form of word Organize (base form) is Organizing, organizes etc.

Since the process may involve complex tasks such as understanding context and determining the part of speech(pos) of a word in a sentence, it can be a hard task to implement a lemmatiser for a new language. 

It takes a part of speech parameter, “pos”. If not supplied, the default is “noun”. We create a function called get_pos to get pos of the word to get lemma’s.
For this, import another corpus reader “wordnet” and Counter to keep the count.

```{python}
def get_pos(word):
    #another reader
    from nltk.corpus import wordnet 
    #synsets() is used to lookup a word
    #pos argument is used to constrain the part of speech of the word
    w_synsets = wordnet.synsets(word)
    #for counting the 
    from collections import Counter
    pos_counts = Counter()
    pos_counts["n"] = len([item for item in w_synsets if item.pos()=="n"])
    pos_counts["v"] = len([item for item in w_synsets if item.pos()=="v"])
    pos_counts["a"] = len([item for item in w_synsets if item.pos()=="a"])
    pos_counts["r"] = len([item for item in w_synsets if item.pos()=="r"])
    
    most_common_pos_list = pos_counts.most_common(3)
    # first index for getting the top POS from list, second index for getting POS from tuple
    return most_common_pos_list[0][0]
```
 
The below function is then used to fetch the base form :\
```{python}

def lemmatize_text(word):
    # NLTK has a lemmatizer that uses WordNet as a dictionary
    from collections import Counter 
    # To get words in dictionary with their parts of speech
    from nltk.corpus import wordnet 
    # lemmatizes word based on it's parts of speech
    from nltk.stem import WordNetLemmatizer
    wnl = WordNetLemmatizer()
    lemma_words=[wnl.lemmatize(t, get_pos(t)) for t in word]  

    return lemma_words
```

Now, we remove the stop words since they don't make any sense.
I have added some stop words after looking at the tokens to the default stop words list.

```{python}
def remove_stopwords(words):
    stopwords = n.corpus.stopwords.words('english')
    #custom stop words, not much meaningful for analysis point of view
    stopwords = stopwords + ['ebook', 'ebooks', 'january', 'february', 'march',
                            'april', 'may', 'june', 'july', 'august',
                            'september', 'october', 'november', 'december',
                             'state','address','copyright','union']

    filter2_words = [w for w in words if w not in stopwords]
    return filter2_words]
```

## Which are the Top frequency words?

We extract top 50 words by frequency normalized by the length of the document. We filter the frequency by normalizing it against length of filtered Tokens.\
This means that the distribution will give scores on behalf of filtered tokens, not the whole document.\
We use the filtered words list for the length of document instead of taking the raw corpus length. This is because the latter case will not give any good results since it contains text which is not useful at all. Hence, we remove those tokens from the token list.\

Import FreqDist package to count tota occurance of same token in a corpus. 

```{python}
def Top50_freq(w):
    from nltk import FreqDist
    words={}
    Part1dist = FreqDist(w)
    d = dict(Part1dist.most_common(50))
    for token,f in d.items():
        #normalizing it by length of filtered document w
         words[token]=f/float(len(w))
    return words
    
```
## Are there any Bigrams?

Another	way	to	look	for	interesting	characterizations	of	a	corpus	is	to	look	at	pairs	of words	that	are	frequently	collocated,	that	is,	they	occur	in	a	sequence	called	a	bigram.
(The	nltk.bigrams()	function	returns	a	generator, but	we	can	turn	it	into	a	list	by	applying	
the	type	‘list’	to	it	as	a	function.)\
For more simplicity, we combine the filteration steps in the function below once again. Collocation is a sequence of words that occur together unusually often. The collocations package provides collocation finders which by default consider all ngrams in a text as candidate collocations. 
\
We use bigram_with_filter() function to apply various filter functions to finder. We first apply Get_aplha_words() function to get only alphabetical words. Then further we apply filter to remove stop words. We are not lemmatizing the words, so we include ‘addresses’ also in stop words list.
At last we apply filter to get words having least frequency of 2 since otherwise result will not be much helpful. We sort the scores in order of decreasing frequency. The Result has given the top 50 bigrams by frequencies of meaningful important bigrams.
```{python}
# setup for bigrams and bigram measures
from nltk.corpus import stopwords

def bigram_with_filter(w):
    finder = BigramCollocationFinder.from_words(w)
    bigram_measures = n.collocations.BigramAssocMeasures()
    stopwords = n.corpus.stopwords.words('english')
    #custom stop word
    stopwords = stopwords + ['ebook', 'ebooks', 'january', 'february', 'march',
                            'april', 'may', 'june', 'july', 'august',
                            'september', 'october', 'november', 'december',
                             'state','address','copyright','union','addresses']
    # apply a filter to remove non-alphabetical tokens
    finder.apply_word_filter(Get_aplha_words)
    #removed low frequency words
    finder.apply_freq_filter(2)
    # apply a filter to remove stop words
    finder.apply_word_filter(lambda w: w in stopwords)
    scored = finder.score_ngrams(bigram_measures.raw_freq)   
    for bscore in scored[:50]:
        print (bscore)
        
```
## Frequency by Mutual Information Scores

Mutual Information is a score introduced in the paper by Church and Hanks, where they defined it as an Association Ratio. Note that technically the original information theoretic definition of mutual information allows the two words to be in either order, but that the association ratio defined by Church and Hanks requires the words to be in order from left to right wherever they appear in the window
In NLTK, the mutual information score is given by a function for Pointwise Mutual Information, where this is the version without the window.\
Here, Apply	the	non-alphabetical	and	stopword	filters,	noting	that	the	filters	substantially	reduce the	numbers	of	bigrams	scored.
\
When	you	apply	the	Mutual	Information	score	to	small	documents,	the	results	don’t	really make	sense.		In	particular,	here	all	our	scores	are	the	same.		It	is	recommended	to	run	the	
PMI	scorer	with	a	minimum	frequency	of	5,	which	will	make	more	sense	on	very	large	 documents.\
Then,I have run PMI score with a minimum frequency because it will not be useful to apply the Mutual Information score to all the bigrams as results don’t really make sense and expressions are also very infrequent, because uniquely occurring pairs of words get high scores. Thus, it will be good to filter it, to make sense for very heavy documents.PMI gives strange results when frequencies are very low e.g. 1-3 tokens, thus set a minimum frequency for the collocates, which takes care of most of the problem. \


```{python}
def Top50bigrams_MutualInfoScore(w):  
    finder2 = BigramCollocationFinder.from_words(w)
    bigram_measures = n.collocations.BigramAssocMeasures()
    stopwords =  n.corpus.stopwords.words('english')
    stopwords = stopwords + ['ebook', 'ebooks', 'january', 'february', 'march',
                            'april', 'may', 'june', 'july', 'august',
                            'september', 'october', 'november', 'december',
                             'state','address','copyright','union','addresses']
    # apply a filter to remove non-alphabetical tokens
    finder2.apply_word_filter(Get_aplha_words)
    finder2.apply_word_filter(lambda w: w in stopwords)
    finder2.apply_freq_filter(5)
    scored = finder2.score_ngrams(bigram_measures.pmi)
    for bscore in scored[:50]:
        print (bscore)
```

The result has given us the bigram pairs which are frequent by their associativity.

## Let's Apply all above to data

Finally, we combine all the code into two separate functions, one for each part.\

For Part-1 :\
```{python}
def Part_1_execution():
    #fetching the corpus
    Part_string=Get_Corpus('state_union_part1.txt')
    #Tokenization
    Tokens=Tokenize_Corpus(Part_string)
    #lowercase conversion after fetching alphabetical words
    Part1_filter1_words= [w.lower() for w in Tokens if not Get_aplha_words(w)]
    #Lemmatization
    Part1_filter2_words=lemmatize_text(Part1_filter1_words)
    #Removal of Stop Words
    Part1_filter3_words=remove_stopwords(Part1_filter2_words)
    #Functions calling and Results
    Top50_freq_words=Top50_freq(Part1_filter3_words)
    print("####################### PROBLEM-2(STATE UNION PART-1 FILE) ##########################")
    print("#############################Top 50 words by frequency###########")
    print (Top50_freq_words)
    print("##################### Top 50 Bigrams by frequency ##################################")
    bigram_with_filter(Tokens)
    print("###################### Top 50 Bigrams by thier Mutual Information Scores ########################")
    Top50bigrams_MutualInfoScore(Tokens)
```
For Part-2 :\

```{python}
def Part_2_execution():
    #fetching the corpus
    Part_string=Get_Corpus('state_union_part2.txt')
    #Tokenization
    Tokens=Tokenize_Corpus(Part_string)
    #lowercase conversion after fetching alphabetical words
    Part2_filter1_words= [w.lower() for w in Tokens if not Get_aplha_words(w)]
    #lemmatization
    Part2_filter2_words=lemmatize_text(Part2_filter1_words)
    #Stop words Removal
    Part2_filter3_words=remove_stopwords(Part2_filter2_words)
    #Functions calling and results
    Top50_freq_words=Top50_freq(Part2_filter3_words)
    print("####################### PROBLEM-3 (STATE UNION PART-2 FILE) ##########################")
    print("##################### Top 50 words by frequency #############################")
    print (Top50_freq_words)
    print("######################## Top 50 Bigrams by frequency ###############################")
    bigram_with_filter(Tokens)
    print("###################### Top 50 Bigrams by thier Mutual Information Scores ########################")
    Top50bigrams_MutualInfoScore(Tokens)
```

## Any Differences ?

Both the files have similar language because they are written by the same author and thus have the similar writing style with similar kind of words. Also, the author has used same words in both the files in writing the speeches. \
Because of the same reason, the Top-50 frequency words also give similar results for both the documents. \
The only difference is the speeches and their delivered time.

## How are the top 50 bigrams by frequency different from the top 50 bigrams scored by Mutual Information?

For both the documents, Top-50 bigrams scored by Mutual Information are different due to following reasons:  \
\
1. The bigrams which we got from bigram_with_filter() function have words which are important and meaningful for analysis. We have applied minimum frequency of 2 as a filter to remove low frequency words whereas minimum frequency is 5 in case of bigrams by mutual information scores. The above tokens are used together in speeches many a times; hence, they are high in frequency. \
\
2. The bigrams we got from Top50bigrams_MutualInfoScore() function are not common words and are of low frequency. They give poor result when frequency is low (0-2) because of frequency filter 5. The results given by the top 50 bigrams scored by Mutual Information, are highly associative but not useful in terms of analysis.  \
\
3. Mutual Information score cannot be applied to all the bigrams(unfiltered), because the results don’t really make sense, since uniquely occurring pairs of words get high scores whereas bigrams can be applied to filtered or unfiltered words.

You can also refer to the [NLTK Book](http://www.gutenberg.org/) for  any doubts on the concepts.

