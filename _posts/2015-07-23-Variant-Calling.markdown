---
layout: post
title:  "Variant Calling"
createdate:   2015-07-23
lastupdate:   2015-07-23
categories: "bioinformatique"
---

# Introduction

# Les différents types de polymorphisme

## SNP ou SNV

## Insertions / Déletions

Les insertions/déletions sont communément désignées ensemble par le terme d'INDELs. Quand on parle d'INDELs, on désigne donc à la fois les évenements d'insertions et de déletions. (le terme semble avoir été inventé par Kruskal ..à voir avec wikipedia : Sankoff et Kruskal, Time warps, string edits, and macromolecules: The theory and practice of sequence comparaison., Addison-Wesley,‎ 1983 (ISBN 0201078090) Les INDELs sont souvents divisées en 2 types, les courtes et les longues. Cela s'explique notamment par le type de données et les traitements nécessaires pour révéler chacun des 2 types. 

### InDels courts

### InDels longs

Les InDels longs sont plutôt assimilés à ce que l'on appelle des variants structuraux (Structural Variants, SV). 
blbala...
i
Read this review paper from 2011 on Structural variant calling:

http://www.ncbi.nlm.nih.gov/pubmed/21358748

Basicly there are 3 signals you can use for structural variant calling:

1) discordant pair signal

2) readdepth signal

3) split read signal or ( with as a special case denovo assembly split contig mapping signal)

The discordant pair signal and readdepth signal you can get from paired sequencing data produced on all platforms. To use the
 split read signal you nead long reads and do split mapping of these reads, this is not really usefull on solid data or other
 short read sequences.

A good discordant pair signal SV caller is breakdancer.

A good split read signal SV caller is pindel.

A good readdepth signal SV caller is cnvator.

2 upcomping multisignal SV callers are SVMiner and Lumpy




## Duplications


# Outils

Il existe de nombreux outils permettant la detection de polymorphisme, avec différentes algorithmes implémentés. La détéction de variant se fait en 2 étapes: l'assignation de génotype, puis l'identification d'une variation par rapport à la référence.

