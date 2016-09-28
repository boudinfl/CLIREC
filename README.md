# CLinical Information Retrieval Evaluation Collection (CLIREC)

Version 1.0 - July 2010

This archive contains three files:

1. clirec.queries.xml : XML-formatted file containing the queries

	Given this example :

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


2. collection.pmids: file containing the PMIDs of the collection of documents
	we used in our experiments. You can use this list of 1,212,042 identifiers to
	gather your collection of documents or alternatively reconduct the search 
	using the following PubMed query:

	hasabstract[text] AND "humans"[MeSH Terms] AND (Clinical Trial[ptyp] OR 
	Editorial[ptyp] OR Letter[ptyp] OR Meta-Analysis[ptyp] OR Practice 
	Guideline[ptyp] OR Randomized Controlled Trial[ptyp] OR Review[ptyp]) AND 
	English[lang]

3. clirec.qrels : relevance judgments (TREC-Like formatted)

	Given this example :

	A10.1 1991 1724669 1
	A10.1 1998 9540427 1
	A10.1 1998 6130330 1
	A10.1 2006 16616232 1
	...

	This first column is the query identifier, the second is the publication 
	date of the relevant document and the third its PMID.