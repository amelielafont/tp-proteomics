# TP de Protéomique LYON@BioInfo 2021
## Contexte Biologique
Vous allez utiliser des outils informatiques qui vous permettront d’analyser des données brutes issues d’une analyse Shotgun Proteomics récemment publiée dans le journal Science sous le titre "Real-time visualization of drug resistance acquisition by horizontal gene transfer reveals a new role for AcrAB-TolC multidrug efflux pump".

Les données associées à cette publication sont publiques et accessibles sur la plateforme [PRIDE](https://www.ebi.ac.uk/pride/archive/projects/PXD011286). Le PDF de la publication est [`data/Nolivos_2019.pdf`](data/Nolivos_2019.pdf).

## Mise en place

### Méthodologie

Vous forkerez le présent "repository" pour vous permettre de sauvegarder votre travail.
Vous le clonerez ensuite dans votre espace de travail.
Vous éditerez ce fichier `README.md` pour répondre aux questions dans les encarts prévus à cet effet et inserer les figures que vous aurez générées. Ce "repository" vous appartenant, vous pouvez créer tous les repertoires et fichiers necessaires à la conduite du TP.

### Ressources

Seul le langage Python (v3.X) est requis pour ce travail.
Il vous est conseillé d'installer un environnement virtuel python pour installer les libraries requises independamment de votre systèmes d'exploitation.

* Le systême de gestion de paquets [Conda](https://docs.conda.io/en/latest/) est très pratique et disponible pour la plupat des systèmes d'exploitation. Une version légère suffisante pour nos besoin est téléchargeable [ici](https://docs.conda.io/en/latest/miniconda.html)
* Si vous disposez d'un interpreteur python 3.X installé sur votre systême [virtualenv](https://docs.python.org/3/library/venv.html) est désormais "built-in".

#### Procédure conda

Depuis le repertoire de votre repository Git, installez le package scipy et lancez jupyter.

```sh
$PATH_TO_CONDA_DIR/bin/conda install -c conda-forge scipy notebook
$PATH_TO_CONDA_DIR/bin/jupyter notebook
```

#### Procédure virtualenv

Créer l'environnement virtuel.

```python -m venv MADP_TP1```

Activer l'environnement virtuel et installer les packages.

```
source MADP_TP1/bin/activate.sh
pip install --user ipykernel scipy notebook
```
#### Procédure VM IFB

Une "appliance" IFB a été préparée avec les dépendances Python requises.
Elle est accessible [ici](https://biosphere.france-bioinformatique.fr/catalogue/appliance/160/).
Jupyter vous permettra d'ouvrir des terminaux SHELL et des notebook Python.
Le repertoire racine de Jupyter est `/mnt/mydatalocal/`


#### Intégration des environnements au notebook

Il peut être pratique d'ajouter votre environnement à Jupyter. Cela se réalise depuis l'environnement (conda ou venv) activé.

```
python -m ipykernel install --user --name=MADP_TP1
```


Jupyter est une environnement de type notebook permettant l'execution de code python dans des cellules avec une persitance des variables entre chaque evaluation de cellule. Jupyter fournit nativement le support de la librarie graphique matplotlib.

### Test de l'installation

Dans l'inteface de jupyter, créez un nouveau fichier notebook (*.ipynb) localisé dans votre repertoire git.
Dans la première cellule copiez le code suivant:

```python
%matplotlib nbagg
import matplotlib.pyplot as plt
import numpy as np
from scipy.stats import norm
```

La première ligne sert à activer le rendu graphique, pour tout le fichier notebook. Pour **dessiner des graphiques**, il vous est conseillé de suivre la méthode illustrée par le code suivant que vous pouvez executer dans une deuxième cellule notebook.

```python
fig, ax = plt.subplots()
x = np.linspace(norm.ppf(0.01),
                norm.ppf(0.99), 100)

ax.plot(x, norm.pdf(x),
       'r-', lw=5, alpha=0.6)

ax.plot(x, np.full(len(x), 0.2),
       'b-', lw=1)

fig.show()
```
! [] (index.png)

* Creation des objets `fig`et `ax`
* Ajout successif de graphiques sur la même figure par l'appel à des methodes de l'objet `ax`
* Affichage de la figure complète via `fig.show()`
* Evaluation de la cellule pour visualisation dans la cellule de résultats.

L'affichage dans la cellule de rendu du notebook devrait confirmer la bonne installation des dépendances.

#### On entend par figure un graphique avec des **axes légendés et un titre**.

La documentation matplotlib est bien faite, mais **attention** il vous est demandé, pour la construction des graphiques, **d'ignorer les méthodes de l'objet `plt`** (frequemment proposées sur le net) et d'utiliser uniquement les méthodes de l'[objet Axes](https://matplotlib.org/api/axes_api.html?highlight=axe#plotting). `plt` est un objet global susceptible d'agir simultanement sur tous les graphiques d'un notebook. A ce stade, son utilisation est donc à éviter.

## Données disponibles

### Mesures experimentales

Un fichier `data/TCL_wt1.tsv` contient les données d'abondances differentielles mesurées sur une souche sauvage d'*Escherichia coli* entre deux conditions: avec Tetracycline et en milieu riche. Le contrôle est le milieu riche.

| Accession | Description | Gene Symbol  |   Corrected Abundance ratio (1.53)    | Log2 Corrected Abundance Ratio | Abundance Ratio Adj. P-Value |   -LOG10 Adj.P-val |
| --- | --- | --- | --- | --- | --- | ---|
| [Accesseur Uniprot](https://www.uniprot.org/help/accession_numbers)  | Texte libre | Texte libre  | <img src="https://render.githubusercontent.com/render/math?math=\frac{\text{WildType}_{\text{Tc}}}{\text{WildType}_{\text{rich}}}"> | <img src="https://render.githubusercontent.com/render/math?math=Log_2(\frac{\text{WildType}_{\text{Tc}}}{\text{WildType}_{\text{rich}}})">  | <img src="https://render.githubusercontent.com/render/math?math=\mathbb{R}^%2B"> | <img src="https://render.githubusercontent.com/render/math?math=\mathbb{R}^%2B">  |

Attention certaines valeurs numériques sont manquantes ou erronées, constatez par vous même en parcourant rapidement le fichier.

### Fiches uniprot

Les fiches de toutes les protéines de *E.coli* sont stockées dans un seul document XML `data/uniprot-proteome_UP000000625.xml`. Nous allons extraire de ce fichier les informations dont nous aurons besoin, à l'aide du module de la librarie standard [XML.etree](https://docs.python.org/3/library/xml.etree.elementtree.html#module-xml.etree.ElementTree). Pour vous facilitez la tâche, les exemples suivants vous sont fournis. Prenez le temps de les executer et de les modifier dans un notebook. Vous vous familliariserez ainsi avec la structure du document XML que vous pouvez egalement inspecter dans un navigateur.

```python
from xml.etree.ElementTree import parse, dump
# Parse the E.Coli proteome XML Document
tree = parse('data/uniprot-proteome_UP000000625.xml')
root = tree.getroot()
ns = '{http://uniprot.org/uniprot}' # MANDATORY PREFIX FOR ANY SEARCH within document
# Store all entries aka proteins in a list of xml nodes
proteins = root.findall(ns + 'entry')
# Display the xml subtree of the first protein 
dump(proteins[0])
```
Out: 
```xml
<ns0:entry xmlns:ns0="http://uniprot.org/uniprot" dataset="Swiss-Prot" created="1989-10-01" modified="2020-08-12" version="166">
<ns0:accession>P11446</ns0:accession>
<ns0:accession>Q2M8Q3</ns0:accession>
<ns0:name>ARGC_ECOLI</ns0:name>
<ns0:protein>
<ns0:recommendedName>
<ns0:fullName evidence="1 4">N-acetyl-gamma-glutamyl-phosphate reductase</ns0:fullName>
<ns0:shortName evidence="1 4">AGPR</ns0:shortName>
<ns0:ecNumber evidence="1 2">1.2.1.38</ns0:ecNumber>
</ns0:recommendedName>
<ns0:alternativeName>
<ns0:fullName evidence="1 3">N-acetyl-glutamate semialdehyde dehydrogenase</ns0:fullName>
<ns0:shortName evidence="1 3">NAGSA dehydrogenase</ns0:shortName>
</ns0:alternativeName>
</ns0:protein>
<ns0:gene>
<ns0:name type="primary" evidence="1">argC</ns0:name>
<ns0:name type="ordered locus">b3958</ns0:name>
<ns0:name type="ordered locus">JW3930</ns0:name>
</ns0:gene>
<ns0:organism>
<ns0:name type="scientific">Escherichia coli (strain K12)</ns0:name>
<ns0:dbReference type="NCBI Taxonomy" id="83333" />
<ns0:lineage>
<ns0:taxon>Bacteria</ns0:taxon>
<ns0:taxon>Proteobacteria</ns0:taxon>
<ns0:taxon>Gammaproteobacteria</ns0:taxon>
<ns0:taxon>Enterobacterales</ns0:taxon>
<ns0:taxon>Enterobacteriaceae</ns0:taxon>
<ns0:taxon>Escherichia</ns0:taxon>
</ns0:lineage>
</ns0:organism>
<ns0:reference key="1">
<ns0:citation type="journal article" date="1988" name="Gene" volume="68" first="275" last="283">
<ns0:title>Nucleotide sequence of Escherichia coli argB and argC genes: comparison of N-acetylglutamate kinase and N-acetylglutamate-gamma-semialdehyde dehydrogenase with homologous and analogous enzymes.</ns0:title>
<ns0:authorList>
<ns0:person name="Parsot C." />
<ns0:person name="Boyen A." />
<ns0:person name="Cohen G.N." />
<ns0:person name="Glansdorff N." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="2851495" />
<ns0:dbReference type="DOI" id="10.1016/0378-1119(88)90030-3" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA]</ns0:scope>
</ns0:reference>
<ns0:reference key="2">
<ns0:citation type="journal article" date="1993" name="Nucleic Acids Res." volume="21" first="5408" last="5417">
<ns0:title>Analysis of the Escherichia coli genome. IV. DNA sequence of the region from 89.2 to 92.8 minutes.</ns0:title>
<ns0:authorList>
<ns0:person name="Blattner F.R." />
<ns0:person name="Burland V.D." />
<ns0:person name="Plunkett G. III" />
<ns0:person name="Sofia H.J." />
<ns0:person name="Daniels D.L." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="8265357" />
<ns0:dbReference type="DOI" id="10.1093/nar/21.23.5408" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="3">
<ns0:citation type="journal article" date="1997" name="Science" volume="277" first="1453" last="1462">
<ns0:title>The complete genome sequence of Escherichia coli K-12.</ns0:title>
<ns0:authorList>
<ns0:person name="Blattner F.R." />
<ns0:person name="Plunkett G. III" />
<ns0:person name="Bloch C.A." />
<ns0:person name="Perna N.T." />
<ns0:person name="Burland V." />
<ns0:person name="Riley M." />
<ns0:person name="Collado-Vides J." />
<ns0:person name="Glasner J.D." />
<ns0:person name="Rode C.K." />
<ns0:person name="Mayhew G.F." />
<ns0:person name="Gregor J." />
<ns0:person name="Davis N.W." />
<ns0:person name="Kirkpatrick H.A." />
<ns0:person name="Goeden M.A." />
<ns0:person name="Rose D.J." />
<ns0:person name="Mau B." />
<ns0:person name="Shao Y." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="9278503" />
<ns0:dbReference type="DOI" id="10.1126/science.277.5331.1453" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="4">
<ns0:citation type="journal article" date="2006" name="Mol. Syst. Biol." volume="2" first="E1" last="E5">
<ns0:title>Highly accurate genome sequences of Escherichia coli K-12 strains MG1655 and W3110.</ns0:title>
<ns0:authorList>
<ns0:person name="Hayashi K." />
<ns0:person name="Morooka N." />
<ns0:person name="Yamamoto Y." />
<ns0:person name="Fujita K." />
<ns0:person name="Isono K." />
<ns0:person name="Choi S." />
<ns0:person name="Ohtsubo E." />
<ns0:person name="Baba T." />
<ns0:person name="Wanner B.L." />
<ns0:person name="Mori H." />
<ns0:person name="Horiuchi T." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16738553" />
<ns0:dbReference type="DOI" id="10.1038/msb4100049" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / W3110 / ATCC 27325 / DSM 5911</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="5">
<ns0:citation type="journal article" date="1982" name="Nucleic Acids Res." volume="10" first="8031" last="8048">
<ns0:title>The regulatory region of the divergent argECBH operon in Escherichia coli K-12.</ns0:title>
<ns0:authorList>
<ns0:person name="Piette J." />
<ns0:person name="Cunin R." />
<ns0:person name="Boyen A." />
<ns0:person name="Charlier D.R.M." />
<ns0:person name="Crabeel M." />
<ns0:person name="van Vliet F." />
<ns0:person name="Glansdorff N." />
<ns0:person name="Squires C." />
<ns0:person name="Squires C.L." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="6761650" />
<ns0:dbReference type="DOI" id="10.1093/nar/10.24.8031" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA] OF 1-48</ns0:scope>
<ns0:source>
<ns0:strain>K12</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="6">
<ns0:citation type="journal article" date="1992" name="J. Bacteriol." volume="174" first="2323" last="2331">
<ns0:title>Structural and biochemical characterization of the Escherichia coli argE gene product.</ns0:title>
<ns0:authorList>
<ns0:person name="Meinnel T." />
<ns0:person name="Schmitt E." />
<ns0:person name="Mechulam Y." />
<ns0:person name="Blanquet S." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="1551850" />
<ns0:dbReference type="DOI" id="10.1128/jb.174.7.2323-2331.1992" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA] OF 1-19</ns0:scope>
<ns0:source>
<ns0:strain>K12</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="7">
<ns0:citation type="journal article" date="1962" name="Biochem. Biophys. Res. Commun." volume="7" first="491" last="496">
<ns0:title>N-Acetyl-gamma-Ilutamokinase and N-acetylglutamic gamma-semialdehyde dehydrogenase: repressible enzymes of arginine synthesis in Escherichia coli.</ns0:title>
<ns0:authorList>
<ns0:person name="Baich A." />
<ns0:person name="Vogel H.J." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="13863980" />
<ns0:dbReference type="DOI" id="10.1016/0006-291x(62)90342-x" />
</ns0:citation>
<ns0:scope>FUNCTION</ns0:scope>
<ns0:scope>CATALYTIC ACTIVITY</ns0:scope>
<ns0:scope>PATHWAY</ns0:scope>
<ns0:source>
<ns0:strain>W / ATCC 11105 / DSM 1900</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:comment type="function">
<ns0:text evidence="1 2">Catalyzes the NADPH-dependent reduction of N-acetyl-5-glutamyl phosphate to yield N-acetyl-L-glutamate 5-semialdehyde.</ns0:text>
</ns0:comment>
<ns0:comment type="catalytic activity">
<ns0:reaction evidence="1 2">
<ns0:text>N-acetyl-L-glutamate 5-semialdehyde + NADP(+) + phosphate = H(+) + N-acetyl-L-glutamyl 5-phosphate + NADPH</ns0:text>
<ns0:dbReference type="Rhea" id="RHEA:21588" />
<ns0:dbReference type="ChEBI" id="CHEBI:15378" />
<ns0:dbReference type="ChEBI" id="CHEBI:29123" />
<ns0:dbReference type="ChEBI" id="CHEBI:43474" />
<ns0:dbReference type="ChEBI" id="CHEBI:57783" />
<ns0:dbReference type="ChEBI" id="CHEBI:57936" />
<ns0:dbReference type="ChEBI" id="CHEBI:58349" />
<ns0:dbReference type="EC" id="1.2.1.38" />
</ns0:reaction>
</ns0:comment>
<ns0:comment type="pathway" evidence="1 5">
<ns0:text>Amino-acid biosynthesis; L-arginine biosynthesis; N(2)-acetyl-L-ornithine from L-glutamate: step 3/4.</ns0:text>
</ns0:comment>
<ns0:comment type="subcellular location">
<ns0:subcellularLocation>
<ns0:location evidence="1">Cytoplasm</ns0:location>
</ns0:subcellularLocation>
</ns0:comment>
<ns0:comment type="similarity">
<ns0:text evidence="1 4">Belongs to the NAGSA dehydrogenase family. Type 1 subfamily.</ns0:text>
</ns0:comment>
<ns0:dbReference type="EC" id="1.2.1.38" evidence="1 2" />
<ns0:dbReference type="EMBL" id="M21446">
<ns0:property type="protein sequence ID" value="AAA23477.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U00006">
<ns0:property type="protein sequence ID" value="AAC43064.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U00096">
<ns0:property type="protein sequence ID" value="AAC76940.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="AP009048">
<ns0:property type="protein sequence ID" value="BAE77353.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="J01587">
<ns0:property type="protein sequence ID" value="AAB59146.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="X55417">
<ns0:property type="status" value="NOT_ANNOTATED_CDS" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="PIR" id="JT0332">
<ns0:property type="entry name" value="RDECEP" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="NP_418393.1">
<ns0:property type="nucleotide sequence ID" value="NC_000913.3" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="WP_000935370.1">
<ns0:property type="nucleotide sequence ID" value="NZ_STEB01000037.1" />
</ns0:dbReference>
<ns0:dbReference type="SMR" id="P11446" />
<ns0:dbReference type="BioGRID" id="4262190">
<ns0:property type="interactions" value="6" />
</ns0:dbReference>
<ns0:dbReference type="BioGRID" id="852751">
<ns0:property type="interactions" value="1" />
</ns0:dbReference>
<ns0:dbReference type="IntAct" id="P11446">
<ns0:property type="interactions" value="1" />
</ns0:dbReference>
<ns0:dbReference type="STRING" id="511145.b3958" />
<ns0:dbReference type="PaxDb" id="P11446" />
<ns0:dbReference type="PRIDE" id="P11446" />
<ns0:dbReference type="EnsemblBacteria" id="AAC76940">
<ns0:property type="protein sequence ID" value="AAC76940" />
<ns0:property type="gene ID" value="b3958" />
</ns0:dbReference>
<ns0:dbReference type="EnsemblBacteria" id="BAE77353">
<ns0:property type="protein sequence ID" value="BAE77353" />
<ns0:property type="gene ID" value="BAE77353" />
</ns0:dbReference>
<ns0:dbReference type="GeneID" id="948455" />
<ns0:dbReference type="KEGG" id="ecj:JW3930" />
<ns0:dbReference type="KEGG" id="eco:b3958" />
<ns0:dbReference type="PATRIC" id="fig|1411691.4.peg.2747" />
<ns0:dbReference type="EchoBASE" id="EB0063" />
<ns0:dbReference type="eggNOG" id="COG0002">
<ns0:property type="taxonomic scope" value="Bacteria" />
</ns0:dbReference>
<ns0:dbReference type="HOGENOM" id="CLU_006384_0_1_6" />
<ns0:dbReference type="InParanoid" id="P11446" />
<ns0:dbReference type="KO" id="K00145" />
<ns0:dbReference type="PhylomeDB" id="P11446" />
<ns0:dbReference type="BioCyc" id="EcoCyc:N-ACETYLGLUTPREDUCT-MONOMER" />
<ns0:dbReference type="BioCyc" id="ECOL316407:JW3930-MONOMER" />
<ns0:dbReference type="BioCyc" id="MetaCyc:N-ACETYLGLUTPREDUCT-MONOMER" />
<ns0:dbReference type="BRENDA" id="1.2.1.38">
<ns0:property type="organism ID" value="2026" />
</ns0:dbReference>
<ns0:dbReference type="UniPathway" id="UPA00068">
<ns0:property type="reaction ID" value="UER00108" />
</ns0:dbReference>
<ns0:dbReference type="PRO" id="PR:P11446" />
<ns0:dbReference type="Proteomes" id="UP000000318">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="Proteomes" id="UP000000625">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0005737">
<ns0:property type="term" value="C:cytoplasm" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="UniProtKB-SubCell" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0003942">
<ns0:property type="term" value="F:N-acetyl-gamma-glutamyl-phosphate reductase activity" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0051287">
<ns0:property type="term" value="F:NAD binding" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="InterPro" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0046983">
<ns0:property type="term" value="F:protein dimerization activity" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="InterPro" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0006526">
<ns0:property type="term" value="P:arginine biosynthetic process" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="HAMAP" id="MF_00150">
<ns0:property type="entry name" value="ArgC_type1" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR023013">
<ns0:property type="entry name" value="AGPR_AS" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR000706">
<ns0:property type="entry name" value="AGPR_type-1" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR036291">
<ns0:property type="entry name" value="NAD(P)-bd_dom_sf" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR000534">
<ns0:property type="entry name" value="Semialdehyde_DH_NAD-bd" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR012280">
<ns0:property type="entry name" value="Semialdhyde_DH_dimer_dom" />
</ns0:dbReference>
<ns0:dbReference type="Pfam" id="PF01118">
<ns0:property type="entry name" value="Semialdhyde_dh" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="Pfam" id="PF02774">
<ns0:property type="entry name" value="Semialdhyde_dhC" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="SMART" id="SM00859">
<ns0:property type="entry name" value="Semialdhyde_dh" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="SUPFAM" id="SSF51735">
<ns0:property type="entry name" value="SSF51735" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="TIGRFAMs" id="TIGR01850">
<ns0:property type="entry name" value="argC" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="PROSITE" id="PS01224">
<ns0:property type="entry name" value="ARGC" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:proteinExistence type="evidence at protein level" />
<ns0:keyword id="KW-0028">Amino-acid biosynthesis</ns0:keyword>
<ns0:keyword id="KW-0055">Arginine biosynthesis</ns0:keyword>
<ns0:keyword id="KW-0963">Cytoplasm</ns0:keyword>
<ns0:keyword id="KW-0521">NADP</ns0:keyword>
<ns0:keyword id="KW-0560">Oxidoreductase</ns0:keyword>
<ns0:keyword id="KW-1185">Reference proteome</ns0:keyword>
<ns0:feature type="chain" description="N-acetyl-gamma-glutamyl-phosphate reductase" id="PRO_0000112401">
<ns0:location>
<ns0:begin position="1" />
<ns0:end position="334" />
</ns0:location>
</ns0:feature>
<ns0:feature type="active site" evidence="1">
<ns0:location>
<ns0:position position="154" />
</ns0:location>
</ns0:feature>
<ns0:evidence key="1" type="ECO:0000255">
<ns0:source>
<ns0:dbReference type="HAMAP-Rule" id="MF_00150" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="2" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="13863980" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="3" type="ECO:0000303">
<ns0:source>
<ns0:dbReference type="PubMed" id="2851495" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="4" type="ECO:0000305" />
<ns0:evidence key="5" type="ECO:0000305">
<ns0:source>
<ns0:dbReference type="PubMed" id="13863980" />
</ns0:source>
</ns0:evidence>
<ns0:sequence length="334" mass="35952" checksum="67AC195ECE1C4789" modified="1989-10-01" version="1">MLNTLIVGASGYAGAELVTYVNRHPHMNITALTVSAQSNDAGKLISDLHPQLKGIVDLPLQPMSDISEFSPGVDVVFLATAHEVSHDLAPQFLEAGCVVFDLSGAFRVNDATFYEKYYGFTHQYPELLEQAAYGLAEWCGNKLKEANLIAVPGCYPTAAQLALKPLIDADLLDLNQWPVINATSGVSGAGRKAAISNSFCEVSLQPYGVFTHRHQPEIATHLGADVIFTPHLGNFPRGILETITCRLKSGVTQAQVAQVLQQAYAHKPLVRLYDKGVPALKNVVGLPFCDIGFAVQGEHLIIVATEDNLLKGAAAQAVQCANIRFGYAETQSLI</ns0:sequence>
</ns0:entry>
```

```python
# Find the xml subtree of a protein with accession "P31224"
for entry in proteins:
    accessions = entry.findall(ns+"accession")
    for acc in accessions:
        if acc.text == "P31224":
            dump(entry)
            break
```
Out: 
```xml
ns0:entry xmlns:ns0="http://uniprot.org/uniprot" dataset="Swiss-Prot" created="1993-07-01" modified="2020-08-12" version="187">
<ns0:accession>P31224</ns0:accession>
<ns0:accession>Q2MBW5</ns0:accession>
<ns0:name>ACRB_ECOLI</ns0:name>
<ns0:protein>
<ns0:recommendedName>
<ns0:fullName>Multidrug efflux pump subunit AcrB</ns0:fullName>
</ns0:recommendedName>
<ns0:alternativeName>
<ns0:fullName>AcrAB-TolC multidrug efflux pump subunit AcrB</ns0:fullName>
</ns0:alternativeName>
<ns0:alternativeName>
<ns0:fullName>Acridine resistance protein B</ns0:fullName>
</ns0:alternativeName>
</ns0:protein>
<ns0:gene>
<ns0:name type="primary">acrB</ns0:name>
<ns0:name type="synonym">acrE</ns0:name>
<ns0:name type="ordered locus">b0462</ns0:name>
<ns0:name type="ordered locus">JW0451</ns0:name>
</ns0:gene>
<ns0:organism>
<ns0:name type="scientific">Escherichia coli (strain K12)</ns0:name>
<ns0:dbReference type="NCBI Taxonomy" id="83333" />
<ns0:lineage>
<ns0:taxon>Bacteria</ns0:taxon>
<ns0:taxon>Proteobacteria</ns0:taxon>
<ns0:taxon>Gammaproteobacteria</ns0:taxon>
<ns0:taxon>Enterobacterales</ns0:taxon>
<ns0:taxon>Enterobacteriaceae</ns0:taxon>
<ns0:taxon>Escherichia</ns0:taxon>
</ns0:lineage>
</ns0:organism>
<ns0:reference key="1">
<ns0:citation type="submission" date="1993-05" db="EMBL/GenBank/DDBJ databases">
<ns0:title>Nucleotide sequence of the acrAB operon from Escherichia coli.</ns0:title>
<ns0:authorList>
<ns0:person name="Xu J." />
<ns0:person name="Bertrand K.P." />
</ns0:authorList>
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="2">
<ns0:citation type="journal article" date="1993" name="J. Bacteriol." volume="175" first="6299" last="6313">
<ns0:title>Molecular cloning and characterization of acrA and acrE genes of Escherichia coli.</ns0:title>
<ns0:authorList>
<ns0:person name="Ma D." />
<ns0:person name="Cook D.N." />
<ns0:person name="Alberti M." />
<ns0:person name="Pon N.G." />
<ns0:person name="Nikaido H." />
<ns0:person name="Hearst J.E." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="8407802" />
<ns0:dbReference type="DOI" id="10.1128/jb.175.19.6299-6313.1993" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / W4573</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="3">
<ns0:citation type="submission" date="1997-01" db="EMBL/GenBank/DDBJ databases">
<ns0:title>Sequence of minutes 4-25 of Escherichia coli.</ns0:title>
<ns0:authorList>
<ns0:person name="Chung E." />
<ns0:person name="Allen E." />
<ns0:person name="Araujo R." />
<ns0:person name="Aparicio A.M." />
<ns0:person name="Davis K." />
<ns0:person name="Duncan M." />
<ns0:person name="Federspiel N." />
<ns0:person name="Hyman R." />
<ns0:person name="Kalman S." />
<ns0:person name="Komp C." />
<ns0:person name="Kurdi O." />
<ns0:person name="Lew H." />
<ns0:person name="Lin D." />
<ns0:person name="Namath A." />
<ns0:person name="Oefner P." />
<ns0:person name="Roberts D." />
<ns0:person name="Schramm S." />
<ns0:person name="Davis R.W." />
</ns0:authorList>
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="4">
<ns0:citation type="journal article" date="1997" name="Science" volume="277" first="1453" last="1462">
<ns0:title>The complete genome sequence of Escherichia coli K-12.</ns0:title>
<ns0:authorList>
<ns0:person name="Blattner F.R." />
<ns0:person name="Plunkett G. III" />
<ns0:person name="Bloch C.A." />
<ns0:person name="Perna N.T." />
<ns0:person name="Burland V." />
<ns0:person name="Riley M." />
<ns0:person name="Collado-Vides J." />
<ns0:person name="Glasner J.D." />
<ns0:person name="Rode C.K." />
<ns0:person name="Mayhew G.F." />
<ns0:person name="Gregor J." />
<ns0:person name="Davis N.W." />
<ns0:person name="Kirkpatrick H.A." />
<ns0:person name="Goeden M.A." />
<ns0:person name="Rose D.J." />
<ns0:person name="Mau B." />
<ns0:person name="Shao Y." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="9278503" />
<ns0:dbReference type="DOI" id="10.1126/science.277.5331.1453" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="5">
<ns0:citation type="journal article" date="2006" name="Mol. Syst. Biol." volume="2" first="E1" last="E5">
<ns0:title>Highly accurate genome sequences of Escherichia coli K-12 strains MG1655 and W3110.</ns0:title>
<ns0:authorList>
<ns0:person name="Hayashi K." />
<ns0:person name="Morooka N." />
<ns0:person name="Yamamoto Y." />
<ns0:person name="Fujita K." />
<ns0:person name="Isono K." />
<ns0:person name="Choi S." />
<ns0:person name="Ohtsubo E." />
<ns0:person name="Baba T." />
<ns0:person name="Wanner B.L." />
<ns0:person name="Mori H." />
<ns0:person name="Horiuchi T." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16738553" />
<ns0:dbReference type="DOI" id="10.1038/msb4100049" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / W3110 / ATCC 27325 / DSM 5911</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="6">
<ns0:citation type="journal article" date="1995" name="Mol. Microbiol." volume="16" first="45" last="55">
<ns0:title>Genes acrA and acrB encode a stress-induced efflux system of Escherichia coli.</ns0:title>
<ns0:authorList>
<ns0:person name="Ma D." />
<ns0:person name="Cook D.N." />
<ns0:person name="Alberti M." />
<ns0:person name="Pon N.G." />
<ns0:person name="Nikaido H." />
<ns0:person name="Hearst J.E." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="7651136" />
<ns0:dbReference type="DOI" id="10.1111/j.1365-2958.1995.tb02390.x" />
</ns0:citation>
<ns0:scope>CHARACTERIZATION</ns0:scope>
</ns0:reference>
<ns0:reference key="7">
<ns0:citation type="journal article" date="2000" name="J. Biochem." volume="128" first="195" last="200">
<ns0:title>Molecular construction of a multidrug exporter system, AcrAB: molecular interaction between AcrA and AcrB, and cleavage of the N-terminal signal sequence of AcrA.</ns0:title>
<ns0:authorList>
<ns0:person name="Kawabe T." />
<ns0:person name="Fujihira E." />
<ns0:person name="Yamaguchi A." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="10920254" />
<ns0:dbReference type="DOI" id="10.1093/oxfordjournals.jbchem.a022741" />
</ns0:citation>
<ns0:scope>INTERACTION WITH ACRA</ns0:scope>
</ns0:reference>
<ns0:reference key="8">
<ns0:citation type="journal article" date="2004" name="Mol. Microbiol." volume="53" first="697" last="706">
<ns0:title>Interactions underlying assembly of the Escherichia coli AcrAB-TolC multidrug efflux system.</ns0:title>
<ns0:authorList>
<ns0:person name="Touze T." />
<ns0:person name="Eswaran J." />
<ns0:person name="Bokma E." />
<ns0:person name="Koronakis E." />
<ns0:person name="Hughes C." />
<ns0:person name="Koronakis V." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="15228545" />
<ns0:dbReference type="DOI" id="10.1111/j.1365-2958.2004.04158.x" />
</ns0:citation>
<ns0:scope>INTERACTION WITH ACRA AND TOLC</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
<ns0:scope>SUBCELLULAR LOCATION</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MC1061 / ATCC 53338 / DSM 7140</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="9">
<ns0:citation type="journal article" date="2005" name="J. Biol. Chem." volume="280" first="34409" last="34419">
<ns0:title>Protein complexes of the Escherichia coli cell envelope.</ns0:title>
<ns0:authorList>
<ns0:person name="Stenberg F." />
<ns0:person name="Chovanec P." />
<ns0:person name="Maslen S.L." />
<ns0:person name="Robinson C.V." />
<ns0:person name="Ilag L." />
<ns0:person name="von Heijne G." />
<ns0:person name="Daley D.O." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16079137" />
<ns0:dbReference type="DOI" id="10.1074/jbc.m506479200" />
</ns0:citation>
<ns0:scope>SUBUNIT</ns0:scope>
<ns0:scope>SUBCELLULAR LOCATION</ns0:scope>
<ns0:source>
<ns0:strain>BL21-DE3</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="10">
<ns0:citation type="journal article" date="2005" name="Science" volume="308" first="1321" last="1323">
<ns0:title>Global topology analysis of the Escherichia coli inner membrane proteome.</ns0:title>
<ns0:authorList>
<ns0:person name="Daley D.O." />
<ns0:person name="Rapp M." />
<ns0:person name="Granseth E." />
<ns0:person name="Melen K." />
<ns0:person name="Drew D." />
<ns0:person name="von Heijne G." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="15919996" />
<ns0:dbReference type="DOI" id="10.1126/science.1109730" />
</ns0:citation>
<ns0:scope>TOPOLOGY [LARGE SCALE ANALYSIS]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="11">
<ns0:citation type="journal article" date="2008" name="Mol. Microbiol." volume="70" first="323" last="340">
<ns0:title>Contact-dependent growth inhibition requires the essential outer membrane protein BamA (YaeT) as the receptor and the inner membrane transport protein AcrB.</ns0:title>
<ns0:authorList>
<ns0:person name="Aoki S.K." />
<ns0:person name="Malinverni J.C." />
<ns0:person name="Jacoby K." />
<ns0:person name="Thomas B." />
<ns0:person name="Pamma R." />
<ns0:person name="Trinh B.N." />
<ns0:person name="Remers S." />
<ns0:person name="Webb J." />
<ns0:person name="Braaten B.A." />
<ns0:person name="Silhavy T.J." />
<ns0:person name="Low D.A." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="18761695" />
<ns0:dbReference type="DOI" id="10.1111/j.1365-2958.2008.06404.x" />
</ns0:citation>
<ns0:scope>FUNCTION (MICROBIAL INFECTION)</ns0:scope>
<ns0:scope>DISRUPTION PHENOTYPE</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MC4100</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="12">
<ns0:citation type="journal article" date="2012" name="Proc. Natl. Acad. Sci. U.S.A." volume="109" first="16696" last="16701">
<ns0:title>Conserved small protein associates with the multidrug efflux pump AcrB and differentially affects antibiotic resistance.</ns0:title>
<ns0:authorList>
<ns0:person name="Hobbs E.C." />
<ns0:person name="Yin X." />
<ns0:person name="Paul B.J." />
<ns0:person name="Astarita J.L." />
<ns0:person name="Storz G." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="23010927" />
<ns0:dbReference type="DOI" id="10.1073/pnas.1210093109" />
</ns0:citation>
<ns0:scope>FUNCTION</ns0:scope>
<ns0:scope>INTERACTION WITH ACRZ</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
<ns0:scope>INDUCTION</ns0:scope>
<ns0:scope>MUTAGENESIS OF HIS-526</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="13">
<ns0:citation type="journal article" date="2002" name="Nature" volume="419" first="587" last="593">
<ns0:title>Crystal structure of bacterial multidrug efflux transporter AcrB.</ns0:title>
<ns0:authorList>
<ns0:person name="Murakami S." />
<ns0:person name="Nakashima R." />
<ns0:person name="Yamashita E." />
<ns0:person name="Yamaguchi A." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="12374972" />
<ns0:dbReference type="DOI" id="10.1038/nature01050" />
</ns0:citation>
<ns0:scope>X-RAY CRYSTALLOGRAPHY (3.5 ANGSTROMS)</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
</ns0:reference>
<ns0:reference key="14">
<ns0:citation type="journal article" date="2003" name="Science" volume="300" first="976" last="980">
<ns0:title>Structural basis of multiple drug-binding capacity of the AcrB multidrug efflux pump.</ns0:title>
<ns0:authorList>
<ns0:person name="Yu E.W." />
<ns0:person name="McDermott G." />
<ns0:person name="Zgurskaya H.I." />
<ns0:person name="Nikaido H." />
<ns0:person name="Koshland D.E. Jr." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="12738864" />
<ns0:dbReference type="DOI" id="10.1126/science.1083137" />
</ns0:citation>
<ns0:scope>X-RAY CRYSTALLOGRAPHY (3.48 ANGSTROMS) IN COMPLEXES WITH SUBSTRATES</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
</ns0:reference>
<ns0:reference key="15">
<ns0:citation type="journal article" date="2006" name="Nature" volume="443" first="173" last="179">
<ns0:title>Crystal structures of a multidrug transporter reveal a functionally rotating mechanism.</ns0:title>
<ns0:authorList>
<ns0:person name="Murakami S." />
<ns0:person name="Nakashima R." />
<ns0:person name="Yamashita E." />
<ns0:person name="Matsumoto T." />
<ns0:person name="Yamaguchi A." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16915237" />
<ns0:dbReference type="DOI" id="10.1038/nature05076" />
</ns0:citation>
<ns0:scope>X-RAY CRYSTALLOGRAPHY (2.8 ANGSTROMS) IN COMPLEX WITH SUBSTRATE</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
<ns0:scope>FUNCTION</ns0:scope>
</ns0:reference>
<ns0:reference key="16">
<ns0:citation type="journal article" date="2006" name="Science" volume="313" first="1295" last="1298">
<ns0:title>Structural asymmetry of AcrB trimer suggests a peristaltic pump mechanism.</ns0:title>
<ns0:authorList>
<ns0:person name="Seeger M.A." />
<ns0:person name="Schiefner A." />
<ns0:person name="Eicher T." />
<ns0:person name="Verrey F." />
<ns0:person name="Diederichs K." />
<ns0:person name="Pos K.M." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16946072" />
<ns0:dbReference type="DOI" id="10.1126/science.1131542" />
</ns0:citation>
<ns0:scope>X-RAY CRYSTALLOGRAPHY (2.5 ANGSTROMS)</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
<ns0:scope>FUNCTION</ns0:scope>
</ns0:reference>
<ns0:reference key="17">
<ns0:citation type="journal article" date="2007" name="PLoS Biol." volume="5" first="107" last="113">
<ns0:title>Drug export pathway of multidrug exporter AcrB revealed by DARPin inhibitors.</ns0:title>
<ns0:authorList>
<ns0:person name="Sennhauser G." />
<ns0:person name="Amstutz P." />
<ns0:person name="Briand C." />
<ns0:person name="Storchenegger O." />
<ns0:person name="Gruetter M.G." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="17194213" />
<ns0:dbReference type="DOI" id="10.1371/journal.pbio.0050007" />
</ns0:citation>
<ns0:scope>X-RAY CRYSTALLOGRAPHY (2.9 ANGSTROMS)</ns0:scope>
<ns0:scope>FUNCTION</ns0:scope>
<ns0:scope>SUBUNIT</ns0:scope>
</ns0:reference>
<ns0:comment type="function">
<ns0:text evidence="23 24 25 27">AcrA-AcrB-AcrZ-TolC is a drug efflux protein complex with broad substrate specificity that uses the proton motive force to export substrates.</ns0:text>
</ns0:comment>
<ns0:comment type="function">
<ns0:text evidence="26">(Microbial infection) Involved in contact-dependent growth inhibition (CDI), acts downstream of BamA, the receptor for CDI. Its role in CDI is independent of the AcrA-AcrB-TolC efflux pump complex.</ns0:text>
</ns0:comment>
<ns0:comment type="subunit">
<ns0:text evidence="17 18 19 20 22 23 24 25 27">Homotrimer, with large domains that extend into the periplasm, interacts with AcrA and TolC. AcrA may be required to stably link this protein and TolC. Interacts with AcrZ. Part of the AcrA-AcrB-AcrZ-TolC efflux pump.</ns0:text>
</ns0:comment>
<ns0:comment type="interaction">
<ns0:interactant intactId="EBI-551006">
<ns0:id>P31224</ns0:id>
</ns0:interactant>
<ns0:interactant intactId="EBI-551006">
<ns0:id>P31224</ns0:id>
<ns0:label>acrB</ns0:label>
</ns0:interactant>
<ns0:organismsDiffer>false</ns0:organismsDiffer>
<ns0:experiments>9</ns0:experiments>
</ns0:comment>
<ns0:comment type="interaction">
<ns0:interactant intactId="EBI-551006">
<ns0:id>P31224</ns0:id>
</ns0:interactant>
<ns0:interactant intactId="EBI-6313593">
<ns0:id>P0AAW9</ns0:id>
<ns0:label>acrZ</ns0:label>
</ns0:interactant>
<ns0:organismsDiffer>false</ns0:organismsDiffer>
<ns0:experiments>7</ns0:experiments>
</ns0:comment>
<ns0:comment type="interaction">
<ns0:interactant intactId="EBI-551006">
<ns0:id>P31224</ns0:id>
</ns0:interactant>
<ns0:interactant intactId="EBI-1130723">
<ns0:id>P0ADZ7</ns0:id>
<ns0:label>yajC</ns0:label>
</ns0:interactant>
<ns0:organismsDiffer>false</ns0:organismsDiffer>
<ns0:experiments>2</ns0:experiments>
</ns0:comment>
<ns0:comment type="subcellular location">
<ns0:subcellularLocation>
<ns0:location evidence="20 22">Cell inner membrane</ns0:location>
<ns0:topology evidence="20 22">Multi-pass membrane protein</ns0:topology>
</ns0:subcellularLocation>
</ns0:comment>
<ns0:comment type="induction">
<ns0:text evidence="27">Positively regulated by MarA, Rob and SoxS transcriptional regulators (at protein level).</ns0:text>
</ns0:comment>
<ns0:comment type="disruption phenotype">
<ns0:text evidence="26">Loss of susceptibility to contact-dependent growth inhibition (CDI); inhibiting cells still contact the target.</ns0:text>
</ns0:comment>
<ns0:comment type="similarity">
<ns0:text evidence="28">Belongs to the resistance-nodulation-cell division (RND) (TC 2.A.6) family.</ns0:text>
</ns0:comment>
<ns0:dbReference type="EMBL" id="M94248">
<ns0:property type="protein sequence ID" value="AAA23411.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U00734">
<ns0:property type="protein sequence ID" value="AAA67135.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U82664">
<ns0:property type="protein sequence ID" value="AAB40216.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U00096">
<ns0:property type="protein sequence ID" value="AAC73564.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="AP009048">
<ns0:property type="protein sequence ID" value="BAE76241.1" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="PIR" id="B36938">
<ns0:property type="entry name" value="B36938" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="NP_414995.1">
<ns0:property type="nucleotide sequence ID" value="NC_000913.3" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="WP_001132469.1">
<ns0:property type="nucleotide sequence ID" value="NZ_STEB01000007.1" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1IWG">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.50" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1OY6">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.68" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1OY8">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.63" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1OY9">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.80" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1OYD">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.80" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1OYE">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.48" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9T">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.23" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9U">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.11" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9V">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.80" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9W">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.23" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9X">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.08" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="1T9Y">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.64" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2DHH">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.80" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2DR6">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.30" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2DRD">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.10" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2GIF">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.90" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2HQC">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.56" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2HQD">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.65" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2HQF">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.38" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2HQG">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.38" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2HRT">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.00" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2I6W">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.10" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2J8S">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.54" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2RDD">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.50" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="2W1B">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.85" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3AOA">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.35" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3AOB">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.35" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3AOC">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.34" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3AOD">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.30" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3D9B">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.42" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3NOC">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.70" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3NOG">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.34" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="3W9H">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.05" />
<ns0:property type="chains" value="A/B/C=1-1033" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4C48">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.30" />
<ns0:property type="chains" value="A=1-1047" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4CDI">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.70" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4DX5">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="1.90" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4DX6">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.90" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4DX7">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.25" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4K7Q">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.50" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4U8V">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.30" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4U8Y">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.10" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4U95">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.00" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4U96">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.20" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZIT">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.30" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZIV">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.16" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZIW">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.40" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZJL">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.47" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZJO">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.60" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZJQ">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.59" />
<ns0:property type="chains" value="A/B/C/D/E/F=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZLJ">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.26" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZLL">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.36" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="4ZLN">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.56" />
<ns0:property type="chains" value="A=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5EN5">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.30" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENO">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.20" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENP">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="1.90" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENQ">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="1.80" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENR">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.30" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENS">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.80" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5ENT">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.50" />
<ns0:property type="chains" value="A/B/C=39-329, A/B/C=561-869" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5JMN">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.50" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5NC5">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.20" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5NG5">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="6.50" />
<ns0:property type="chains" value="J/K/L=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5O66">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="5.90" />
<ns0:property type="chains" value="J/K/L=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5V5S">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="6.50" />
<ns0:property type="chains" value="J/K/L=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5YIL">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="3.00" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6BAJ">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.20" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6CSX">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.00" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6Q4N">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.80" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6Q4O">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.80" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6Q4P">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.80" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6SGR">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.17" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6SGS">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.20" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6SGT">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.46" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="6SGU">
<ns0:property type="method" value="EM" />
<ns0:property type="resolution" value="3.27" />
<ns0:property type="chains" value="A/B/C=1-1049" />
</ns0:dbReference>
<ns0:dbReference type="PDBsum" id="1IWG" />
<ns0:dbReference type="PDBsum" id="1OY6" />
<ns0:dbReference type="PDBsum" id="1OY8" />
<ns0:dbReference type="PDBsum" id="1OY9" />
<ns0:dbReference type="PDBsum" id="1OYD" />
<ns0:dbReference type="PDBsum" id="1OYE" />
<ns0:dbReference type="PDBsum" id="1T9T" />
<ns0:dbReference type="PDBsum" id="1T9U" />
<ns0:dbReference type="PDBsum" id="1T9V" />
<ns0:dbReference type="PDBsum" id="1T9W" />
<ns0:dbReference type="PDBsum" id="1T9X" />
<ns0:dbReference type="PDBsum" id="1T9Y" />
<ns0:dbReference type="PDBsum" id="2DHH" />
<ns0:dbReference type="PDBsum" id="2DR6" />
<ns0:dbReference type="PDBsum" id="2DRD" />
<ns0:dbReference type="PDBsum" id="2GIF" />
<ns0:dbReference type="PDBsum" id="2HQC" />
<ns0:dbReference type="PDBsum" id="2HQD" />
<ns0:dbReference type="PDBsum" id="2HQF" />
<ns0:dbReference type="PDBsum" id="2HQG" />
<ns0:dbReference type="PDBsum" id="2HRT" />
<ns0:dbReference type="PDBsum" id="2I6W" />
<ns0:dbReference type="PDBsum" id="2J8S" />
<ns0:dbReference type="PDBsum" id="2RDD" />
<ns0:dbReference type="PDBsum" id="2W1B" />
<ns0:dbReference type="PDBsum" id="3AOA" />
<ns0:dbReference type="PDBsum" id="3AOB" />
<ns0:dbReference type="PDBsum" id="3AOC" />
<ns0:dbReference type="PDBsum" id="3AOD" />
<ns0:dbReference type="PDBsum" id="3D9B" />
<ns0:dbReference type="PDBsum" id="3NOC" />
<ns0:dbReference type="PDBsum" id="3NOG" />
<ns0:dbReference type="PDBsum" id="3W9H" />
<ns0:dbReference type="PDBsum" id="4C48" />
<ns0:dbReference type="PDBsum" id="4CDI" />
<ns0:dbReference type="PDBsum" id="4DX5" />
<ns0:dbReference type="PDBsum" id="4DX6" />
<ns0:dbReference type="PDBsum" id="4DX7" />
<ns0:dbReference type="PDBsum" id="4K7Q" />
<ns0:dbReference type="PDBsum" id="4U8V" />
<ns0:dbReference type="PDBsum" id="4U8Y" />
<ns0:dbReference type="PDBsum" id="4U95" />
<ns0:dbReference type="PDBsum" id="4U96" />
<ns0:dbReference type="PDBsum" id="4ZIT" />
<ns0:dbReference type="PDBsum" id="4ZIV" />
<ns0:dbReference type="PDBsum" id="4ZIW" />
<ns0:dbReference type="PDBsum" id="4ZJL" />
<ns0:dbReference type="PDBsum" id="4ZJO" />
<ns0:dbReference type="PDBsum" id="4ZJQ" />
<ns0:dbReference type="PDBsum" id="4ZLJ" />
<ns0:dbReference type="PDBsum" id="4ZLL" />
<ns0:dbReference type="PDBsum" id="4ZLN" />
<ns0:dbReference type="PDBsum" id="5EN5" />
<ns0:dbReference type="PDBsum" id="5ENO" />
<ns0:dbReference type="PDBsum" id="5ENP" />
<ns0:dbReference type="PDBsum" id="5ENQ" />
<ns0:dbReference type="PDBsum" id="5ENR" />
<ns0:dbReference type="PDBsum" id="5ENS" />
<ns0:dbReference type="PDBsum" id="5ENT" />
<ns0:dbReference type="PDBsum" id="5JMN" />
<ns0:dbReference type="PDBsum" id="5NC5" />
<ns0:dbReference type="PDBsum" id="5NG5" />
<ns0:dbReference type="PDBsum" id="5O66" />
<ns0:dbReference type="PDBsum" id="5V5S" />
<ns0:dbReference type="PDBsum" id="5YIL" />
<ns0:dbReference type="PDBsum" id="6BAJ" />
<ns0:dbReference type="PDBsum" id="6CSX" />
<ns0:dbReference type="PDBsum" id="6Q4N" />
<ns0:dbReference type="PDBsum" id="6Q4O" />
<ns0:dbReference type="PDBsum" id="6Q4P" />
<ns0:dbReference type="PDBsum" id="6SGR" />
<ns0:dbReference type="PDBsum" id="6SGS" />
<ns0:dbReference type="PDBsum" id="6SGT" />
<ns0:dbReference type="PDBsum" id="6SGU" />
<ns0:dbReference type="SMR" id="P31224" />
<ns0:dbReference type="BioGRID" id="4259859">
<ns0:property type="interactions" value="386" />
</ns0:dbReference>
<ns0:dbReference type="ComplexPortal" id="CPX-4263">
<ns0:property type="entry name" value="AcrAB-TolC multidrug efflux transport complex" />
</ns0:dbReference>
<ns0:dbReference type="DIP" id="DIP-9049N" />
<ns0:dbReference type="IntAct" id="P31224">
<ns0:property type="interactions" value="9" />
</ns0:dbReference>
<ns0:dbReference type="MINT" id="P31224" />
<ns0:dbReference type="STRING" id="511145.b0462" />
<ns0:dbReference type="ChEMBL" id="CHEMBL1681614" />
<ns0:dbReference type="DrugBank" id="DB03619">
<ns0:property type="generic name" value="Deoxycholic acid" />
</ns0:dbReference>
<ns0:dbReference type="DrugBank" id="DB04209">
<ns0:property type="generic name" value="Dequalinium" />
</ns0:dbReference>
<ns0:dbReference type="DrugBank" id="DB03825">
<ns0:property type="generic name" value="Rhodamine 6G" />
</ns0:dbReference>
<ns0:dbReference type="TCDB" id="2.A.6.2.2">
<ns0:property type="family name" value="the resistance-nodulation-cell division (rnd) superfamily" />
</ns0:dbReference>
<ns0:dbReference type="jPOST" id="P31224" />
<ns0:dbReference type="PaxDb" id="P31224" />
<ns0:dbReference type="PRIDE" id="P31224" />
<ns0:dbReference type="EnsemblBacteria" id="AAC73564">
<ns0:property type="protein sequence ID" value="AAC73564" />
<ns0:property type="gene ID" value="b0462" />
</ns0:dbReference>
<ns0:dbReference type="EnsemblBacteria" id="BAE76241">
<ns0:property type="protein sequence ID" value="BAE76241" />
<ns0:property type="gene ID" value="BAE76241" />
</ns0:dbReference>
<ns0:dbReference type="GeneID" id="49585741" />
<ns0:dbReference type="GeneID" id="945108" />
<ns0:dbReference type="KEGG" id="ecj:JW0451" />
<ns0:dbReference type="KEGG" id="eco:b0462" />
<ns0:dbReference type="PATRIC" id="fig|1411691.4.peg.1814" />
<ns0:dbReference type="EchoBASE" id="EB1655" />
<ns0:dbReference type="eggNOG" id="COG0841">
<ns0:property type="taxonomic scope" value="Bacteria" />
</ns0:dbReference>
<ns0:dbReference type="HOGENOM" id="CLU_002755_0_0_6" />
<ns0:dbReference type="InParanoid" id="P31224" />
<ns0:dbReference type="KO" id="K18138" />
<ns0:dbReference type="PhylomeDB" id="P31224" />
<ns0:dbReference type="BioCyc" id="EcoCyc:ACRB-MONOMER" />
<ns0:dbReference type="BioCyc" id="ECOL316407:JW0451-MONOMER" />
<ns0:dbReference type="BioCyc" id="MetaCyc:ACRB-MONOMER" />
<ns0:dbReference type="EvolutionaryTrace" id="P31224" />
<ns0:dbReference type="PRO" id="PR:P31224" />
<ns0:dbReference type="Proteomes" id="UP000000318">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="Proteomes" id="UP000000625">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:1990281">
<ns0:property type="term" value="C:efflux pump complex" />
<ns0:property type="evidence" value="ECO:0000353" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0005887">
<ns0:property type="term" value="C:integral component of plasma membrane" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0016020">
<ns0:property type="term" value="C:membrane" />
<ns0:property type="evidence" value="ECO:0007005" />
<ns0:property type="project" value="UniProtKB" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0030288">
<ns0:property type="term" value="C:outer membrane-bounded periplasmic space" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0005886">
<ns0:property type="term" value="C:plasma membrane" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0015562">
<ns0:property type="term" value="F:efflux transmembrane transporter activity" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="InterPro" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0042802">
<ns0:property type="term" value="F:identical protein binding" />
<ns0:property type="evidence" value="ECO:0000353" />
<ns0:property type="project" value="IntAct" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0042908">
<ns0:property type="term" value="P:xenobiotic transport" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="InterPro" />
</ns0:dbReference>
<ns0:dbReference type="Gene3D" id="3.30.2090.10">
<ns0:property type="match status" value="2" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR027463">
<ns0:property type="entry name" value="AcrB_DN_DC_subdom" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR001036">
<ns0:property type="entry name" value="Acrflvin-R" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR004764">
<ns0:property type="entry name" value="HAE1" />
</ns0:dbReference>
<ns0:dbReference type="PANTHER" id="PTHR32063">
<ns0:property type="entry name" value="PTHR32063" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="Pfam" id="PF00873">
<ns0:property type="entry name" value="ACR_tran" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="PRINTS" id="PR00702">
<ns0:property type="entry name" value="ACRIFLAVINRP" />
</ns0:dbReference>
<ns0:dbReference type="SUPFAM" id="SSF82714">
<ns0:property type="entry name" value="SSF82714" />
<ns0:property type="match status" value="2" />
</ns0:dbReference>
<ns0:dbReference type="TIGRFAMs" id="TIGR00915">
<ns0:property type="entry name" value="2A0602" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:proteinExistence type="evidence at protein level" />
<ns0:keyword id="KW-0002">3D-structure</ns0:keyword>
<ns0:keyword id="KW-0997">Cell inner membrane</ns0:keyword>
<ns0:keyword id="KW-1003">Cell membrane</ns0:keyword>
<ns0:keyword id="KW-0472">Membrane</ns0:keyword>
<ns0:keyword id="KW-1185">Reference proteome</ns0:keyword>
<ns0:keyword id="KW-0812">Transmembrane</ns0:keyword>
<ns0:keyword id="KW-1133">Transmembrane helix</ns0:keyword>
<ns0:keyword id="KW-0813">Transport</ns0:keyword>
<ns0:feature type="chain" description="Multidrug efflux pump subunit AcrB" id="PRO_0000161811">
<ns0:location>
<ns0:begin position="1" />
<ns0:end position="1049" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="1" />
<ns0:end position="9" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=1">
<ns0:location>
<ns0:begin position="10" />
<ns0:end position="28" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="29" />
<ns0:end position="336" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=2">
<ns0:location>
<ns0:begin position="337" />
<ns0:end position="356" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="357" />
<ns0:end position="365" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=3">
<ns0:location>
<ns0:begin position="366" />
<ns0:end position="385" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="386" />
<ns0:end position="391" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=4">
<ns0:location>
<ns0:begin position="392" />
<ns0:end position="413" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="414" />
<ns0:end position="438" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=5">
<ns0:location>
<ns0:begin position="439" />
<ns0:end position="457" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="458" />
<ns0:end position="465" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=6">
<ns0:location>
<ns0:begin position="466" />
<ns0:end position="490" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="491" />
<ns0:end position="538" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=7">
<ns0:location>
<ns0:begin position="539" />
<ns0:end position="555" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="556" />
<ns0:end position="871" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=8">
<ns0:location>
<ns0:begin position="872" />
<ns0:end position="888" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="889" />
<ns0:end position="898" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=9">
<ns0:location>
<ns0:begin position="899" />
<ns0:end position="918" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="919" />
<ns0:end position="924" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=10">
<ns0:location>
<ns0:begin position="925" />
<ns0:end position="943" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="944" />
<ns0:end position="972" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=11">
<ns0:location>
<ns0:begin position="973" />
<ns0:end position="992" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Periplasmic" evidence="21">
<ns0:location>
<ns0:begin position="993" />
<ns0:end position="998" />
</ns0:location>
</ns0:feature>
<ns0:feature type="transmembrane region" description="Helical; Name=12">
<ns0:location>
<ns0:begin position="999" />
<ns0:end position="1018" />
</ns0:location>
</ns0:feature>
<ns0:feature type="topological domain" description="Cytoplasmic" evidence="21">
<ns0:location>
<ns0:begin position="1019" />
<ns0:end position="1049" />
</ns0:location>
</ns0:feature>
<ns0:feature type="mutagenesis site" description="Partially restores chloramphenicol resistance to an AcrZ G30R mutant." evidence="27">
<ns0:original>H</ns0:original>
<ns0:variation>Y</ns0:variation>
<ns0:location>
<ns0:position position="526" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="2" />
<ns0:end position="6" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="9" />
<ns0:end position="29" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="5">
<ns0:location>
<ns0:begin position="32" />
<ns0:end position="35" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="42" />
<ns0:end position="48" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="54" />
<ns0:end position="60" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="62" />
<ns0:end position="66" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="75" />
<ns0:end position="83" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="86" />
<ns0:end position="94" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="4">
<ns0:location>
<ns0:begin position="95" />
<ns0:end position="97" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="100" />
<ns0:end position="114" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="115" />
<ns0:end position="117" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="120" />
<ns0:end position="123" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="128" />
<ns0:end position="130" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="132" />
<ns0:end position="144" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="9">
<ns0:location>
<ns0:begin position="145" />
<ns0:end position="147" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="151" />
<ns0:end position="161" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="163" />
<ns0:end position="168" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="2">
<ns0:location>
<ns0:begin position="169" />
<ns0:end position="171" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="172" />
<ns0:end position="179" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="182" />
<ns0:end position="188" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="190" />
<ns0:end position="195" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="200" />
<ns0:end position="210" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="215" />
<ns0:end position="220" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="232" />
<ns0:end position="235" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="243" />
<ns0:end position="247" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="250" />
<ns0:end position="253" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="3">
<ns0:location>
<ns0:begin position="255" />
<ns0:end position="257" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="259" />
<ns0:end position="261" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="262" />
<ns0:end position="264" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="266" />
<ns0:end position="273" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="278" />
<ns0:end position="281" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="284" />
<ns0:end position="294" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="3">
<ns0:location>
<ns0:begin position="295" />
<ns0:end position="298" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="299" />
<ns0:end position="313" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="314" />
<ns0:end position="316" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="3">
<ns0:location>
<ns0:begin position="317" />
<ns0:end position="319" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="321" />
<ns0:end position="328" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="330" />
<ns0:end position="359" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="362" />
<ns0:end position="385" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="392" />
<ns0:end position="423" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="427" />
<ns0:end position="453" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="455" />
<ns0:end position="458" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="461" />
<ns0:end position="496" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="5">
<ns0:location>
<ns0:begin position="505" />
<ns0:end position="508" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="16">
<ns0:location>
<ns0:begin position="509" />
<ns0:end position="511" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="512" />
<ns0:end position="536" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="8">
<ns0:location>
<ns0:begin position="537" />
<ns0:end position="539" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="540" />
<ns0:end position="558" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="4">
<ns0:location>
<ns0:begin position="561" />
<ns0:end position="564" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="571" />
<ns0:end position="577" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="15">
<ns0:location>
<ns0:begin position="579" />
<ns0:end position="581" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="584" />
<ns0:end position="599" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="13">
<ns0:location>
<ns0:begin position="600" />
<ns0:end position="605" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="606" />
<ns0:end position="616" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="619" />
<ns0:end position="631" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="634" />
<ns0:end position="636" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="640" />
<ns0:end position="642" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="644" />
<ns0:end position="655" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="7">
<ns0:location>
<ns0:begin position="658" />
<ns0:end position="660" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="662" />
<ns0:end position="666" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="672" />
<ns0:end position="674" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="15">
<ns0:location>
<ns0:begin position="675" />
<ns0:end position="677" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="679" />
<ns0:end position="686" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="692" />
<ns0:end position="707" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="13">
<ns0:location>
<ns0:begin position="710" />
<ns0:end position="712" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="713" />
<ns0:end position="720" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="724" />
<ns0:end position="731" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="733" />
<ns0:end position="739" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="743" />
<ns0:end position="755" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="757" />
<ns0:end position="764" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="767" />
<ns0:end position="775" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="777" />
<ns0:end position="779" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="780" />
<ns0:end position="782" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="783" />
<ns0:end position="788" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="790" />
<ns0:end position="792" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="7">
<ns0:location>
<ns0:begin position="794" />
<ns0:end position="796" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="798" />
<ns0:end position="800" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="801" />
<ns0:end position="803" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="804" />
<ns0:end position="812" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="814" />
<ns0:end position="819" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="822" />
<ns0:end position="831" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="7">
<ns0:location>
<ns0:begin position="833" />
<ns0:end position="835" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="837" />
<ns0:end position="848" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="13">
<ns0:location>
<ns0:begin position="855" />
<ns0:end position="859" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="13">
<ns0:location>
<ns0:begin position="861" />
<ns0:end position="863" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="11">
<ns0:location>
<ns0:begin position="865" />
<ns0:end position="868" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="1">
<ns0:location>
<ns0:begin position="869" />
<ns0:end position="871" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="873" />
<ns0:end position="892" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="14">
<ns0:location>
<ns0:begin position="894" />
<ns0:end position="897" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="898" />
<ns0:end position="902" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="905" />
<ns0:end position="919" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="925" />
<ns0:end position="954" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="10">
<ns0:location>
<ns0:begin position="955" />
<ns0:end position="957" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="960" />
<ns0:end position="985" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="987" />
<ns0:end position="990" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="6">
<ns0:location>
<ns0:begin position="994" />
<ns0:end position="996" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="11">
<ns0:location>
<ns0:begin position="997" />
<ns0:end position="1032" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="12">
<ns0:location>
<ns0:begin position="1033" />
<ns0:end position="1035" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="5">
<ns0:location>
<ns0:begin position="1036" />
<ns0:end position="1038" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="6">
<ns0:location>
<ns0:begin position="1039" />
<ns0:end position="1041" />
</ns0:location>
</ns0:feature>
<ns0:evidence key="1" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="1T9W" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="2" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="1T9X" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="3" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="2DHH" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="4" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="2DR6" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="5" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="2GIF" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="6" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="2J8S" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="7" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="3NOC" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="8" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="3NOG" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="9" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="3W9H" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="10" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="4C48" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="11" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="4DX5" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="12" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="4K7Q" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="13" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="5ENQ" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="14" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="5YIL" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="15" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="6BAJ" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="16" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="6Q4P" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="17" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="10920254" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="18" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="12374972" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="19" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="12738864" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="20" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="15228545" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="21" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="15919996" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="22" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="16079137" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="23" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="16915237" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="24" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="16946072" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="25" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="17194213" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="26" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="18761695" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="27" type="ECO:0000269">
<ns0:source>
<ns0:dbReference type="PubMed" id="23010927" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="28" type="ECO:0000305" />
<ns0:sequence length="1049" mass="113574" checksum="19670E3C4CC29055" modified="1993-07-01" version="1">MPNFFIDRPIFAWVIAIIIMLAGGLAILKLPVAQYPTIAPPAVTISASYPGADAKTVQDTVTQVIEQNMNGIDNLMYMSSNSDSTGTVQITLTFESGTDADIAQVQVQNKLQLAMPLLPQEVQQQGVSVEKSSSSFLMVVGVINTDGTMTQEDISDYVAANMKDAISRTSGVGDVQLFGSQYAMRIWMNPNELNKFQLTPVDVITAIKAQNAQVAAGQLGGTPPVKGQQLNASIIAQTRLTSTEEFGKILLKVNQDGSRVLLRDVAKIELGGENYDIIAEFNGQPASGLGIKLATGANALDTAAAIRAELAKMEPFFPSGLKIVYPYDTTPFVKISIHEVVKTLVEAIILVFLVMYLFLQNFRATLIPTIAVPVVLLGTFAVLAAFGFSINTLTMFGMVLAIGLLVDDAIVVVENVERVMAEEGLPPKEATRKSMGQIQGALVGIAMVLSAVFVPMAFFGGSTGAIYRQFSITIVSAMALSVLVALILTPALCATMLKPIAKGDHGEGKKGFFGWFNRMFEKSTHHYTDSVGGILRSTGRYLVLYLIIVVGMAYLFVRLPSSFLPDEDQGVFMTMVQLPAGATQERTQKVLNEVTHYYLTKEKNNVESVFAVNGFGFAGRGQNTGIAFVSLKDWADRPGEENKVEAITMRATRAFSQIKDAMVFAFNLPAIVELGTATGFDFELIDQAGLGHEKLTQARNQLLAEAAKHPDMLTSVRPNGLEDTPQFKIDIDQEKAQALGVSINDINTTLGAAWGGSYVNDFIDRGRVKKVYVMSEAKYRMLPDDIGDWYVRAADGQMVPFSAFSSSRWEYGSPRLERYNGLPSMEILGQAAPGKSTGEAMELMEQLASKLPTGVGYDWTGMSYQERLSGNQAPSLYAISLIVVFLCLAALYESWSIPFSVMLVVPLGVIGALLAATFRGLTNDVYFQVGLLTTIGLSAKNAILIVEFAKDLMDKEGKGLIEATLDAVRMRLRPILMTSLAFILGVMPLVISTGAGSGAQNAVGTGVMGGMVTATVLAIFFVPVFFVVVRRRFSRKNEDIEHSHTVDHH</ns0:sequence>
</ns0:entry>
```


Cherchez par exemple le sous arbre de la protéine nommée `DACD_ECOLI`

```python
# Find the xml subtree of a protein with name "DACD_ECOLI"
for entry in proteins:
    names = entry.findall(ns+"name")
    for n in names:
        if n.text == "DACD_ECOLI":
            dump(entry)
            break
```            
Out: 
```xml
<ns0:entry xmlns:ns0="http://uniprot.org/uniprot" dataset="Swiss-Prot" created="1994-02-01" modified="2020-08-12" version="163">
<ns0:accession>P33013</ns0:accession>
<ns0:name>DACD_ECOLI</ns0:name>
<ns0:protein>
<ns0:recommendedName>
<ns0:fullName>D-alanyl-D-alanine carboxypeptidase DacD</ns0:fullName>
<ns0:shortName>DD-carboxypeptidase</ns0:shortName>
<ns0:shortName>DD-peptidase</ns0:shortName>
<ns0:ecNumber>3.4.16.4</ns0:ecNumber>
</ns0:recommendedName>
<ns0:alternativeName>
<ns0:fullName>Penicillin-binding protein 6b</ns0:fullName>
<ns0:shortName>PBP-6b</ns0:shortName>
</ns0:alternativeName>
</ns0:protein>
<ns0:gene>
<ns0:name type="primary">dacD</ns0:name>
<ns0:name type="synonym">phsE</ns0:name>
<ns0:name type="synonym">yeeC</ns0:name>
<ns0:name type="ordered locus">b2010</ns0:name>
<ns0:name type="ordered locus">JW5329</ns0:name>
</ns0:gene>
<ns0:organism>
<ns0:name type="scientific">Escherichia coli (strain K12)</ns0:name>
<ns0:dbReference type="NCBI Taxonomy" id="83333" />
<ns0:lineage>
<ns0:taxon>Bacteria</ns0:taxon>
<ns0:taxon>Proteobacteria</ns0:taxon>
<ns0:taxon>Gammaproteobacteria</ns0:taxon>
<ns0:taxon>Enterobacterales</ns0:taxon>
<ns0:taxon>Enterobacteriaceae</ns0:taxon>
<ns0:taxon>Escherichia</ns0:taxon>
</ns0:lineage>
</ns0:organism>
<ns0:reference key="1">
<ns0:citation type="journal article" date="1996" name="J. Bacteriol." volume="178" first="7106" last="7111">
<ns0:title>dacD, an Escherichia coli gene encoding a novel penicillin-binding protein (PBP6b) with DD-carboxypeptidase activity.</ns0:title>
<ns0:authorList>
<ns0:person name="Baquero M.-R." />
<ns0:person name="Bouzon M." />
<ns0:person name="Quintela J.C." />
<ns0:person name="Ayala J.A." />
<ns0:person name="Moreno F." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="8955390" />
<ns0:dbReference type="DOI" id="10.1128/jb.178.24.7106-7111.1996" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [GENOMIC DNA]</ns0:scope>
<ns0:scope>CHARACTERIZATION</ns0:scope>
</ns0:reference>
<ns0:reference key="2">
<ns0:citation type="submission" date="1993-10" db="EMBL/GenBank/DDBJ databases">
<ns0:title>Automated multiplex sequencing of the E.coli genome.</ns0:title>
<ns0:authorList>
<ns0:person name="Richterich P." />
<ns0:person name="Lakey N." />
<ns0:person name="Gryan G." />
<ns0:person name="Jaehn L." />
<ns0:person name="Mintz L." />
<ns0:person name="Robison K." />
<ns0:person name="Church G.M." />
</ns0:authorList>
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / BHB2600</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="3">
<ns0:citation type="journal article" date="1996" name="DNA Res." volume="3" first="379" last="392">
<ns0:title>A 460-kb DNA sequence of the Escherichia coli K-12 genome corresponding to the 40.1-50.0 min region on the linkage map.</ns0:title>
<ns0:authorList>
<ns0:person name="Itoh T." />
<ns0:person name="Aiba H." />
<ns0:person name="Baba T." />
<ns0:person name="Fujita K." />
<ns0:person name="Hayashi K." />
<ns0:person name="Inada T." />
<ns0:person name="Isono K." />
<ns0:person name="Kasai H." />
<ns0:person name="Kimura S." />
<ns0:person name="Kitakawa M." />
<ns0:person name="Kitagawa M." />
<ns0:person name="Makino K." />
<ns0:person name="Miki T." />
<ns0:person name="Mizobuchi K." />
<ns0:person name="Mori H." />
<ns0:person name="Mori T." />
<ns0:person name="Motomura K." />
<ns0:person name="Nakade S." />
<ns0:person name="Nakamura Y." />
<ns0:person name="Nashimoto H." />
<ns0:person name="Nishio Y." />
<ns0:person name="Oshima T." />
<ns0:person name="Saito N." />
<ns0:person name="Sampei G." />
<ns0:person name="Seki Y." />
<ns0:person name="Sivasundaram S." />
<ns0:person name="Tagami H." />
<ns0:person name="Takeda J." />
<ns0:person name="Takemoto K." />
<ns0:person name="Wada C." />
<ns0:person name="Yamamoto Y." />
<ns0:person name="Horiuchi T." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="9097040" />
<ns0:dbReference type="DOI" id="10.1093/dnares/3.6.379" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / W3110 / ATCC 27325 / DSM 5911</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="4">
<ns0:citation type="journal article" date="1997" name="Science" volume="277" first="1453" last="1462">
<ns0:title>The complete genome sequence of Escherichia coli K-12.</ns0:title>
<ns0:authorList>
<ns0:person name="Blattner F.R." />
<ns0:person name="Plunkett G. III" />
<ns0:person name="Bloch C.A." />
<ns0:person name="Perna N.T." />
<ns0:person name="Burland V." />
<ns0:person name="Riley M." />
<ns0:person name="Collado-Vides J." />
<ns0:person name="Glasner J.D." />
<ns0:person name="Rode C.K." />
<ns0:person name="Mayhew G.F." />
<ns0:person name="Gregor J." />
<ns0:person name="Davis N.W." />
<ns0:person name="Kirkpatrick H.A." />
<ns0:person name="Goeden M.A." />
<ns0:person name="Rose D.J." />
<ns0:person name="Mau B." />
<ns0:person name="Shao Y." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="9278503" />
<ns0:dbReference type="DOI" id="10.1126/science.277.5331.1453" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / MG1655 / ATCC 47076</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:reference key="5">
<ns0:citation type="journal article" date="2006" name="Mol. Syst. Biol." volume="2" first="E1" last="E5">
<ns0:title>Highly accurate genome sequences of Escherichia coli K-12 strains MG1655 and W3110.</ns0:title>
<ns0:authorList>
<ns0:person name="Hayashi K." />
<ns0:person name="Morooka N." />
<ns0:person name="Yamamoto Y." />
<ns0:person name="Fujita K." />
<ns0:person name="Isono K." />
<ns0:person name="Choi S." />
<ns0:person name="Ohtsubo E." />
<ns0:person name="Baba T." />
<ns0:person name="Wanner B.L." />
<ns0:person name="Mori H." />
<ns0:person name="Horiuchi T." />
</ns0:authorList>
<ns0:dbReference type="PubMed" id="16738553" />
<ns0:dbReference type="DOI" id="10.1038/msb4100049" />
</ns0:citation>
<ns0:scope>NUCLEOTIDE SEQUENCE [LARGE SCALE GENOMIC DNA]</ns0:scope>
<ns0:source>
<ns0:strain>K12 / W3110 / ATCC 27325 / DSM 5911</ns0:strain>
</ns0:source>
</ns0:reference>
<ns0:comment type="function">
<ns0:text>Removes C-terminal D-alanyl residues from sugar-peptide cell wall precursors.</ns0:text>
</ns0:comment>
<ns0:comment type="catalytic activity">
<ns0:reaction>
<ns0:text>Preferential cleavage: (Ac)(2)-L-Lys-D-Ala-|-D-Ala. Also transpeptidation of peptidyl-alanyl moieties that are N-acyl substituents of D-alanine.</ns0:text>
<ns0:dbReference type="EC" id="3.4.16.4" />
</ns0:reaction>
</ns0:comment>
<ns0:comment type="pathway">
<ns0:text>Cell wall biogenesis; peptidoglycan biosynthesis.</ns0:text>
</ns0:comment>
<ns0:comment type="subcellular location">
<ns0:subcellularLocation>
<ns0:location evidence="4">Cell inner membrane</ns0:location>
<ns0:topology evidence="4">Peripheral membrane protein</ns0:topology>
</ns0:subcellularLocation>
<ns0:text evidence="4">N-terminal lies in the periplasmic space.</ns0:text>
</ns0:comment>
<ns0:comment type="similarity">
<ns0:text evidence="4">Belongs to the peptidase S11 family.</ns0:text>
</ns0:comment>
<ns0:comment type="sequence caution" evidence="4">
<ns0:conflict type="erroneous initiation">
<ns0:sequence resource="EMBL-CDS" id="AAA16416" version="1" />
</ns0:conflict>
<ns0:text>Truncated N-terminus.</ns0:text>
</ns0:comment>
<ns0:dbReference type="EC" id="3.4.16.4" />
<ns0:dbReference type="EMBL" id="U00009">
<ns0:property type="protein sequence ID" value="AAA16416.1" />
<ns0:property type="status" value="ALT_INIT" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="U00096">
<ns0:property type="protein sequence ID" value="AAC75071.2" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="EMBL" id="AP009048">
<ns0:property type="protein sequence ID" value="BAA15838.2" />
<ns0:property type="molecule type" value="Genomic_DNA" />
</ns0:dbReference>
<ns0:dbReference type="PIR" id="A64966">
<ns0:property type="entry name" value="A64966" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="NP_416514.4">
<ns0:property type="nucleotide sequence ID" value="NC_000913.3" />
</ns0:dbReference>
<ns0:dbReference type="RefSeq" id="WP_000830156.1">
<ns0:property type="nucleotide sequence ID" value="NZ_LN832404.1" />
</ns0:dbReference>
<ns0:dbReference type="PDB" id="5FSR">
<ns0:property type="method" value="X-ray" />
<ns0:property type="resolution" value="2.40" />
<ns0:property type="chains" value="A/B=22-374" />
</ns0:dbReference>
<ns0:dbReference type="PDBsum" id="5FSR" />
<ns0:dbReference type="SMR" id="P33013" />
<ns0:dbReference type="BioGRID" id="4260413">
<ns0:property type="interactions" value="266" />
</ns0:dbReference>
<ns0:dbReference type="STRING" id="511145.b2010" />
<ns0:dbReference type="MEROPS" id="S11.009" />
<ns0:dbReference type="PaxDb" id="P33013" />
<ns0:dbReference type="PRIDE" id="P33013" />
<ns0:dbReference type="EnsemblBacteria" id="AAC75071">
<ns0:property type="protein sequence ID" value="AAC75071" />
<ns0:property type="gene ID" value="b2010" />
</ns0:dbReference>
<ns0:dbReference type="EnsemblBacteria" id="BAA15838">
<ns0:property type="protein sequence ID" value="BAA15838" />
<ns0:property type="gene ID" value="BAA15838" />
</ns0:dbReference>
<ns0:dbReference type="GeneID" id="946518" />
<ns0:dbReference type="KEGG" id="ecj:JW5329" />
<ns0:dbReference type="KEGG" id="eco:b2010" />
<ns0:dbReference type="PATRIC" id="fig|1411691.4.peg.242" />
<ns0:dbReference type="EchoBASE" id="EB1839" />
<ns0:dbReference type="eggNOG" id="COG1686">
<ns0:property type="taxonomic scope" value="Bacteria" />
</ns0:dbReference>
<ns0:dbReference type="HOGENOM" id="CLU_027070_8_1_6" />
<ns0:dbReference type="InParanoid" id="P33013" />
<ns0:dbReference type="KO" id="K07258" />
<ns0:dbReference type="PhylomeDB" id="P33013" />
<ns0:dbReference type="BioCyc" id="EcoCyc:RPOA-MONOMER" />
<ns0:dbReference type="BioCyc" id="ECOL316407:JW5329-MONOMER" />
<ns0:dbReference type="BioCyc" id="MetaCyc:RPOA-MONOMER" />
<ns0:dbReference type="UniPathway" id="UPA00219" />
<ns0:dbReference type="PRO" id="PR:P33013" />
<ns0:dbReference type="Proteomes" id="UP000000318">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="Proteomes" id="UP000000625">
<ns0:property type="component" value="Chromosome" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0005886">
<ns0:property type="term" value="C:plasma membrane" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="UniProtKB-SubCell" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0008800">
<ns0:property type="term" value="F:beta-lactamase activity" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0004180">
<ns0:property type="term" value="F:carboxypeptidase activity" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0004175">
<ns0:property type="term" value="F:endopeptidase activity" />
<ns0:property type="evidence" value="ECO:0000318" />
<ns0:property type="project" value="GO_Central" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0008658">
<ns0:property type="term" value="F:penicillin binding" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0009002">
<ns0:property type="term" value="F:serine-type D-Ala-D-Ala carboxypeptidase activity" />
<ns0:property type="evidence" value="ECO:0000255" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0071555">
<ns0:property type="term" value="P:cell wall organization" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="UniProtKB-KW" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0009252">
<ns0:property type="term" value="P:peptidoglycan biosynthetic process" />
<ns0:property type="evidence" value="ECO:0000501" />
<ns0:property type="project" value="UniProtKB-UniPathway" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0000270">
<ns0:property type="term" value="P:peptidoglycan metabolic process" />
<ns0:property type="evidence" value="ECO:0000314" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="GO" id="GO:0008360">
<ns0:property type="term" value="P:regulation of cell shape" />
<ns0:property type="evidence" value="ECO:0000316" />
<ns0:property type="project" value="EcoCyc" />
</ns0:dbReference>
<ns0:dbReference type="Gene3D" id="2.60.410.10">
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="Gene3D" id="3.40.710.10">
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR012338">
<ns0:property type="entry name" value="Beta-lactam/transpept-like" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR015956">
<ns0:property type="entry name" value="Peniciliin-bd_prot_C_sf" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR018044">
<ns0:property type="entry name" value="Peptidase_S11" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR012907">
<ns0:property type="entry name" value="Peptidase_S11_C" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR037167">
<ns0:property type="entry name" value="Peptidase_S11_C_sf" />
</ns0:dbReference>
<ns0:dbReference type="InterPro" id="IPR001967">
<ns0:property type="entry name" value="Peptidase_S11_N" />
</ns0:dbReference>
<ns0:dbReference type="Pfam" id="PF07943">
<ns0:property type="entry name" value="PBP5_C" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="Pfam" id="PF00768">
<ns0:property type="entry name" value="Peptidase_S11" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="PRINTS" id="PR00725">
<ns0:property type="entry name" value="DADACBPTASE1" />
</ns0:dbReference>
<ns0:dbReference type="SMART" id="SM00936">
<ns0:property type="entry name" value="PBP5_C" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="SUPFAM" id="SSF56601">
<ns0:property type="entry name" value="SSF56601" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:dbReference type="SUPFAM" id="SSF69189">
<ns0:property type="entry name" value="SSF69189" />
<ns0:property type="match status" value="1" />
</ns0:dbReference>
<ns0:proteinExistence type="evidence at protein level" />
<ns0:keyword id="KW-0002">3D-structure</ns0:keyword>
<ns0:keyword id="KW-0121">Carboxypeptidase</ns0:keyword>
<ns0:keyword id="KW-0997">Cell inner membrane</ns0:keyword>
<ns0:keyword id="KW-1003">Cell membrane</ns0:keyword>
<ns0:keyword id="KW-0133">Cell shape</ns0:keyword>
<ns0:keyword id="KW-0961">Cell wall biogenesis/degradation</ns0:keyword>
<ns0:keyword id="KW-0378">Hydrolase</ns0:keyword>
<ns0:keyword id="KW-0472">Membrane</ns0:keyword>
<ns0:keyword id="KW-0573">Peptidoglycan synthesis</ns0:keyword>
<ns0:keyword id="KW-0645">Protease</ns0:keyword>
<ns0:keyword id="KW-1185">Reference proteome</ns0:keyword>
<ns0:keyword id="KW-0732">Signal</ns0:keyword>
<ns0:feature type="signal peptide" evidence="3">
<ns0:location>
<ns0:begin position="1" />
<ns0:end position="21" />
</ns0:location>
</ns0:feature>
<ns0:feature type="chain" description="D-alanyl-D-alanine carboxypeptidase DacD" id="PRO_0000027234">
<ns0:location>
<ns0:begin position="22" />
<ns0:end position="388" />
</ns0:location>
</ns0:feature>
<ns0:feature type="active site" description="Acyl-ester intermediate" evidence="2">
<ns0:location>
<ns0:position position="63" />
</ns0:location>
</ns0:feature>
<ns0:feature type="active site" description="Proton acceptor" evidence="2">
<ns0:location>
<ns0:position position="66" />
</ns0:location>
</ns0:feature>
<ns0:feature type="active site" evidence="2">
<ns0:location>
<ns0:position position="129" />
</ns0:location>
</ns0:feature>
<ns0:feature type="binding site" description="Substrate" evidence="2">
<ns0:location>
<ns0:position position="232" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="36" />
<ns0:end position="43" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="1">
<ns0:location>
<ns0:begin position="44" />
<ns0:end position="46" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="49" />
<ns0:end position="54" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="62" />
<ns0:end position="64" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="65" />
<ns0:end position="78" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="87" />
<ns0:end position="89" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="92" />
<ns0:end position="94" />
</ns0:location>
</ns0:feature>
<ns0:feature type="turn" evidence="1">
<ns0:location>
<ns0:begin position="100" />
<ns0:end position="104" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="115" />
<ns0:end position="117" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="118" />
<ns0:end position="126" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="131" />
<ns0:end position="142" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="145" />
<ns0:end position="158" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="168" />
<ns0:end position="171" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="181" />
<ns0:end position="192" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="196" />
<ns0:end position="200" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="201" />
<ns0:end position="203" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="206" />
<ns0:end position="209" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="212" />
<ns0:end position="215" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="219" />
<ns0:end position="222" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="226" />
<ns0:end position="228" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="232" />
<ns0:end position="235" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="241" />
<ns0:end position="249" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="252" />
<ns0:end position="263" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="264" />
<ns0:end position="281" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="282" />
<ns0:end position="291" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="315" />
<ns0:end position="320" />
</ns0:location>
</ns0:feature>
<ns0:feature type="helix" evidence="1">
<ns0:location>
<ns0:begin position="321" />
<ns0:end position="326" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="327" />
<ns0:end position="332" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="349" />
<ns0:end position="355" />
</ns0:location>
</ns0:feature>
<ns0:feature type="strand" evidence="1">
<ns0:location>
<ns0:begin position="358" />
<ns0:end position="365" />
</ns0:location>
</ns0:feature>
<ns0:evidence key="1" type="ECO:0000244">
<ns0:source>
<ns0:dbReference type="PDB" id="5FSR" />
</ns0:source>
</ns0:evidence>
<ns0:evidence key="2" type="ECO:0000250" />
<ns0:evidence key="3" type="ECO:0000255" />
<ns0:evidence key="4" type="ECO:0000305" />
<ns0:sequence length="388" mass="43346" checksum="453FE18C21D207D0" modified="1998-07-15" version="3" precursor="true">MKRRLIIAASLFVFNLSSGFAAENIPFSPQPPEIHAGSWVLMDYTTGQILTAGNEHQQRNPASLTKLMTGYVVDRAIDSHRITPDDIVTVGRDAWAKDNPVFVGSSLMFLKEGDRVSVRDLSRGLIVDSGNDACVALADYIAGGQRQFVEMMNNYAEKLHLKDTHFETVHGLDAPGQHSSAYDLAVLSRAIIHGEPEFYHMYSEKSLTWNGITQQNRNGLLWDKTMNVDGLKTGHTSGAGFNLIASAVDGQRRLIAVVMGADSAKGREEEARKLLRWGQQNFTTVQILHRGKKVGTERIWYGDKENIDLGTEQEFWMVLPKAEIPHIKAKYTLDGKELTAPISAHQRVGEIELYDRDKQVAHWPLVTLESVGEGSMFSRLSDYFHHKA</ns0:sequence>
</ns0:entry>
```

### Statistiques de termes GO

Les nombres d'occurences de tous les termes GO trouvés dans le protéome de E.coli sont stockés dans le fichier `data/EColiK12_GOcounts.json`. Ces statistiques ont été dérivées du fichier `data/uniprot-proteome_UP000000625.xml`.

## Objectifs

1. Manipuler les données experimentales tabulées
2. Representer graphiquement les données d'abondance
3. Construire la pvalue des fonctions biologiques (termes GO) portées par les protéines surabondantes
4. Visualiser interactivement les pathways plus enrichis.

### Description statistique des Fold Change
La lecture des données au format tabulé est l'occasion de se familliariser avec la [librairie pandas](https://pandas.pydata.org).

##### Lecture de données

###### source:`data/TCL_wt1.tsv`

La fonction `read_csv` accepte différents [arguments](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html) de format de données très utiles.

```python
df = pandas.read_csv()
```

Quel est le type de l'objet `df`?
```
dataframe (pandas.DataFrame)
```

##### Descriptions d'une table de données
Que permettent les méthodes suivantes?
###### df.shape
```
Returns a tuple representing the dimensionality of the DataFrame.

(2024, 7)
```
###### df.head()
```
Returns the first n rows.

Accession 	Description 	Gene Symbol 	Corrected Abundance ratio (1.53) 	Log2 Corrected Abundance Ratio 	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT) 	-LOG10 Adj.P-val
0 	P75936 	Basal-body rod modification protein FlgD OS=Es... 	flgD 	0.075816993 	-3.721334942 	0.000055 	4.260067469
1 	P76231 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC 	0.092810458 	-3.429568818 	0.000351 	3.45462743
2 	P0A8S9 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD 	0.102614379 	-3.284695189 	0.000027 	4.571899347
3 	P0CE48 	Elongation factor Tu 2 OS=Escherichia coli (st... 	tufB 	#VALEUR! 	#VALEUR! 	NaN 	#VALEUR!
4 	P05706 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB 	0.108496732 	-3.204276506 	0.019963 	1.699767669
```
###### df.tail()
```
Returns the last n rows.

Accession 	Description 	Gene Symbol 	Corrected Abundance ratio (1.53) 	Log2 Corrected Abundance Ratio 	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT) 	-LOG10 Adj.P-val
2019 	P24240 	6-phospho-beta-glucosidase AscB OS=Escherichia... 	ascB 	#VALEUR! 	#VALEUR! 	NaN 	#VALEUR!
2020 	P0A917 	Outer membrane protein X OS=Escherichia coli (... 	ompX 	1.579738562 	0.65968582 	0.002226 	2.652390664
2021 	P02931 	Outer membrane protein F OS=Escherichia coli (... 	ompF 	1.754901961 	0.811390435 	0.000068 	4.16495627
2022 	P0AB40 	Multiple stress resistance protein BhsA OS=Esc... 	bhsA 	1.798039216 	0.846424487 	0.035928 	1.444561032
2023 	P76042 	Putative ABC transporter periplasmic-binding p... 	ycjN 	#VALEUR! 	#VALEUR! 	NaN 	#VALEUR!
```
###### df.columns
```
Returns the column labels of the DataFrame.

Index(['Accession', 'Description', 'Gene Symbol',
       'Corrected Abundance ratio (1.53)', 'Log2 Corrected Abundance Ratio',
       'Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)',
       '-LOG10 Adj.P-val'],
      dtype='object')
```
###### df.dtypes
```
Returns a Series with the data type of each column.

Accession                                                        object
Description                                                      object
Gene Symbol                                                      object
Corrected Abundance ratio (1.53)                                 object
Log2 Corrected Abundance Ratio                                   object
Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)    float64
-LOG10 Adj.P-val                                                 object
dtype: object
```
###### df.info
```
Prints a concise summary of a DataFrame.

<bound method DataFrame.info of      Accession                                        Description Gene Symbol  \
0       P75936  Basal-body rod modification protein FlgD OS=Es...        flgD   
1       P76231  Uncharacterized protein YeaC OS=Escherichia co...        yeaC   
2       P0A8S9  Flagellar transcriptional regulator FlhD OS=Es...        flhD   
3       P0CE48  Elongation factor Tu 2 OS=Escherichia coli (st...        tufB   
4       P05706  PTS system glucitol/sorbitol-specific EIIA com...        srlB   
...        ...                                                ...         ...   
2019    P24240  6-phospho-beta-glucosidase AscB OS=Escherichia...        ascB   
2020    P0A917  Outer membrane protein X OS=Escherichia coli (...        ompX   
2021    P02931  Outer membrane protein F OS=Escherichia coli (...        ompF   
2022    P0AB40  Multiple stress resistance protein BhsA OS=Esc...        bhsA   
2023    P76042  Putative ABC transporter periplasmic-binding p...        ycjN   

     Corrected Abundance ratio (1.53) Log2 Corrected Abundance Ratio  \
0                         0.075816993                   -3.721334942   
1                         0.092810458                   -3.429568818   
2                         0.102614379                   -3.284695189   
3                            #VALEUR!                       #VALEUR!   
4                         0.108496732                   -3.204276506   
...                               ...                            ...   
2019                         #VALEUR!                       #VALEUR!   
2020                      1.579738562                     0.65968582   
2021                      1.754901961                    0.811390435   
2022                      1.798039216                    0.846424487   
2023                         #VALEUR!                       #VALEUR!   

      Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)  \
0                                              0.000055              
1                                              0.000351              
2                                              0.000027              
3                                                   NaN              
4                                              0.019963              
...                                                 ...              
2019                                                NaN              
2020                                           0.002226              
2021                                           0.000068              
2022                                           0.035928              
2023                                                NaN              

     -LOG10 Adj.P-val  
0         4.260067469  
1          3.45462743  
2         4.571899347  
3            #VALEUR!  
4         1.699767669  
...               ...  
2019         #VALEUR!  
2020      2.652390664  
2021       4.16495627  
2022      1.444561032  
2023         #VALEUR!  

[2024 rows x 7 columns]>
```
###### df.describe()
```
Generates descriptive statistics.

Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)
count 	1.750000e+03
mean 	8.238311e-01
std 	3.506349e-01
min 	1.034030e-08
25% 	1.000000e+00
50% 	1.000000e+00
75% 	1.000000e+00
max 	1.000000e+00
```
###### df.dropna()
```
Remove missing values.

Accession 	Description 	Gene Symbol 	Corrected Abundance ratio (1.53) 	Log2 Corrected Abundance Ratio 	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT) 	-LOG10 Adj.P-val
0 	P75936 	Basal-body rod modification protein FlgD OS=Es... 	flgD 	0.075816993 	-3.721334942 	0.000055 	4.260067469
1 	P76231 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC 	0.092810458 	-3.429568818 	0.000351 	3.45462743
2 	P0A8S9 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD 	0.102614379 	-3.284695189 	0.000027 	4.571899347
4 	P05706 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB 	0.108496732 	-3.204276506 	0.019963 	1.699767669
5 	P29744 	Flagellar hook-associated protein 3 OS=Escheri... 	flgL 	0.124183007 	-3.009460329 	0.036746 	1.434786589
... 	... 	... 	... 	... 	... 	... 	...
2011 	P77330 	Prophage lipoprotein Bor homolog OS=Escherichi... 	borD 	1.535947712 	0.619129104 	0.310725 	0.507623276
2016 	P02930 	Outer membrane protein TolC OS=Escherichia col... 	tolC 	1.552287582 	0.634395861 	0.013373 	1.873756665
2020 	P0A917 	Outer membrane protein X OS=Escherichia coli (... 	ompX 	1.579738562 	0.65968582 	0.002226 	2.652390664
2021 	P02931 	Outer membrane protein F OS=Escherichia coli (... 	ompF 	1.754901961 	0.811390435 	0.000068 	4.16495627
2022 	P0AB40 	Multiple stress resistance protein BhsA OS=Esc... 	bhsA 	1.798039216 	0.846424487 	0.035928 	1.444561032

1746 rows × 7 columns
```

##### Accès aux éléments d'une table de données

```python
values = df[['Description', 'Gene Symbol']]

Description 	Gene Symbol
0 	Basal-body rod modification protein FlgD OS=Es... 	flgD
1 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC
2 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD
3 	Elongation factor Tu 2 OS=Escherichia coli (st... 	tufB
4 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB
... 	... 	...
2019 	6-phospho-beta-glucosidase AscB OS=Escherichia... 	ascB
2020 	Outer membrane protein X OS=Escherichia coli (... 	ompX
2021 	Outer membrane protein F OS=Escherichia coli (... 	ompF
2022 	Multiple stress resistance protein BhsA OS=Esc... 	bhsA
2023 	Putative ABC transporter periplasmic-binding p... 	ycjN

2024 rows × 2 columns
```

Quel est le type de `values` ?
```
```

Verifiez si certaines méthodes de `DataFrame` lui sont applicables.
Ce type supporte l'accès par indice et les slice `[a:b]`
```
values.shape
(2024, 2)

values.head()
Description 	Gene Symbol
0 	Basal-body rod modification protein FlgD OS=Es... 	flgD
1 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC
2 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD
3 	Elongation factor Tu 2 OS=Escherichia coli (st... 	tufB
4 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB

values.tail()
Description 	Gene Symbol
2019 	6-phospho-beta-glucosidase AscB OS=Escherichia... 	ascB
2020 	Outer membrane protein X OS=Escherichia coli (... 	ompX
2021 	Outer membrane protein F OS=Escherichia coli (... 	ompF
2022 	Multiple stress resistance protein BhsA OS=Esc... 	bhsA
2023 	Putative ABC transporter periplasmic-binding p... 	ycjN


values.columns
Index(['Description', 'Gene Symbol'], dtype='object')

values.dtypes
Description    object
Gene Symbol    object
dtype: object

values.info
<bound method DataFrame.info of                                             Description Gene Symbol
0     Basal-body rod modification protein FlgD OS=Es...        flgD
1     Uncharacterized protein YeaC OS=Escherichia co...        yeaC
2     Flagellar transcriptional regulator FlhD OS=Es...        flhD
3     Elongation factor Tu 2 OS=Escherichia coli (st...        tufB
4     PTS system glucitol/sorbitol-specific EIIA com...        srlB
...                                                 ...         ...
2019  6-phospho-beta-glucosidase AscB OS=Escherichia...        ascB
2020  Outer membrane protein X OS=Escherichia coli (...        ompX
2021  Outer membrane protein F OS=Escherichia coli (...        ompF
2022  Multiple stress resistance protein BhsA OS=Esc...        bhsA
2023  Putative ABC transporter periplasmic-binding p...        ycjN
[2024 rows x 2 columns]>

values.describe()
Description 	Gene Symbol
count 	2024 	2017
unique 	2024 	2015
top 	DNA translocase FtsK OS=Escherichia coli (stra... 	rlmC; rlmD
freq 	1 	2

values.dropna()
Description 	Gene Symbol
0 	Basal-body rod modification protein FlgD OS=Es... 	flgD
1 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC
2 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD
3 	Elongation factor Tu 2 OS=Escherichia coli (st... 	tufB
4 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB
... 	... 	...
2019 	6-phospho-beta-glucosidase AscB OS=Escherichia... 	ascB
2020 	Outer membrane protein X OS=Escherichia coli (... 	ompX
2021 	Outer membrane protein F OS=Escherichia coli (... 	ompF
2022 	Multiple stress resistance protein BhsA OS=Esc... 	bhsA
2023 	Putative ABC transporter periplasmic-binding p... 	ycjN

2017 rows × 2 columns
```

##### Accès indicé

On peut accéder aux valeurs du DataFrame via des indices ou plages d'indice. La structure se comporte alors comme une matrice. La cellule en haut et à gauche est de coordonnées (0,0).
Il y a différentes manières de le faire, l'utilisation de `.iloc[slice_ligne,slice_colonne]` constitue une des solutions les plus simples. N'oublions pas que shape permet d'obtenir les dimensions (lignes et colonnes) du DataFrame.
###### Acceder aux cinq premières lignes de toutes les colonnes
```python
df.iloc[:5]

 	Accession 	Description 	Gene Symbol 	Corrected Abundance ratio (1.53) 	Log2 Corrected Abundance Ratio 	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT) 	-LOG10 Adj.P-val
0 	P75936 	Basal-body rod modification protein FlgD OS=Es... 	flgD 	0.075816993 	-3.721334942 	0.000055 	4.260067469
1 	P76231 	Uncharacterized protein YeaC OS=Escherichia co... 	yeaC 	0.092810458 	-3.429568818 	0.000351 	3.45462743
2 	P0A8S9 	Flagellar transcriptional regulator FlhD OS=Es... 	flhD 	0.102614379 	-3.284695189 	0.000027 	4.571899347
3 	P0CE48 	Elongation factor Tu 2 OS=Escherichia coli (st... 	tufB 	#VALEUR! 	#VALEUR! 	NaN 	#VALEUR!
4 	P05706 	PTS system glucitol/sorbitol-specific EIIA com... 	srlB 	0.108496732 	-3.204276506 	0.019963 	1.699767669
```

###### Acceder à toutes les lignes de la dernière colonne
```python
df.iloc[:, lambda df: [6]]

-LOG10 Adj.P-val
0 	4.260067469
1 	3.45462743
2 	4.571899347
3 	#VALEUR!
4 	1.699767669
... 	...
2019 	#VALEUR!
2020 	2.652390664
2021 	4.16495627
2022 	1.444561032
2023 	#VALEUR!

2024 rows × 1 columns
```

###### Acceder aux cinq premières lignes des colonnes 0, 2 et 3
```python
df.iloc[:5, [0,2,3]]
Accession 	Gene Symbol 	Corrected Abundance ratio (1.53)
0 	P75936 	flgD 	0.075816993
1 	P76231 	yeaC 	0.092810458
2 	P0A8S9 	flhD 	0.102614379
3 	P0CE48 	tufB 	#VALEUR!
4 	P05706 	srlB 	0.108496732
```

##### Conversion de type

Le type des valeurs d'une colonne peut être spécifiée:

* à la lecture

```python
pandas.read_csv('data/TCL_wt1.tsv', sep="\t", dtype = {'Accession': str, 'Description': str, 'Gene Symbol': str, 
                                                 'Corrected Abundance ratio (1.53)': np.float,  'Log2 Corrected Abundance Ratio': np.float, 
                                                 'Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)': np.float, '-LOG10 Adj.P-val': np.float})
```

On peut specifier la chaine de caractères correspondant au valeurs "incorrectes" produites par excel comme-ci
'''python
pandas.read_csv('data/TCL_wt1.tsv', sep="\t", na_values="#VALEUR!")
'''

* modifiée à la volée

```python
df = df.astype({'Log2 Corrected Abundance Ratio': float, '-LOG10 Adj.P-val': float } )
```

##### Selection avec contraintes
La méthode `loc` permet de selectionner toutes les lignes/colonnes respectant certaines contraintes

* Contraintes de valeurs continues

```python
df.loc[(df['-LOG10 Adj.P-val'] > 0 )  & (df['Log2 Corrected Abundance Ratio'] > 0.0 ) ]
```

* Contraintes de valeurs discrètes

```python
df.loc[ df['Gene Symbol'].isin(['fadR', 'arcA'] ) ]
```


#### Appliquons ces outils à l'analyse de données protéomique

##### 1. Charger le contenu du fichier `data/TCL_wt1.tsv` dans un notebook en eliminant les lignes porteuses de valeurs numériques aberrantes
```python
import pandas
pandas.read_csv('data/TCL_wt1.tsv', sep="\t", na_values="#VALEUR!")
```

##### 2. Representez par un histogramme les valeurs de `Log2 Corrected Abundance Ratio`
```python
log2_car = DF["Log2 Corrected Abundance Ratio"]
mean = np.mean(log2_car)
std = np.std(log2_car)

fig, ax = plt.subplots()
hist = ax.hist(log2_car, bins=100)
x = np.linspace(min(log2_car), max(log2_car), 100) # generate PDF domain points
dx = hist[1][1] - hist[1][0] # Get single value bar height
scale = len(log2_car)*dx # scale accordingly
ax.plot(x, norm.pdf(x, mean, std**2)*scale) # compute theoritical PDF and draw it
fig.show()
```

##### 3. A partir de cette échantillon de ratio d'abondance,  estimez la moyenne <img src="https://render.githubusercontent.com/render/math?math=\mu"> et l'ecart-type <img src="https://render.githubusercontent.com/render/math?math=\sigma"> d'une loi normale.
```


```

##### 4. Superposez la densité de probabilité de cette loi sur l'histogramme. Attention, la densité de probabilité devra être mis à l'echelle de l'histogramme (cf ci-dessous)


```python
fig, ax = plt.subplots()
hist = ax.hist(_, bins=100) # draw histogram
x = np.linspace(min(_), max(_), 100) # generate PDF domain points
dx = hist[1][1] - hist[1][0] # Get single value bar height
scale = len(_)*dx # scale accordingly
ax.plot(x, norm.pdf(x, mu, sqrt(S_2))*scale) # compute theoritical PDF and draw it
```

![Histogramme à inserez ici](histogram_log2FC.png "Title")

##### 5. Quelles remarques peut-on faire à l'observation de l'histogramme et de loi théorique?

```


```

#### Construction d'un volcano plot

##### A l'aide de la méthode [scatter](https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.axes.Axes.scatter.html) representer <img src="https://render.githubusercontent.com/render/math?math=-\text{Log}_{10}({\text{p-value}}) = f(\text{Log}_2(\text{abundance ratio}))">

##### Matérialisez le quadrant des protéines surabondantes, par deux droites ou un rectangle
Sont condidérées comme surabondantes les proteines remplissant ces deux critères:

* <img src="https://render.githubusercontent.com/render/math?math=\text{Log}_2(\text{abundance ratio})\gt\mu%2B\sigma">  
* <img src="https://render.githubusercontent.com/render/math?math=\text{p-value}>0.001">

![Volcano plot + quadrant à inserez ici](histogram_log2FC.png "Title")

### Analyse Fonctionelle de pathway

Nous allons implementer une approche ORA (Over Representation Analysis) naive.

##### 1. Retrouver les entrées du fichier TSV des protéines surabondates

Quelles sont leurs identifiants UNIPROT ?
``` 



```

#### 2. Lister les termes GO portés par ces protéines surabondates

Les `entry` du fichier `data/uniprot-proteome_UP000000625.xml` présentent des balises de ce type:

```xml
<dbReference type="GO" id="GO:0005737">
<property type="term" value="C:cytoplasm"/>
<property type="evidence" value="ECO:0000501"/>
<property type="project" value="UniProtKB-SubCell"/>
</dbReference>
```

A ce stade, on se contentera des identifiants GO (eg `GO:0005737`). Vous pouvez faire un [set](https://docs.python.org/3.8/library/stdtypes.html#set) de cette liste d'identifiants GO pour en éliminer rapidement la redondance.

#### 3. Obtention des paramètres du modèle

Nous evaluerons la significativité de la présence de tous les termes GO portés par les protéines surabondantes à l'aide d'un modèle hypergéometrique.

Si k protéines surabondantes porte un terme GO, la pvalue de ce terme sera équivalente à <img src="https://render.githubusercontent.com/render/math?math=P(X\ge k), X \sim H(k,K,n,N)">.
Completer le tableau ci-dessous avec les quantités vous  semblant adéquates

| Symboles | Paramètres | Quantités Biologiques |
| --- | --- | --- |
| k | nombre de succès observés| |
| K | nombre de succès possibles| |
| n | nombre d'observations| |
| N | nombre d'elements observables| |

#### 4. Calcul de l'enrichissement en fonction biologiques

A l'aide du contenu de `data/EColiK12_GOcounts.json` parametrez la loi hypergeometrique et calculez la pvalue
de chaque terme GO portés par les protéines surabondantes. Vous reporterez ces données dans le tableau ci-dessous

| identifiant GO | définition | occurence | pvalue|
|---|---|---|---|
|   |   |   |   |

Quelle interpretation biologique faites-vous de cet enrichissement en termes GO spécifiques ?

```




```