* samtools
* GATK
* Freebayes ([Garrison *et al*, 2012](#Garrison-2012))
* Platypus ([Rimmer *et al*, 2014](#Rimmer-2014))
* popoolation
* Atlas2
* glfTools
* MAQ
* SOAPSnp
* SeqEM
* SNPSVM

voir publi Comparison of variant Callers -Liu ... tableau sur algo des outils

Il existe un autre types d'outils basés sur de l'assemblage ....

* DiscoSNP
* Kissplice
* Platypus ([Rimmer *et al*, 2014](#Rimmer-2014))
....

# Protocole d'analyse

L'analyse de variant est influencée par plusieurs variables que sont le type de données séquencées (whole-genome, exome), la nature de l'échantillon, la technologie de séquençage et les logiciels d'analyses bioinformatiques. 

## Mapping

Le mapping est une étape très importante de l'analyse de variant. Le choix du mapper est un élement important à prendre en compte et peut être plus important dans certaines analyses que d'autres. En effet, l'algorithme de l'outil peut impacter la détection de variant de 2 façons ([Highnam *et al.*, 2015](#Highnam-2015)): 

* mauvais positionnement de la lecture sur le génome
* alignement local incorrecte la lecture autour des zones complexes ou des indels

A la vue des erreurs possibles, on comprend facilement que l'algorithme de mapping est lui aussi influencé par le type de données analysées. Ainsi, en fonction de la structure du génome de réference ou des génomes séquencés, la richesse en régions de répétées (Elements transposables, micro-macro-satellites, zones de faible complexité) peut fortemment impacter les résultats. 

## Détection de variant

## Filtrage des variants

Le but de l'étape de filtrage va être de maximiser le nombre de variants retenus selon leurs utilisations futures et les seuils de tolérance au erreurs fixés. Par exemple, la recherche de variant rare peut s'avérer complexe surtout si cela sont peu soutenu (nombre de reads, ...). En revanche, l'utilisation de variants comme marqueurs à des fins de cartographie, peut autoriser un filtre fort des données pour ne conserver qu'un jeu très sûr.

Filtre des régions comme LC, repeats ([Li, 2014](#Li-2014))

### Les biais

Les erreurs de détection des variants sont influencés par l'outil de calling utilisé (algo, ..). Mais il y a aussi d'autres types de perturbations due à l'échantillon analysé, au séquençage, au protocole d'analyse... Il y a un ensemble de biais à prendre en compte lors de l'analyse qu'il faut essayer de corriger au moment le plus propice ou alors simplement accépté comme tel en proposant des niveaux de confiance plus ou moins fiable sur les variants trouvés. Plusieurs études on permit de révéler les biais associés aux analyses de polymorphisme ([Meynert *et al*, 2014](#Meynert-2014)), ...

Dans le papier de ([Meynert *et al*, 2014](#Meynert-2014)), il est mis en évidence que l'uniformité en couverture est un critère important pour la détection des variants, et plus spécifiquement ceux hétérozygote. Pour cela il compare des analyses d'exome-seq dont la couverture peut être très variable, à des données de génome entier. A couverture plus faible, les données de génomes sont plus performantes car ayant une profondeur de couverture assez homogène. La profondeur de séquençage est un élément important qui peut doit être réflechie selon l'étude réalisée. Il est tout à fait possible de faire des analyse de génome de population d'individu à très faible couverture quand on combine cela à du génotypage type multi-échantillon ([Sims *et al*, 2014](#Sims-2014)). La puissance du nombre d'échantillon contre-balance la faible couverture. 

Par ailleurs, la faible couverture de certaines zones peut être directement liée à la composition de la séquence. Les zones riche en GC induisent des biais dans les réactions de PCR ([Kozarewa *et al*, 2009](#Kozarewa-2009), [Aird *et al*, 2011](#Aird-2011), [Veal *et al*, 2012](#Veal-2012)), conduisant à des variation de couverture. Les kits de préparation de librairie sans réaction de PCR peuvent réduire de façon très importante ce problème. [Kozarewa *et al*, 2009](#Kozarewa-2009) ont mis en évidence ce biais d'enrichissement des zones GC par rapport aux zones AT, en évaluant les produits de séquençage de librairies avec ou sans PCR. Ils ont utilisé 3 types de génomes : un très pauvre en GC, *Plamodium falciparum*, agent du paludisme (composition moyenne des exons > 75% AT et régions intergenic proche de 100% AT), un neutre en GC, *Escherichia coli* et un riche en GC, *Bordetella pertussis*, bactérie responsable de la coqueluche (contenu moyen en GC > 68%). Les résultats montre le biais très important qu'il y a sur le génome de *Plasmodium* en comparant la profondeur de couverture de chaque base du génome (écart important par rapport à la [loi de Poisson][loi-de-Poisson] attendue) et le contenu en GC des lecture. En revanche le biais est beaucoup plus modéré pour les génomes avec un taux de GC standard.  [Aird *et al*, 2011](#Aird-2011) ont travaillé sur l'amélioration des protocoles de PCR pour proposer des astuces permettant d'éviter les biais du à la composition en GC. Pour cela, ils ont testé différents protocole et mesure l'abondance des fragments par qPCR. Au final, ils arrivent à minimiser les biais sauf pour les zones très riches en GC, mal amplifiées. [Veal *et al*, 2012](#Veal-2012) ont montré que certaines zones étaient difficilement amplifiées à cause de leur proximité avec des zones riches en GC, type ilot CpG. En fonction de la méthode de fragmentation de l'ADB, ces régions sont embarqués dans les fragments et sont des zones qui bloquent la dénaturation de l'ADN ou favorisent sa reconstitution en double brin. 


Il peut aussi être corrigé informatiquement par les outils de déduplication de mapping de lecture (papier).

### Format de stockage

Le format le plus couramment utilisé est le VCF pour Variant Calling Format.

### Outils de filtre


# Benchmarks / Comparatifs

Il existe beaucoup de comparatifs de performance des analyses de variants, mais ceux-ci s'appuient généralement sur les mêmes types de données (set NA ... humains). Le manque de genome complet parfaitement séquencés rend difficile la comparaison des résultats. De même, comme nous discuté plus haut, il y a différentes variables qui influent sur l'analyse. Il est donc très difficile de comparé la performance des outils de calling, sans prendre en compte l'outil de mapping utilisé en amont par exemple.

* Une revue de l'ensemble des outils à utiliser pour faire de l'analyse de données NGS ([Pabinger *et al.*, 2013](#Pabinger-2013)). Ils indiquent qu'ils ont testé plus de 205 outils, dont 32 pour l'aspect détection de variant. Les résultats de seulement 5 outils sont présentés (CRISP, GATK, Samtools, SNVer, Varscan 2). Pas de conclusion intéressante, il n'y a rien de très marquant. On ne retire pas grand chose de l'article, qui fait un catalogue d'outil. Il n'y a rien sur l'aspect filtre des données. Pour leur classification d'outil, ils établissent 4 catégories : les germline callers (mutation héréditaires, transmises), les somatic callers (mutation acquise, cancer ...), les outils de détection de CNV et ceux de SV.
* Test de 4 variant caller (Samtools, GATK, glftools, Atlas 2) avec le même mapper (BWA)([Liu *et al.*, 2013](#Liu-2013)). Les outils sont aussi testé sur leur capacité à faire du traitement simple échantillon vs multiple échantillon. Les données utilisées sont des données simulées génome entier à différentes couvertures ou des données d'exomes de 20 individus. Il y a des validation et comparaison avec des données de puces ou des séquençage Sanger. Il y a des comparatifs TP, FP, sensibilité, spécificité, ...([sensibilité et de précision][sen-spe].). L'outil le plus performant semble être GATK. L'approche multi-échantillon lorsqu'elle possible est plus sensible, mais induit plus de faux positifs. Dans le cas des Samtools, la profondeur de reads étant limitée à 8000, ceci pénalise l'approche multi-échnatillon (notamment sur les exomes) qui devient moins performantes. Le filtrage des résultats est aussi important à réaliser pour améliorer la précision. Par ailleurs, il est parfois difficile de comparer les InDels détecté par différents outils, car ils peuvent êre écrit de manière différentes (ex : TAAAA/TAAA pour Samtools, ou TA/T pour GATK). 
* ([Pirooznia *et al.*, 2014](#Pirooznia-2014))
* Comparatif de 4 outils de détection de variant dans le cadre d'analyse de 65 génomes de bovins liés à la race laitière suisse ([Baes *et al.*, 2014](#Baes-2014)). Le but est d'avoir un comparatif sur un génome non finalisé et moins bien annoté que l'humain. Les outils testés sont Platypus, Samtools, GATK HC et UG. Les outils sont testés en mode simple et multi-échantillons. Les différentes étapes sont évalués pour leur gains sur l'analyse vs le temps de calcul. Cela révèle notamment que la recalibration et le réalignment sont couteux et pas forcément utiles vu le peu d'information disponibles (indel, snp de référence). En revanchel'approche multi-échantillon est intéressante pour la detction des indels, car dans ce cas les indels homozygote comme la réference sont détectés et permettent d'amliorer la qualité des variants.
* Test intéressant sur multi-echantillon avec GATK .... à compléter ([Nho *et al*, 2014](#Nho-2014))
* Comparatif de pipeline d'analyses (30) avec toutes les combinaisons entre 6 mappers (Bowtie 2, BWA sampe, BWA mem, CUSHAW3, MOSAIK, Novoalign) et 5 genotypers (Freebayes, GATK HC, GATK UG, Samtools, SNPSVM) ([Cornish *et al.*, 2014](#Cornish-2014)). Les données de tests sont humaines sur le genomehg19 avec le très connu exome issu d'un génome féminin du projet 1000 génomes (NA12878). Résultats présentés sous formes d'analyse de TP, FP avec les valeurs de [sensibilité et de précision][sen-spe]. La combinaison gagnante semble être Novoalign et GATK UG sur leurs critères. Les grandes tendances sont que Freebayes détecte beaucoup de SNP (TP), mais a aussi bcp de FP et que SNPSVM ne marche pas très bien, sûrement du à son algorithme qui est basé sur les SVM. Un pré-traitement a été réalisé avant chaque genotyper (filtre, realignement, recalibration), alors que certains outils comme Freebayes sont pensés pour faire leur propre correction, il y a un impact potentiel sur les résultats. Au final, le nombre de variant trouvés par tous les pipeline est bien en dessous du nombre attendu par rapport aux données du génome entier. Une analyse whole-genome à faible couverture est plus puissante que une analyse exome avec haute couverture, car la profondeur des reads est plus homogène.
* [GCAT][GCAT], the Genome Comparison and Analytic Testing platform ([Highnam *et al.*, 2015](#Highnam-2015)) est un outil web permettant de présenter et comparer des analyses de polymorphisme. Très riche en graphique, il présente des analyses avec différents mappers (BWA, Bowtie, Novoalign) et variant callers (Freebayes, GATK, Samtools). Les résultats présentés mettent en évidence que l'amélioration des algorithme de mapping améliorent la puissance de détection de polymorphisme et que les benchmarking réalisés sur des données de types exomes sur-évaluent les performances sur génome entier.
* [bcbio-nextgen][bcbio-nextgen] est un framework d'analyse de variants (SNV, structuraux) issu des tests réalisés par B.Chapman et ses collaborateurs. L'intérêt est que les outils disponibles ont été comparés les uns aux autres et que les résultats sont disponibles et discutés sur le blog [bcbio-nextgen][bcbio-nextgen]. Le [premier test réalisé en 2012][bcbio-nextgen-1] comparait Freebayes, GATK et samtools après un mapping avec Novoalign. Il mettait en évidence les grandes différences de résultats obtenus entre les outils. Ils ont par la suite rajouté au test [des outils d'assemblage local (Cortex) pour des approches de détection hybrides et développé un système de classification basé sur SVM (Support Vector Machine)][bcbio-nextgen-2] pour la comparaison des résultats. Ils ont ainsi amélioré les critères de comparaison en entrainant le classifieur SVM sur la localisation des SNPs (fin des reads, fréquences alleliques autre que 100% homo ou 50% hétérozygote). Leur travail suivant a porté sur [la combinaison des résultats des différents variant callers][bcbio-nextgen-3]. Ils ont réalisé des tests en travaillant sur des réplicats de données. Les résultats ont mis en avant GATK. Samtools montre de bon résultats pour les SNPs, en revanche Freebayes est performant pour la détection des InDels. Un autre [post][bcbio-nextgen-4] intéressant sur la réduction du nombre de valeur de qualité possible d'une base séquencée (dans le but de réduire la taille du fichier de données brutes compressé) montre le faible impacte sur la détection des SNPs sauf dans les zones de faible couverture. Il y a aussi des posts sur des aspects très pratiques comme [la parallelisation des traitements][bcbio-nextgen-5], sur l'utilisation de [Docker pour les installaltions d'outils][bcbio-nextgen-6], ou encore l'application de [filtre sur les SNPs trouvés][bcbio-nextgen-7]. En parallèle, ils continuent de faire évoluer leur comparatif d'outil en testant régulièrement les nouvelles versions. Ils ont ainsi testé [plusieurs pipelines d'analyse][bcbio-nextgen-8] avec les mappers bwa-mem ou Novoalign, avec des prétraitement par Markduplicate de Picard + recalibration GATK ou rmdup (samtools) + ogap. Le tout étant ensuite soumis aux différents callers. Encore une fois l'analyse des résultats a montré que les régions qui divergent sont celles avec une faible couverture (les résultats ont été mis à jour dans un [post plus récent][bcbio-nextgen-9]). Pour finir, ils ont aussi comparé la puissance des différents callers dans des [analyses de type multi-échantillon][bcbio-nextgen-10]. Ils ont notamment testé la procédure d'analyse incrémentale proposée par GATK via le format gVCF (toutes les bases) afin d'éviter un temps de calcul trop important. Il y a une amélioration des résultats vis à vis de l'analyse simple échantillon. Le résultat le plus flagrant est pour samtools (nouvelle version), mais il y a aussi beaucoup de faux positifs. Dans les outils disponibles via leur pipeline, il y a aussi l'outil [VarDict][vardict] ou [VarDictJava][vardictjava] (réimplementation en Java) qui ont été développés par AstraZeneca. 

Si on essaye de résumer les résultats de ces différentes analyses. On retrouve l'importance du mapping. Plus l'algo est performant, meilleur le caller sera (biais) => Highnam. De même le type de données est important ... meilleurs resultats sur exome vs genome => Highnam. Les différents comparatifs ont été fait sur des versions des logiciels qui ont été mis à jour et améliorés. Il y a donc parfois de grandes différences avec les version actuelles.
En gros, tous mettent en évidence les mêmes choses ... des biais connus et inhérent aux méthodes....

# Réferences

## Publications

* <a name="Aird-2011"></a>Aird, D. et al. Analyzing and minimizing PCR amplification bias in Illumina sequencing libraries. Genome Biol. 12, R18 (2011).
* <a name="Baes-2014"></a>Baes, C. F. et al. Evaluation of variant identification methods for whole genome sequencing data in dairy cattle. BMC Genomics 15, 948 (2014).
* <a name="Cornish-2014"></a>Cornish, A. & Guda, C. A Comparison of Variant Calling Pipelines Using Genome in a Bottle as a Reference. Biomed Res. Int. at 
* <a name="Garrison-2012"></a> Garrison, E. & Marth, G. Haplotype-based variant detection from short-read sequencing. arxiv.org 9 (2012). at <http://arxiv.org/abs/1207.3907>
* <a name="Highnam-2015"></a>Highnam, G. et al. An analytical framework for optimizing variant discovery from personal genomes. Nat. Commun. 6, 6275 (2015).
* <a name="Kozarewa-2009"></a>Kozarewa, I. et al. Amplification-free Illumina sequencing-library preparation facilitates improved mapping and assembly of (G+C)-biased genomes. Nat. Methods 6, 291–5 (2009).
* <a name="Li-2014"></a>Li, H. Toward better understanding of artifacts in variant calling from high-coverage samples. Bioinformatics 30, 2843–2851 (2014).
* <a name="Liu-2013"></a>Liu, X., Han, S., Wang, Z., Gelernter, J. & Yang, B.-Z. Variant callers for next-generation sequencing data: a comparison study. PLoS One 8, e75619 (2013).
* <a name="Meynert-2014"></a>Meynert, A. M., Ansari, M., FitzPatrick, D. R. & Taylor, M. S. Variant detection sensitivity and biases in whole genome and exome sequencing. BMC Bioinformatics 15, 247 (2014).
* <a name="Nho-2014"></a>Nho, K. et al. Comparison of Multi-Sample Variant Calling Methods for Whole Genome Sequencing. IEEE Int. Conf. Syst. Biol.  [proceedings]. IEEE Int. Conf. Syst. Biol. 2014, 59–62 (2014).
* <a name="Pabinger-2013"></a>Pabinger, S. et al. A survey of tools for variant analysis of next-generation genome sequencing data. Brief. Bioinform. bbs086– (2013). doi:10.1093/bib/bbs086
* <a name="Pirooznia-2014"></a>Pirooznia, M. et al. Validation and assessment of variant calling pipelines for next-generation sequencing. Hum. Genomics 8, 14 (2014).
* <a name="Rimmer-2014"></a>Rimmer, A. et al. Integrating mapping-, assembly- and haplotype-based approaches for calling variants in clinical sequencing applications. Nat. Genet. 46, 912–918 (2014).
* <a name="Sims-2014"></a>Sims, D., Sudbery, I., Ilott, N. E., Heger, A. & Ponting, C. P. Sequencing depth and coverage: key considerations in genomic analyses. Nat. Rev. Genet. 15, 121–32 (2014).
* <a name="Veal-2012"></a>Veal, C. D. et al. A mechanistic basis for amplification differences between samples and between genome regions. BMC Genomics 13, 455 (2012).

## Web


[GCAT]: http:/
[bcbio-nextgen]: http://bcb.io
[bcbio-nextgen-1]: http://bcb.io/2012/08/15/genomics-x-prize-public-phase-reference-genome-preparation-and-comparisons-to-illumina-and-complete-genomics
[bcbio-nextgen-2]: http://bcb.io/2012/09/17/genomics-x-prize-public-phase-update-variant-classification-and-de-novo-calling/
[bcbio-nextgen-3]: http://bcb.io/2013/02/06/an-automated-ensemble-method-for-combining-and-evaluating-genomic-variants-from-multiple-callers/
[bcbio-nextgen-4]: http://bcb.io/2013/02/13/the-influence-of-reduced-resolution-quality-scores-on-alignment-and-variant-calling/ 
[bcbio-nextgen-5]: http://bcb.io/2013/05/22/scaling-variant-detection-pipelines-for-whole-genome-sequencing-analysis/
[bcbio-nextgen-6]: http://bcb.io/2014/03/06/improving-reproducibility-and-installation-of-genomic-analysis-pipelines-with-docker/
[bcbio-nextgen-7]: http://bcb.io/2014/05/12/wgs-trio-variant-evaluation/
[bcbio-nextgen-8]: http://bcb.io/2013/05/06/framework-for-evaluating-variant-detection-methods-comparison-of-aligners-and-callers/
[bcbio-nextgen-9]: http://bcb.io/2013/10/21/updated-comparison-of-variant-detection-methods-ensemble-freebayes-and-minimal-bam-preparation-pipelines/
[bcbio-nextgen-10]: http://bcb.io/2014/10/07/joint-calling/
[vardict]: https://github.com/AstraZeneca-NGS/VarDict
[vardictjava]: https://github.com/AstraZeneca-NGS/VarDictJava

[loi-de-Poisson]: /statistics/2015/08/19/Poisson-distribution.html
[sen-spe]: /statistics/2015/06/26/sens-spe.html 
