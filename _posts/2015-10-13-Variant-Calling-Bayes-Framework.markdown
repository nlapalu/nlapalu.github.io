---
layout: post
title:  "Variant Calling - algorithmes et modèles"
createdate:   2015-10-13
lastupdate:   2015-10-13
categories: "bioinformatique"
---

# Introduction

# Framework Bayesien

## Approche générale

([Gabor *et al.*, 1999](#Gabor-1999))

Nous allons voir plusieurs outils phare de la detection de variants à partir de données NGS vis à vis d'un génome de référence. Ils utilisent tous un modèle Bayesien afin d'emmettre la probabilité d'observer tel ou tel génotype connaissant les données de séquençage en entrée ou plutôt de mapping sur le génome. Le modèle utilisé se base sur le théorème de Bayes, il est donc plus ou moins commun à tous les outils. Les principales différences entre les outils se font sur l'estimation de certains paramètres et de l'algorithme générale de l'outil. Nous allons essayer de survoler l'ensemble des méthodes et voir leur points communs et différences.
Les résultats sont fournis sous forme de Phred-like score, correspondant à \\( -10 \log_{10} [1-P(Genotype \mid Data)] \\).

## SOAPsnp
L'algorithme de SOAPsnp est décrit la publication scientifique de l'outil fait par ([Li *et al.*, 2009](#Li-2009)).

### Modèle

Sous modèle Bayesien, la probabilité d'observer le génotype \\( T_i \\) connaissant les données \\( D \\) pour un individu à un locus peut s'écrire: \\[ P(T_i \mid D)=\frac{P(T_i)P(D \mid T_i)}{\sum_{x=1}^S P(T_x)P(D \mid T_x)} \\]
S correspond au nombre total de génotypes. Donc avec \\( H_m \\) qui définit l'ensemble des génotypes haploides, pour des génomes haploides \\( T_i=H_m \in \{ A,C,G,T \} , soit S=4 \\) et pour des génomes diploides \\( T_i=H_mH_n \in \{ AA,CC,GG,TT,AC,AG,AT,CG,CT,GT \} , soit S=10 \\). La probabilité a prioiri \\( P(T_i) \\) pour chaque génotype \\( T_i \\) est défini par l'estimation du taux moyen de SNPs entre la référence et l'individu. La vraisemblance de la probabilité d'avoir un jeu de données \\( D \\) connaissant le génotype \\( T_i \\), soit  \\( P(D \mid T_i \\) est calculée à partir des alleles observés dans les données de séquençage.

### Calcul de \\( P(T_i) \\) :

Le taux de mutation entre 2 chromosomes haploides humain est fixé à 0,001 (voir référence dans le papier) et le taux de transition/transversion est de 4/1.
Voici l'exemple fourni pour G avec les taux de mutation fixés tels que le taux de mutation d'heterozygotie est de 0,001 et pour l'homozygotie 0,0005, soit 2 fois moins fréquent (pour génome diploide).
On peut donc en déduire que les probabilités à priori de génotypes haploides sont pour G = 0.999, A = 0,001 x (2/3) = 6.67.10<sup>-4</sup> et C,T = (0,001 x (1/3)) / 2 = 1,67.10<sup>-4</sup>

\\( \sum{P(T_i)} = P(GG) + P(CC) + P(TT) + P(AA) + P(GC) + P(GA) + P(GT) + P(CA) + P(CT) + P(AT) = 1\\)

* GG = 0.9985
* CC = (0.0005 x (1/3)) / 2 = 8.33 x 10<sup>-5</sup>
* TT = (0.0005 x (1/3)) / 2 = 8.33 x 10<sup>-5</sup>
* AA = 0.0005 x (2/3) = 3.33 x 10<sup>-4</sup>
* GC = 0.999 x (0.001 x (1/3) / 2) = 1.67 x 10<sup>-4</sup>
* GT = 0.999 x (0.001 x (1/3) / 2) = 1.67 x 10<sup>-4</sup>
* GA = 0.999 x 0.001 x (2/3) = 6.67 x 10<sup>-4</sup>
* CA = (0.001 x (1/3) / 2) x (0.001 x (2/3)) = 1.11 x 10<sup>-7</sup>
* CT = (0.001 x (1/3) / 2) x (0.001 x (1/3) / 2) = 2.8 x 10<sup>-8</sup>
* AT = (0.001 x (1/3) / 2) x (0.001 x (2/3)) = 1.11 x 10<sup>-7</sup>

Les paramètres de taux de mutation sont modifiables via 2 options du logiciel (-r 0.0005 [HOM] -e 0.001 [HET]), ce qui recalculera les valeurs à priori des génotypes.

### Calcul de \\( P(D \mid T_i) \\) :

En ce qui concerne la probabilité d'observer les données D connaissant le génotype Ti, soit la vraissemblance de \\( P(D \mid T_i) \\). Pour cela, la vraisemblance d'observer l'allèle \\( d_k \\) dans un génotype H haploide est \\( P(d_k \mid H) \\).
Dans le cas d'un génotype \\( T \\) diploide, cela se transforme pour les 2 chromosomes \\( m, n \\) : 

\\[ P(d_k \mid T)= \frac{P(d_k \mid H_m) + P(d_k \mid H_n)}{2} \\]
Donc pour un set \\( n \\) d'allèles au locus considéré, \\( D = {d_1,d_2, ..., d_n} \\):
\\[ P(D \mid T) = \prod_{k=1}^n P(d_k \mid T) \\]


Pour chaque allèle \\( d_k \\) observé depuis un read mappé à un locus, 4 attributs sont pris en compte: 

* \\( o_k \\) : le type d'allèle observé
* \\( q_k \\) : le score de qualité
* \\( c_k \\) : le cycle de séquençage (position dans le read)
* \\( t_k \\) : la \\( t_k \\) enième observation de l'allèle parmi les reads mappés à ce locus

Dans un premier temps, on suppose que les erreurs de séquençage sont independantes. Seules les attributs \\( o_k, q_k, c_k \\) sont utilisé pour le calcul de la vraisemblance comme ci-dessous:
\\[ P(d_k \mid H) = P((o_k,q_k,c_k) \mid H) = P((o_k,c_k) \mid (H,q_k)) \times  P(q_k \mid H) \\]
Une matrice à 4 dimensions est crée donnant la vraisemblance d'observation de l'allele \\( d_k \\) avec le type \\( o_k \\) avec la qualité \\( q_k \\) à la position \\( c_k \\) pour le genotype \\( H \\). 
En ce qui concerne \\( P(q_k \mid H) \\), c'est la probabilité d'avoir une obsrevation de \\( H \\) avec une qualité de \\( q_k \\). La distribution des qualité n'étant pas connue, elle est considérée comme homogène pour les 4 nucléotides et donc exprimable sous forme de fonction de \\( q_k \\). Elle est donc supprimée de la formule bayesienne.

Dans un second temps, il est possible d'inclure un facteur de dépendance des erreurs nommé \\( \theta \\). Les scores de qualité des différentes observations au locus considéré sont classés par ordre croissant. Il y a un traitement qui permet de réduire la qualité observé pour chaque \\( t_k \\) enième observation :
\\[ q'_k = \theta^{t_k} q_k \\]
Dans ce cas la qualité \\( q' \\) est utilsée à la place de \\( q \\). Si \\( \theta \\) est égal à 0, il y a dépendance complète du modèle, si il est égal à 1, il y a indépendance totale.

### Test de rang pour les génotypes Hétérozygotes :

à creuser ....

### Résultat :

Pouvant maintenant calculer \\( P(T_i) \\) et \\( P(D \mid T_i) \\), on en déduit par le théorème de Bayes, la probabilité à postériori de chaque génotype \\( T_i \\). La probabilité à postériori \\( P(T_i \mid D) \\) la plus élevée est retenue et transformé en phred score.

## GATK

## Samtools

## FreeBayes

### Modèle

### Calcul de \\( P(R_i \mid G_i) \\)

\\[ P(R_i \mid G_i) \approx P( B_i^\prime \mid G_i) = \frac{s_i!}{\prod_{j=1}^{k_i} o_j^\prime!} \prod_{j=1}^{k_i} \left( \frac{f_{i_j}}{m_i} \right)^{o_j^\prime} \\]

exemple de calcul, pour un locus sur un échantillon, avec un mapping de 100 reads, dont 98 avec un allele (a) et 2 un autre allele (c). 
Le locus est supposé unique dans le génome et on suppose que pour le génotype homozygote aa, la fréquence de a pour ce génotype selon ces données est de 0,99 et c de 0,01
\\[ P(R_i \mid aa) =  \frac{100!}{98!2!} \left( \frac{0.99}{1} \right)^{98} \left( \frac{0.01}{1} \right)^{2} = 0.185 \\]

on refait le même calcul pour le génotype ac, mais les fréquences passent à a = 0,5 et c = 0,5 
\\[ P(R_i \mid ac) = \frac{100!}{98!2!} \left( \frac{0.5}{1} \right)^{98} \left( \frac{0.5}{1} \right)^{2} \simeq 0 \\]

on refait les même calcul pour les 2 génotypes, aa et ac mais avec 50 reads allele (a) et 50 reads allele (c) 

\\[ P(R_i \mid aa) =  \frac{100!}{50!50!} \left( \frac{0.99}{1} \right)^{50} \left( \frac{0.01}{1} \right)^{50} \simeq 0\\]

\\[ P(R_i \mid ac) = \frac{100!}{50!50!} \left( \frac{0.5}{1} \right)^{50} \left( \frac{0.5}{1} \right)^{50} = 0.08 \\]

### Calcul de \\( P(G_1,...,G_n) \\) (Probabilité à priori)

### Calcul de la fréquence de chaque allèle

## Publications

* <a name="Gabor-1999"></a>.....
* <a name="Li-2009"></a> Li, R. et al. SNP detection for massively parallel whole-genome resequencing. Genome Res. 19, 112432 (2009).


