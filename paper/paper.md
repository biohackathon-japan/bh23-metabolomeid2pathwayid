---
title: 'BioHackJP 2023 Report R1: Bridging Metabolomics IDs to Pathways'
title_short: 'BioHackJP 2023 metabolomeid2pathwayid'
tags:
  - Linked Data
  - Metabolomics
  - Pathway
authors:
  - name: Kozo Nishida
    orcid: 0000-0001-8501-7319
    affiliation: Tokyo University of Agriculture and Technology
  - name: Shuya Ikeda
    orcid: 0000-0002-1357-5159
    affiliation: Database Center for Life Science
  - name: Hiromasa Ono
    orcid: 0000-0001-8675-963X
    affiliation: Database Center for Life Science
  - name: Alexander Pico
    orcid: 0000-0001-5706-2163
    affiliation: Gladstone Institutes
affiliations:
  - name: First Affiliation
    index: 1
  - name: Second Affiliation
    index: 2
date: 30 June 2023
cito-bibliography: paper.bib
event: BH23JP
biohackathon_name: "BioHackathon Japan 2023"
biohackathon_url:   "https://2023.biohackathon.org/"
biohackathon_location: "Kagawa, Japan, 2023"
group: R1
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-jp/bh23-report-template
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: First Author \emph{et al.}
---

# Background

Targeted and non-targeted metabolomic analysis using mass spectrometry is used to understand metabolic processes taking place in a wide variety of organisms, from prokaryotes, plants, and fungi to animals and humans.
Non-targeted approaches allow us to detect as many metabolites as possible at once, find significant metabolic changes among them, and perform new characterizations of metabolites in biological samples.
However, biological interpretation of large, complex data sets, such as those in non-target approaches, is difficult using metabolomics analysis alone (i.e., quantification of metabolites in samples).
The foremost approach to address that challenge would be data integration, which projects the quantitative values of the aforementioned metabolites onto a network of biochemicals (called as pathway).

Metabolome raw data are quantified using preprocessing software such as MS-DIAL [@tsugawa2020lipidome] and data for peak annotation called "library".
Most of the metadata in the library contains InChIKey [@heller2015inchi], and an information infrastructure that serves as an ID bridge between InChIKey and pathways is necessary for data integration between metabolome quantification values and pathways.

| ![Figure 1. Graphical Abstract. In this section we will be presenting the id mapping difficulty between metabolome and pathway to interpret metabolomics data.](graphical_abstract.png) | 
|:--:| 
| *Figure 1. Graphical Abstract. In this section we will be presenting the id mapping difficulty between metabolome and pathway to interpret metabolomics data.* |

WikiPathways [@martens2021wikipathways] is one of the major pathway databases and has a Cytoscape [@shannon2003cytoscape] App [@kutmon2014wikipathways],
which can import the pathway into Cytoscape and help visualization and interpretation of the omics data on the pathway.
However, the data resource for metabolites in WikiPathways is not limited to one, and multiple IDs may be mixed in one pathway.
Furthermore, InChIKey is not the main ID.

```
select str(?datasource) as ?source count(distinct ?identifier) as ?count
where {
  ?mb a wp:Metabolite ;
    dc:source ?datasource ;
    dc:identifier ?identifier .
} order by desc(?count)
```

