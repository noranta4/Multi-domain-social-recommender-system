# Multi domain social recommender system
Multi-domain social recommender system.

Final project of the Web and Social information extraction course, prof. Paola Velardi, prof. Giovanni Stilo.

University project • 2018 - Web and Social information extraction - MSc in Computer Science, II year

The project was developed in Jupyter, a notebook editor based on IPython. Notebook documents
are both human-readable documents containing the analysis description and the results (figures,
tables, etc..) as well as executable documents which can be run to perform data analysis.
IPython notebooks are becoming a standard in data analysis and are ideal for the tasks of this
project.

We used Google Colab to make the notebook runnable directly in a browser from any OS (Python
code runs in cloud on a virtual machine in a google server), moreover Colab guarantees read and
write access to our Google Drive where we stored the datasets in a simple and fast way. 

The project description is in `Project_Description_2018.pdf` and below.
The code and the discussion of the experiments are in the notebook `W&S_project.ipynb` ([Colab notebook](https://colab.research.google.com/drive/1QBNA4Atp6fnCV39EUGrvco93F6kh9WK6)).
The tsv files are in the `data` folder.

## Project description – Multi-domain social recommender system

Use the WikiMID dataset downloadable from [http://wikimid.tweets.di.uniroma1.it/wikimid/](http://wikimid.tweets.di.uniroma1.it/wikimid/) (use WikiMID
dataset in the form of tab separated values (TSV)). For about 500000 users (English and Italian users), it
associates a set of interests extracted from their messages or from their friendship list (by selecting those friends
in the list that indicate an **interest** more than a peer friendship relation). A Wikipedia page is associated to every
interest. Read more in the related paper. Use only the ENGLISH dataset.

1. For every user *ui* , given his/her Wikipedia-mapped interests, finds a set of CATEGORIES representing
    a synthesis of the shown preferences. In general, you should have less categories than interests
    (categories should “synthesize” a user’s main interests).

#### Example:

```
user_176236916
```
```
Interests:
WIKI:EN:American_Tabloid
WIKI:EN:Chester_Himes
WIKI:EN:The_Outsider_(Wright_novel)
WIKI:EN:Sidetracked_(novel)
WIKI:EN:The_Human_Factor_(Graham_Greene_book)
WIKI:EN:Colin_Cotterill
WIKI:EN:Whispers_Under_Ground
WIKI:EN:Adrian_McKinty
WIKI:EN:Monkey_Man_(The_Rolling_Stones_song)
WIKI:EN:Sgt._Pepper's_Lonely_Hearts_Club_Band_(song)
WIKI:EN:Band_of_the_Castle_Guards_and_the_Police_of_the_Czech_Republic
Etc.
```
```
Preferred	interest	categories :
Series
Writers
Journalists
Magazines
Politicians
Seasons
Drama
High_schools
Networks
Actors
Screenwriters
Television
Band
Companies
Etc.	
```
Note that there is no pairwise correspondence between
wikipages and semantic interests. The latter are
collectively extracted from the full set of wikipages for
each user.

You may use any semantic resource (Wikipedia categories, DBPedia, Babelnet..) and any method you
can invent or find in literature (we do not expect anything particularly innovative, so don’t worry)

2. Generate clusters of similar users (i.e. with similar interests) – using a whatever community
    detection method among those presented in class.
3. Use a whatever simple method to evaluate clusters (e.g. average distance between cluster members and
    non-cluster members: you want that any two elements in a cluster are more similar to each other than
    any two elements belonging each to a different cluster).
4. You are further given a set of 1500 Twitter IDs, file **S21.tsv**. Using Twitter API, download profile and
    friendship information, and try to associate each user _uj_ with the cluster of other users most similar to
    _uj_. Explain the adopted similarity method.
5. You are given 500 additional users (file **S22-preferences.tsv** ), for which you have the user ID and the
    list of preferred items in terms of Wikipedia pages. You are further given for each user a list of 6
    Wikipedia pages (file **S23.tsv** ). Recommend to each user 3 out of the 6 proposed items. In selecting 3
    items and discarding the other 3, you should define and implement some algorithm that ranks the 6
    items according to the “induced” user’s interests. Explain the method you use to decide which
    recommendations are more likely to fit each user’s interests.

## *Example: Proposed solution to point 1* (See the python notebook for details and the rest of the points)

### 1. Categories from interests

Extracting categories to syntethize interests using semantic is not a well defined problem, how can we measure the performance of our approach? We should make some assumptions:

1. Each interest should be semantically related to its category
2. Categorization should be balanced
3. Granularity should be appropriate: too many categories are redundant, too few categories weaken the model due to the loss of information.

The balance of the categorization is an easy measure but what about the other two criteria?

Let's see our approach to the problem.

We used [precomputed wikipage embeddings ](https://meta.wikimedia.org/wiki/Research:Wikipedia_Navigation_Vectors)as semantic model, these are obtained applying Wor2vec algorithm to reading sessions (sequence of articles visited by a user during a wikipedia session), where articles that tend to be read in close succession have similar representations. Since people usually generate sequences of semantically related articles while reading, these embeddings also capture semantic similarity between articles.
Each vector has 100 dimensions.

We select all vectors corresponding to wikipedia articles present in the WikiMID dataset (216.000 out of 1.828.000) and perform a clusterization using MiniBatch K-Means, Minibatch because of the size of the dataset, K-Means because it is more prone to generate balanced clusters and the number of clusters is tunable. However clustering suffers from the curse of dimensionality and 100 dimensions are too much, so we performed at first a dimensionality reduction with PCA, the final number of dimensions is chosen looking at the explained variance, 6 dimensions in our case.

The number of clusters is decided looking both at the silhouette coefficient (look below for more details) and at the semantic range of categories (we do not want too few clusters even if the corresponding silhouette coefficient is high, all the categories would be about music).

The final clusters represents the categories, we named each category looking at the centroid of the cluster, the name follows the standard: *Macrocategory (Centroid page name)*. Where Macrocategory refers to the category of the centroid page in Babelnet (e.g. Music, Literature and Theatre...). The simple Babelnet categorization is not good because led to a heavy unbalanced categorization, for instance Music would include more than 60% of all the pages. With our method instead we have many music categories, distinguished by the centroid page name, e.g. Music (Blind_Melon), Music (Yo_Gotti)... This partition of the categories is meaningful, for instance Music (Blind_Melon) can be associated to USA Rock, while Music (Yo_Gotti) to Rap.
