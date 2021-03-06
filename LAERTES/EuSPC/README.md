European Union Adverse Drug Reactions from Summary of Product Characteristics (EU SPC) Database Import 
=============================================================================================

The EU SPC source provides adverse drug events extracted from EU SPCs by the [PROTECT](http://www.imi-protect.eu/about.shtml) collaborative. See the online documentation for the [PROTECT Adverse Drug Reactions Database](http://www.imi-protect.eu/adverseDrugReactions.shtml) for a liability disclaimer.

### Convert EU SPC data to RDF to support the OHDSI KB use cases

- [euSPC2rdf.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/euSPC2rdf.py) : Accepts as input a tab delimitted file containing adverse drug events extracted from EU SPCs by [PROTECT](http://www.imi-protect.eu/adverseDrugReactions.shtml) that has been processed to add concept identifiers for the drugs from RxNorm and MeSH. The script produces as output a ntriples file that represents the data using the Open Annotation Data (OA) schema

NOTE: an editable diagram of the OA model for EU SPC ADE records can be found in [Schema/OpenAnnotationSchemaERDiagrams/](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/Schema/OpenAnnotationSchemaERDiagrams)

NOTE: The output of this script is too large to load into virtuoso through the web interface. See [below](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC#loading-the-rdf-data-into-virtuoso) for instructions on loading the dataset using isql-vt

- [writeLoadableEUSPCcounts.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/writeLoadableEUSPCcounts.py) : Once the RDF output of the
[euSPC2rdf.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/euSPC2rdf.py) script is loaded into a virtuoso endpoint, a query in the
following form is ran at the endpoint (see
[RDF-count-and-drill-down-queries.sparql](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/Schema/RDF-count-and-drill-down-queries.sparql) or the comments in [writeLoadableEUSPCcounts.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/writeLoadableEUSPCcounts.py)): 

```
SELECT count(distinct ?an) ?drug ?hoi
...
```

This query give the counts for records present in the EU SPC for all drugs and HOIs present in the database. This data is saved into a tab delimitted file (most recently [test-query-of-counts-05052015.csv](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/test-query-of-counts-05052015.csv), but historically test-query-of-counts-10272014.csv) that is loaded by [writeLoadableEUSPCcounts.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/writeLoadableEUSPCcounts.py). The script then generates a file (most recently [drug-hoi-counts-with-linkouts-EU-SPC-05052015.tsv](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/drug-hoi-counts-with-linkouts-EU-SPC-05052015.tsv), but historically drug-hoi-counts-with-linkouts-EU-SPC-10272014.tsv) that can be loaded into the relational Schema table 'drug_hoi_evidence'. An example record:

```
drug_hoi_relationship	evidence_type	modality	evidence_source_code_id	statistic_value	evidence_linkout	statistic_type
757688-35708164 SPL_EU_SPC      positive        1       2       http://dbmi-icode-01.dbmi.pitt.edu/l/index.php?id=kza   COUNT
```

The data in this file can then be used 


### Generating the table of EU SPC data:

The scripts in this folder add RxCUIs and MeSH CUIs to the EU SPC drug list. In most cases, there was a simple string match between an entry in the 'substance' column with an RxNorm or MeSH preferred term. However, there are many cases of combination products, and a few things that were unable to be mapped. The combination products were manually mapped where possibly using the Bioportal's ontology search.

The final dataset with the RxCUIs and MeSH CUIs is in the data folder with a folder name like <EU SPC drug-HOI data file>_withCUIs_v1.csv

1. `cd scripts`
2. `python processEuSPCToAddRxNormAndMeSH.py`
3. `python3 getMissingMappings.py ../data/<EU SPC drug-HOI data file>_withCUIs_v1.csv ../data/missing`

### Processing the EU SPC Drug Listing

The **euspc-drug-listing** holds drug names for single ingredient drugs
listed in the adverse event table. This was created by:

1. `cat FinalRepository_DLP30Jun2012.csv| cut -f3 | sort | uniq > euspc-drug-listing.txt`
2. within emacs:
`replace-regexp  ".*, .*^J" -> ""`


### Subfolders
1. [json-rxcui](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC/json-rxcui)
	- Contains the raw output from the [TRIADs drug named entity recognition program](https://swat-4-med-safety.googlecode.com/svn/trunk/u-of-pitt-SPL-drug-NER) used to input one of the big files.
2. [data/missing](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC/data/missing)
	- contains the missing CUIs from the second run of [processEuSPCToAddRxNormAndMeSH.py](https://github.com/OHDSI/KnowledgeBase/blob/master/LAERTES/EuSPC/scripts/processEuSPCToAddRxNormAndMeSH.py)
3. [backup/missingCUIs](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC/backup/missingCUIs)
	- A backup folder containing the output missing CUIs from the first run of [processEuSPCToAddRxNormAndMeSH.py](https://github.com/OHDSI/KnowledgeBase/blob/d2af5e16c2b6f05d59664b93457f90f90da83dea/EuSPC/processEuSPCToAddRxNormAndMeSH.py) using the first [getMissingMappings.py](https://github.com/OHDSI/KnowledgeBase/blob/d933222eca84247c7dcbcc03d203141fb3d98198/EuSPC/getMissingMappings.py) from commit [90f0a68842428c222b9606c05d2d5f129cee7ca4](https://github.com/OHDSI/KnowledgeBase/commit/90f0a68842428c222b9606c05d2d5f129cee7ca4)
	- Also contains the missing CUIs from the first run manually searched in [Bioportal](http://bioportal.bioontology.org/search?opt=advanced)
4. [scripts](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC/scripts)
	- location of the scripts file to run
5. [data](https://github.com/OHDSI/KnowledgeBase/tree/master/LAERTES/EuSPC/data)
	- location of the scripts output files


------------------------------------------------------------

### LOADING THE RDF DATA INTO VIRTUOSO:
```
-- FIRST TIME ONLY 
-- MAKE SURE THAT THE PATH WHERE THE DATA FILE RESIDES IS IN THE DirsAllowed LIST OF virtuoso.ini AND RESTART VIRTUOSO
$ INSERT INTO DB.DBA.load_list (ll_file,ll_graph) values('<path to drug-hoi-eu-spc.nt>', 'http://purl.org/net/nlprepository/ohdsi-adr-eu-spc-poc');
-- END OF FIRST TIME ONLY

$ select * from DB.DBA.load_list
-- IF LL_STATE = 0 THEN THE DATASET IS READY TO LOAD

$ rdf_loader_run();

-- ELSE, CLEAR THE GRAPH AND THE SET LL_STATE TO 0

$ SPARQL CLEAR GRAPH 'http://purl.org/net/nlprepository/ohdsi-adr-eu-spc-poc' ;
$ UPDATE DB.DBA.load_list SET ll_state = 0 WHERE ll_graph = 'http://purl.org/net/nlprepository/ohdsi-adr-eu-spc-poc';
$ rdf_loader_run();
```