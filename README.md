# CLinical Information Retrieval Evaluation Collection (CLIREC)

CLIREC is a test collection for evaluating clinical information retrieval built from systematic
reviews whose purpose is to provide a basis for researchers to experiment with PICO-structured
queries. It contains 1.2M documents collected from PubMed and a set of manually constructed
queries and their corresponding relevant documents (qrels).

## Contents of this repository

Overall structure of the repository

```
|-- collection/           // document collection in jsonl format
  |-- clirec.aa.jsonl.gz  // {"id": "...", "title": "...", "contents": "...", "year": "..."}
  |-- clirec.ab.jsonl.gz  //   
  |-- ...                 //   
  |-- clirec.al.json.gz   //
|-- clirec.queries.xml           // XML-formatted file containing the queries
|-- clirec.queries.keywords.tsv  // keyword queries in tab-separated format
|-- clirec.qrels                 // relevance judgments (TREC-Like formatted)
|-- collection.pmids             // list of pmids for the document collection
```

PICO queries are encoded in an XML-formatted file as shown below:

```
<queries>
  <query id="1" before="20060901" keywords="Barrett's oesophagus treatment">
    <pop>Adults</pop> 
    <prob>Barrett's oesophagus</prob> 
    <int>Pharmacological treatment</int>
    <comp>Non-resectional surgical treatment</comp>
    <out>Complete eradication of dysplasia</out>
    <dur>12 months</dur>
  </query>
  ...
</queries>

* <queries> is the root element
  * Each clinical query is contained in a <query> element

  It has 3 attributes:
    - 'id' is the query unique identifier
    - 'before' is the publication date of the Cochrane review
    - 'keywords' is the corresponding keywords query

  And up to 6 children:
    - <pop> is the population/patient query part	
    - <prob> is the problem query part
    - <int> is the intervention/exposure query part
    - <comp> is the comparison query part
    - <out> is the clinical outcome query part
    - <dur> is the duration query part
```

The document collection contains 1211820 PubMed documents and was collected 
using the following query:

```
hasabstract[text] AND "humans"[MeSH Terms] AND (Clinical Trial[ptyp] OR 
Editorial[ptyp] OR Letter[ptyp] OR Meta-Analysis[ptyp] OR Practice 
Guideline[ptyp] OR Randomized Controlled Trial[ptyp] OR Review[ptyp]) AND 
English[lang]
```

Relevance judgments are in TREC-Like format and were manually mapped to their
PubMed unique IDentifiers (PMID). 

```
A10.1 1991 1724669 1
A10.1 1998 9540427 1
A10.1 1998 6130330 1
A10.1 2006 16616232 1
...
This first column is the query identifier, the second is the publication 
date of the relevant document and the third its PMID.
```

## Retrieval experiments with [anserini](https://github.com/castorini/anserini)

### Indexing the collection

```bash
sh anserini/target/appassembler/bin/IndexCollection \
	-collection JsonCollection \
	-threads 4 \
	-input collection/ \
	-index exp/lucene-index.clirec/ \
	-optimize \
	-storePositions -storeDocvectors -storeRaw -fields title year
```

### Retrieving documents

```bash
sh anserini/target/appassembler/bin/SearchCollection \
	-index exp/lucene-index.clirec/ \
	-topics clirec.queries.keywords.tsv \
	-topicreader TsvString \
	-output exp/clirec.queries.keywords.bm25.out \
	-bm25 -hits 1000 -fields contents=1.0 title=1.0

sh anserini/target/appassembler/bin/SearchCollection \
	-index exp/lucene-index.clirec/ \
	-topics clirec.queries.keywords.tsv \
	-topicreader TsvString \
	-output exp/clirec.queries.keywords.bm25+rm3.out \
	-bm25 -rm3 -hits 1000 -fields contents=1.0 title=1.0
	
sh anserini/target/appassembler/bin/SearchCollection \
	-index exp/lucene-index.clirec/ \
	-topics clirec.queries.pico.tsv \
	-topicreader TsvString \
	-output exp/clirec.queries.pico.bm25.out \
	-bm25 -hits 1000 -fields contents=1.0 title=1.0

sh anserini/target/appassembler/bin/SearchCollection \
	-index exp/lucene-index.clirec/ \
	-topics clirec.queries.pico.tsv \
	-topicreader TsvString \
	-output exp/clirec.queries.pico.bm25+rm3.out \
	-bm25 -rm3 -hits 1000 -fields contents=1.0 title=1.0
```

### Evaluating the retrieval effectiveness

```bash
anserini/tools/eval/trec_eval.9.0.4/trec_eval -m official \
 	clirec.qrels exp/clirec.queries.keywords.bm25.out
anserini/tools/eval/trec_eval.9.0.4/trec_eval -m official \
 	clirec.qrels exp/clirec.queries.keywords.bm25+rm3.out
anserini/tools/eval/trec_eval.9.0.4/trec_eval -m official \
 	clirec.qrels exp/clirec.queries.pico.bm25.out
anserini/tools/eval/trec_eval.9.0.4/trec_eval -m official \
 	clirec.qrels exp/clirec.queries.pico.bm25+rm3.out
```

### Baseline results

#### Keywords queries

| Model     | mAP   | P@5   | P@10  | num_rel_ret |
|-----------|-------|-------|-------|-------|
| BM25      | 12.00 | 16.13 | 13.61 | 1512 |
| BM25+RM3  | 13.14 | 17.03 | 14.32 | 1604 |

#### PICO queries (without using any specific weighting)

| Model     | mAP   | P@5   | P@10  | num_rel_ret |
|-----------|-------|-------|-------|-------|
| BM25      | 14.64 | 21.65 | 17.64 | 5651 |
| BM25+RM3  | 13.97 | 19.10 | 15.79 | 6174 |

## Release history

- Version 1.1 - May 2022 (adding collection of documents and model benchmarking)
- Version 1.0 - July 2010 (initial release)

## Citation

If you use this dataset, please cite the following articles:

 - **[Positional Language Models for Clinical Information Retrieval](http://aclweb.org/anthology/D10-1011).**
   Florian Boudin, Jian-Yun Nie, Martin Dawes.
   *Conference on Empirical Methods in Natural Language Processing (EMNLP), 2010.*

 - **[Clinical Information Retrieval using Document and PICO Structure](http://www.aclweb.org/anthology/N10-1124).**
   Florian Boudin, Jian-Yun Nie, Martin Dawes.
   *Conference of the North American Chapter of the Association for Computational Linguistics (NAACL), 2010.*

 - **[Deriving a test collection for clinical information retrieval from systematic reviews](http://doi.acm.org/10.1145/1871871.1871882).**
   Florian Boudin, Jian-Yun Nie, Martin Dawes.
   *Data and Text Mining in Biomedical Informatics (DTMBIO), 2010.*