![image](https://github.com/biohackathon-japan/bh23-metabolomeid2pathwayid/assets/12192/ffd29f0a-ee24-4a26-8a4e-3f5a46337ed6)

Therefore, a system to align the IDs of scattered namespaces in a pathway into InChIKey is needed for Metabolomics.
It is possible to achieve the ID alignment in WikiPathways SPARQL endpoint (https://sparql.wikipathways.org/) by using a federated query, but at present the query is difficult to write and slow performance.
There is also an id conversion system called BridgeDB [@van2010bridgedb] in the WikiPathways ecosystem, but the BridgeDB is not involved the WikiPathways SPARQL endpoint.
In addition, there are services that convert metabolite IDs such as CTS [@wohlgemuth2010chemical] , Metaboanalyst [@pang2021metaboanalyst] Metabolite ID Conversion and MetaCyc [@caspi2020metacyc] Metabolite Translation Service, but they are not open and are not suitable for system integration that seamlessly compensates for the missing functionality in WikiPathways.

TogoID is an ID conversion service developed by Database Center for Life Science (DBCLS). It covers a broad range of life science databases of various categories such as genes, proteins, compounds, diseases, pathways, and more. TogoID not only enables the conversion of IDs between entities with equivalent meanings but also provides associations with various biological meanings, such as "compounds belonging to a pathway." One of the unique features of TogoID is its multi-step conversion capability, allowing users to convert IDs to target IDs via other types of IDs. TogoID offers a "NAVIGATE" mode, which displays possible paths between input IDs and target IDs.

Prior to this hackathon, only ChEBI, ChEMBL compound, and PubChem compound were directly linked to InChIKey in TogoID. With TogoID's multi-step conversion functionality, it was possible to convert InChIKey to HMDB via ChEMBL Compound. However, this approach resulted in the loss of HMDB entries that lacked corresponding ChEMBL Compound entries. In fact, out of the 217,897 entries in HMDB v5.0, only 6,040 could be mapped to ChEMBL Compound entries.
In this hackathon, we aimed to obtain the pair between HMDB and InChIKey from the data provided by HMDB and add it to TogoID, ensuring that all HMDB entries can be converted to and from InChIKey. Furthermore, additional links between HMDB and other relevant databases were incorporated into TogoID. Additionally, links to other databases related to metabolomics analysis were included in TogoID to enhance the database relationships and provide a foundation for analysis.
The NAVIGATE mode can be helpful for WikiPathways users. If a user intends to retrieve all compounds belonging to a pathway entry and to output as InChIKey, the user can input WikiPathways IDs to TogoID and specify InChIKey as the conversion destination. Regardless of the ID under which the compound is represented in WikiPathways, the user can get all the paths through them to be converted to InChIKey. Before the hackathon, TogoID had only UniProt and ChEBI as the IDs for components within WikiPathways. To expand its coverage, additional commonly used IDs in WikiPathways for representing pathway components were incorporated into TogoID.

# Outcomes

During the BioHackathon 2023, we added the pairs shown in Table 1 to TogoID.

Table 1
|Source|Target|
|--|--|
|WikiPathways | HMDB, NCBI Gene, LIPID MAPS|
|HMDB | InChIKey, ChEBI, PDB CCD, PubChem Compound|
|PubChem Pathway | Reactome Pathway, Pathbank, WikiPathways|
|SwissLipids | InChIKey, UniProt |
|LIPID MAPS | ChEBI, InChIKey,  SwissLipids|

The direct conversion between WikiPathways and NCBI Gene, HMDB, and LIPID MAPS became available on TogoID. Also, InChIKey pairs with HMDB, SwissLipids, and LIPID MAPS were added.
These enable users to obtain one specific type of ID belonging to pathway entries via different paths by the NAVIGATE mode of TogoID. For example, when a user inputs WikiPathways IDs and select InChIKey as the conversion destination, paths of WikiPathways - ChEBI - InChIKey, WikiPathways - HMDB - InChIKey, WikiPathways - LIPID MAPS - InChIKey are displayed and the user can download each conversion result.

We also added pairs between SwissLipids and UniProt. This relationship is between lipids and enzymes that metabolize the lipids.

![Caption for BioHackrXiv logo figure](./biohackrxiv.png)

# Future work



## Acknowledgements

We would like to thank the fellow participants at BioHackathon 2023 for their collaboration and constructive advice, which greatly influenced our project. We are grateful to the organizers for providing this platform and the developers of open source language models. Special thanks to our mentors, advisors, and colleagues for their guidance and support. Without their contributions, our project in linked data standardization with LLMs in bioinformatics would not have been possible.

## References

1. This will be automatically generated.
