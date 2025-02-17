---
title: 'A new standard for drug repurposing by collaborative filtering: stanscofi and benchscofi'
tags:
  - Python
  - drug repurposing
  - collaborative filtering
  - open science
  - science reproducibility
authors:
  - name: Clémence Réda
    orcid: 0000-0003-3238-0258
    affiliation: 1 
  - name: Jill-Jênn Vie
    orcid: 0000-0002-9304-2220
    affiliation: 2 
  - name: Olaf Wolkenhauer
    orcid: 0000-0001-6105-2937
    affiliation: "1,3,4" 
affiliations:
 - name: Department of Systems Biology and Bioinformatics, University of Rostock, Rostock, G-18051, Germany
   index: 1
 - name: Soda Team, Inria Saclay, F-91120 Palaiseau, France
   index: 2
 - name: Leibniz-Institute for Food Systems Biology, Freising, G-85354, Germany
   index: 3
 - name: Stellenbosch Institute of Advanced Study, Wallenberg Research Centre, Stellenbosch, SA-7602, South Africa
   index: 4
date: 5 September 2023
bibliography: paper.bib
---

# Summary

As of 2023, current drug development pipelines last around ten years, costing $2.3 billion on average [@philippidis2023unbearable], while drug commercialization failure rates go up to 90% [@sun202290]. Drug repurposing might mitigate these issues by speeding up the drug discovery phase on well-documented compounds [@jarada2020review], helping to prevent adverse side effects and low accrual in clinical trials [@hingorani2019improving]. Recent papers [@zhang2017computational;@yang2019additional;@he2020hybrid;@meng2022weighted;@yang2022computational;@yang2023self] have reported near-perfect predicting power (*area under the curve*, or AUC) on several repurposing datasets by resorting to collaborative filtering approaches. Collaborative filtering straightforwardly allows the implementation of sparse classifiers which aggregate the information from many diseases. However, a considerable hurdle to developing efficient drug repurposing approaches based on collaborative filtering is the lack of a standard pipeline to train, validate and compare these algorithms on a robust dataset.

# Statement of need

The **stanscofi** Python package [@reda2023stanscofi] comprises method-agnostic training and validation procedures on several public drug repurposing datasets. Implementing these steps is crucial to avoid data leakage, *i*.*e*. the model is learnt over information that should be unavailable at prediction time. Indeed, data leakage is the source of a significant reproducibility crisis in machine learning [@roelofs2019meta;@feldman2019advantages;@kapoor2023leakage]. Our package avoids data leakage in two ways: first, by building weakly correlated training and validation sets for the drug feature vectors, and second, by implementing a generic model class, which allows the automation of the training and validation procedures.

We also propose the Python package **benchscofi**, which builds upon the former package by wrapping the original implementations of 18 drug repurposing algorithms from the state-of-the-art. This is the first time such a package enables a large-scale benchmark of collaborative filtering-based drug repurposing approaches.

The modularity of **stanscofi** and **benchscofi** at model, dataset, and preprocessing levels allows us to enrich the package with newer, more efficient approaches. Moreover, those packages allow access to several public drug repurposing datasets (see Table 1) and state-of-the-art drug repurposing algorithms (see Table 2). **stanscofi** is built around four main modules presented below.

## Module *datasets*

**stanscofi** facilitates benchmarking by allowing the import of several drug repurposing datasets (see Table 1), all under the same form: a drug-disease matrix that summarizes reported clinical trials as either "positive" (denoted by a 1, for drugs which are known to treat the corresponding disease), "negative" (indicated by a -1, for clinical trials where toxic side effects or low accrual, for instance, were reported), and "unknown" (denoted by a 0, the most occurring outcome). Some datasets also comprise drug and disease feature matrices, which bring supplementary information about drug-to-drug and disease-to-disease similarities. Moreover, one can easily convert any other drug repurposing dataset into the *Dataset* class in **stanscofi**. This package also integrates several plotting functions, allowing easier data visualization.

**Table 1:** Datasets in **stanscofi**. Reported drug and disease numbers correspond to the number of drugs and diseases involved in at least one nonzero drug-disease matching. The sparsity number is the percentage of known (positive and negative) matchings times 100 over the total number of possible drug-disease matchings (rounded to the second decimal place). The datasets are Gottlieb [@gottlieb2011predict] -- also called FDataset in [@luo2018computational] -- LRSSL [@liang2017lrssl], CDataset, DNDataset [@luo2018computational], PREDICT-Gottlieb [@gao2022dda] -- which is a version of FDataset with novel types of drug and disease features -- PREDICT [@reda2023PREDICT], and TRANSCRIPT [@reda2023TRANSCRIPT].

Dataset            |   drugs       | diseases      | positive  | negative  | sparsity
-------------------|---------------|---------------|-----------|-----------|----------
CDataset           | 663           | 409           |  2,532    |     0     | 0.93%
(nb. features)     |  (663)        | (409)         |           |           | 
DNDataset          | 550           | 360           | 1,008     |     0     | 0.01%
(nb. features)     |  (1,490)      | (4,516)       |           |           | 
Gottlieb           | 593           | 313           |  1,933    |    0      | 1.04%
(nb. features)     |  (593)        | (313)         |           |           | 
LRSSL              | 763           | 681           | 3,051     |      0    | 0.59%
(nb. features)     |  (2,049)      | (681)         |           |           | 
PREDICT            | 1,351         | 1,066         |   5,624   |  152      | 0.34%
(nb. features)     |  (6,265)      |       (2,914) |           |           | 
PREDICT-Gottlieb   | 593           | 313 (313)     | 1,933     |    0      | 1.04%
(nb. features)     |  (1,779)      | (313)         |           |           | 
TRANSCRIPT         | 204           | 116           |   401     |  11       | 0.45%
(nb. features)     |  (12,096)     |   (12,096)    |           |           | 

## Module *training/testing*

**stanscofi** implements two approaches to build training and validation sets. Along with the standard data splitting at random (function *random_simple_split*), it first proposes splitting into weakly correlated datasets (function *weakly_correlated_split*). This function is based on the hierarchical clustering of drugs based on their features, and the application of a bisection procedure to determine which cut in the dendrogram ensures that the size of the validation set is almost equal to the user-specified value (for instance, 20\% of outcomes). **stanscofi** also provides readily usable functions for cross-validation (function *cv_training*) and grid searches for hyperparameters (*grid_search*). 

## Module *models*

**stanscofi** implements a **BasicModel** class which takes as input **stanscofi** *Dataset* objects, and permits to fit (class method *fit*), to score (*predict_proba*), to label (*predict*) in a fashion which is similar to well-known Python machine learning packages such as **scikit-learn** [@pedregosa2011scikit]. However, contrary to **scikit-learn** procedures, these functions can also handle non-binary outcomes, as is often the case in collaborative filtering (with values -1, 0, and 1). Furthermore, the **BasicModel** class can also tackle recommendation-specific tasks (*e*.*g*., to recommend the top *k* drug-disease pairs with method *recommend_k_pairs*).

## Module *validation*

**stanscofi** evaluates metrics on a testing dataset through function *compute_metrics*, which can be combined with function *plot_metrics* to visualize at a glance the disease-wise Receiver Operating Characteristic (ROC) and Precision-Recall curves, a boxplot of scores obtained on the testing dataset, and the accuracy of predictions on known ratings. Computing those metrics per disease takes into account the variation in predictive power across diseases. **stanscofi** also includes other standard accuracy and ranking metrics, such as F-score, mean rank, or normalized discounted cumulative gain (globally or at a specific position).

## *benchscofi* package

Using **stanscofi**, one can test algorithms from the literature and more quickly develop a benchmark pipeline, which we demonstrated by the implementation of the **benchscofi** package. We have compiled 18 collaborative filtering algorithms from the literature in **benchscofi** [@reda2023benchscofi]. Those cover many platforms (R, MATLAB, Python) and approaches (matrix factorization, graph-based methods). We report in Table 2 some of the results obtained using **benchscofi**.

**Table 2:** Results obtained by combining **stanscofi** and **benchscofi**. Reported values are the standard *area under the curve* (AUC) scores, which are globally computed on all scores associated with drug-disease pairs. An asterisk denotes the maximum value in a column. The algorithms are ALSWR [@ethen2023ALSWR], BNNR [@yang2019drug], DDA-SKF [@gao2022dda], DRRS [@luo2018computational], Fast.ai *collab learner* [@howard2020deep], HAN [@wang2019heterogeneous], LibMF [@chin2016libmf], LogisticMF [@johnson2014logistic], LRSSL [@liang2017lrssl], MBiRW [@luo2016drug], NIMCGCN [@li2020neural], PMF [@mnih2007probabilistic], and SCPMF [@meng2021drug].

  Algorithm  (AUC)         | TRANSCRIPT        | Gottlieb      | CDataset     | LRSSL      
-------------------------- | ----------------- | ------------- | ------------ | ----------
ALSWR                      |  0.507            |  0.677        |  0.724       |  0.685   
BNNR                       |  0.922 *          |  0.949        |  0.959  *    |  0.972     
DDA-SKF                    |  0.453            |  0.544        |  0.264       |  0.542     
DRRS                       |  0.662            |  0.838        |  0.878       |  0.892  
Fast.ai collab learner     |  0.876            |  0.856        |  0.837       |  0.851     
HAN                        |  0.870            |  0.909        |  0.905       |  0.923   
LibMF                      |  0.919            |  0.892        |  0.912       |  0.873  
LogisticMF                 |  0.910            |  0.941        |  0.955       |  0.933     
LRSSL                      |  0.581            |  0.159        |  0.846       |  0.665   
MBiRW                      |  0.913            |  0.954  *     |  0.965       |  0.975  *   
NIMCGCN                    |  0.854            |  0.843        |  0.841       |  0.873      
PMF                        |  0.579            |  0.598        |  0.604       |  0.611    
SCPMF                      |  0.680            |  0.548        |  0.538       |  0.708     

All in all, **benchscofi** allows the design of large-scale benchmarks and enables a fair and comprehensive assessment of the performance of state-of-the-art methods. It will ease the development and testing of competitive drug repurposing approaches.

# Conclusion

The two packages **stanscofi** and **benchscofi** have the potential to alleviate the economic burden of drug discovery pipelines significantly. They could help to find treatments in a more sustainable manner, which still remains a topical question, especially for rare or tropical neglected diseases [@walker2021supporting]. 

# Acknowledgements

The research leading to these results has received funding from the European Union's HORIZON 2020 Programme under grant agreement no. 101102016 (RECeSS, HORIZON MSCA Postdoctoral Fellowships - European Fellowships, C.R.).

# References
