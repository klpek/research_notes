## Rosetta Comparative Model
### Objective
Algorithms that attempt to predict a structure from sequence primarily use two sources of
information.
- The first source is physical in nature: proteins fold into their lowest energy state. The
structure prediction problem becomes a search for the lowest energy structure.
- proteins of similar sequences have similar structure, and therefore proteins of known structure can guide modeling.

#### Rosetta Comparative Model Process(2011 edition)
we develop a probabilistic approach to derive spatial restraints from proteins of known structure using advances in alignment technology and the growth in the number of structures in the Protein Data Bank.
##### Method
###### Distance restraints from a single template
For deriving distance restraints, pairs of amino acids aligned by HHSearch5 were examined.
We computed statistics over all pairs of aligned residues with Cα atoms less than 10 Å apart in the template structure that was separated by more than 10 residues along the query sequence. For each of these pairs, the magnitude of the difference in distances between the C atoms at the two positions in the aligned structures was computed ($$$| {R1}_{ij} – {R2}_{i’j’} |$$$ where $$${R1}_{ij}$$$ is the distance between atoms i and j in structure 1, and $$${R2}_ {i’j’}$$$ is the distance between the equivalent atoms i ’ and j ’ in structure 2). These distance deviations were placed in a bin based on the sequence similarity and structural context of the two residues.
The bins are defined by the global alignment quality ( G —the negative log of the HHSearch e-value), the residue-pair alignment quality ( L —the BLOSUM627 score for aligned residue pair), the average distance to an alignment gap ( D —the distance in number of residues from the aligned pair to the nearest gap in the sequence alignment), and the
burial in the template structure ( B —number of Cβ atoms within 8 Å of the template residue Cβ).
Following the tabulation of the distance deviations, pseudocounts were added to each bin in order to reduce artifacts arising from small counts:
<img src="http://www.cattigers.review/usr/uploads/2018/01/3661736776.png" width = "450" height = "50" >

$$$P(Δ r | G, L, B, D)$$$ is the distribution of deviations between template and native $$$Cα–Cα$$$ distances given particular values of G , L , B , and D. $$$F (Δ r )$$$ is the observed distribution of
distance deviations across all values of G , L , B , and D . N is the total number of observations in the bin, $$$N_{obs} (Δ r | G , L , B , D )$$$ is the number of observations with distance deviation $$$Δ r $$$, $$$N(G, L, B, D)$$$ is the total number of observations with the given values of G, L, B, and D, and C is the number of pseudocounts. Zero-mean Gaussians were fitted to the smoothed distributions in each bin. The 10,000 fitted standard deviations, one for each bin, are the parameters of our model.

###### Combining predictions from multiple templates
<img src="http://www.cattigers.review/usr/uploads/2018/01/3404167006.png" width = "400" height = "50" >
The second term is the single sequence model from the previous section. The first term is the confidence in the alignment i compared to all other alignments. We estimate the first term using:
<img src="http://www.cattigers.review/usr/uploads/2018/01/2961929866.png" width = "200" height = "70" >
where $$$r_i$$$ is the distance between the equivalent atoms in template i , and $$$σ_i$$$ is the standard
deviation associated with that distance. The parameter k determines the extent to which predictions with lower standard deviations dominate over those with higher standard deviations, with a value of k = 0 giving all predictions equal weight.
The restraint potential for a pair of positions, given a set of aligned templates, is then a mixture of Gaussians with weights dependent on the standard deviation:
<img src="http://www.cattigers.review/usr/uploads/2018/01/4103430950.png" width = "250" height = "100" >
###### Comparative modeling with spatial restraints
In previous work,our group previously described an approach to homology modeling that
uses an input protein sequence, a template protein structure, and an alignment relating the
two. This approach has the following steps:

1. Generation of incomplete models by copying coordinates over aligned regions.
2. Completion of the models by building unaligned regions using the Rosetta fragment-based loop-modeling protocol. 
	- This step uses a centroid representation of the protein side chains and explicit backbone atoms, a low-resolution energy function, fragments from known protein structures, and kinematics that allow rebuilding of the unaligned regions without perturbing coordinates in the aligned regions.
3. Refinement of the Rosetta full-atom energy function using discrete optimization of
side-chain rotamers, small perturbations to the local backbone followed by
gradient-based minimization, and a ramping repulsive function to allow atomic
clashes to be resolved smoothly.
4. Iterative rebuilds of randomly selected sections of the chain, followed by refinement. Model selection at each stage alternates between diversification and intensification.

Here, a single iteration (steps 1–3) is carried out for computational efficiency. The restraints
are combined with the Rosetta energy function during optimization by adding to the
calculated energy:

<img src="http://www.cattigers.review/usr/uploads/2018/01/2829765096.png" width = "500" height = "50" >
The subscript pair i,j denotes a pair of residues i and j , $$$d_{i,j}$$$ is the distance between the Cα atoms of residues i and j , and $$$r_{i,j}$$$ is the restraint operating on those atoms. The probability of this distance given the restraint is estimated using the Gaussian mixture. A weight on the restraint term of 0.1 gives the restraints approximately half the contribution of the Rosetta full-atom energy.
Restraints with mean distance > 10 Å were discarded in order to speed up evaluation of the restraint score. The restraint potential was also shifted downward by subtracting the value of the potential at 10 Å in order to make the scores negative for structures that agree well with the restraints.

#### Results
The incorporation of the restraint potential into Rosetta is straightforward and outlined in Methods. A set of 20 proteins from the CASP7 experiment was selected, and alignments to templates were made to pre-CASP7 databases using HHSearch. Restraints were derived from these alignments using the multitemplate Gaussian mixture model outlined in Methods section. For each protein, 10,000 models were made using the **Rosetta rebuild and refine protocol**, both with and without restraints. The GDTMM distribution of models constructed with restraints improved in most cases, and the lowest energy models were more accurate in the restrained runs compared to the unrestrained runs.
![2.png](http://www.cattigers.review/usr/uploads/2018/01/3309077274.png)
Although the restraints improved sampling, they provide poor discrimination near the native state as the native structure generally violates a number of restraints due to structural differences within the various templates.
To investigate the contribution of the spatial restraints more thoroughly, blind predictions were made for target T0569 in the CASP9 structure prediction experiment.
Figure 4 shows the average values of the Rosetta full-atom energy and the restraints described in this work as a function of the GDTMM, which varies from 0 to 1 as model quality increases.

![ph2.png](http://www.cattigers.review/usr/uploads/2018/01/2492350919.png)
Panel A shows that the Rosetta full-atom energy is very flat until the GDTMM values are in the 0.6–0.7 range,
and hence the Rosetta full-atom energy function has difficulty distinguishing between medium (GDTMM between 0.5 and 0.3) and low-quality (GDTMM between 0.3 and 0.1)models.
However, the average energy of models with GDTMM > 0.7 drops sharply, and if samples are generated close enough to the native structure, the Rosetta energy is very effective at selecting these high-quality models.
B: The spatial restraints are qualitatively different in behavior—they are very effective at discriminating between low-quality and medium-quality models, but they are quite poor at discriminating high-quality from
medium-quality models.
C: These scores solve different model discrimination problems; and, a combination of the scores decreases almost monotonically as models become more nativelike.
Thus, the joint optimization of the Rosetta energy and the spatial restraints should be effective across the conformational landscape, even though the native structure violates some restraints, and the Rosetta energy has little ability to discriminate conformations that are far from native.

### RosettaCM
We describe an improved method for comparative modeling, RosettaCM, which optimizes a physically realistic all-atom energy function over the conformational space defined by homologous structures.
#### Method
Given a set of sequence alignments, RosettaCM assembles topologies by recombining aligned segments in Cartesian-space and building unaligned regions de novo in torsion space. The junctions between segments are regularized using a loop-closure method combining fragment superposition with gradient-based minimization.
The energies of the resulting models are optimized by all-atom refinement, and the most representative low energy model is selected.


![20180114151218.png](http://www.cattigers.review/usr/uploads/2018/01/2660108705.png)
##### Preparation
Starting from alignments of the query sequence to templates of known structure, which may be generated using remote homologue detection methods such as PsiBlast or HHsearch,or using expert knowledge.
Probabilistic distance restraints are generated from the weighted input alignments as described previously.
##### Stage 1. Full length model assembly
###### Global superposition
First, for each alignment, the sequence of interest is threaded onto the corresponding template structures to generate a set of partial threads.
One of the partial threads is randomly selected as the base model for the superposition with probability given by the user specified or default weight assigned to the alignment as described above.
For each of the remaining partial threads, the coordinates are transformed to minimize the RMSD with the base thread over the residues they have in common.
Rosetta de novo fragment files can be generated using the Rosetta program or ROBETTA server as described
elsewhere.
###### Template fragment generation
Continuous helices of at least 6 residues or strands of at least 3 residues are added to a fragment list.
###### Fragment recombination
Full chain models are generated by recombining the template-derived segments with Rosetta de novo fragments that cover the regions not represented in the templates.
The scoring function used in the Monte Carlo trajectory is a linear combination of the Rosetta low-resolution (centroid) energy function, which favors compact structures with *buried hydrophobic residues and paired beta strands, the template derived restraint functions described above, and a chain break term which penalizes large distances between residues adjacent in the sequence which can arise at fold tree boundaries *(the middle of unaligned regions).
At the beginning of the trajectory, only excluded volume interactions are considered, then secondary structure pairing and hydrophobic burial, and then the remaining terms.
The models generated in stage 1 contain all residues and generally have the correct overall topology.

##### Stage 2. Local structure optimization and loop closure
Deviations from the templates are explored and gaps in the models are closed using a combination of fragment superposition and Cartesian space minimization.
First, the aligned regions are often very close to one of the input template structures.
Second, the backbone geometry at the junctions between fold tree branches is often quite distorted.

A Monte Carlo trajectory using a two-step move is carried out.
The first step consists of random selection of a de novo or template based fragment, and substitution into the current conformation of the coordinates of the superimposed fragment.
Following the fragment insertion, Cartesian-space quasi Newton (BFGS) minimization is carried out using a differentiable version of the Rosetta centroid energy function described in the next paragraph, the template derived restraints, and explicit bond length, bond angle and improper torsion energy terms in place of the relatively weak chain break term used in stage 1 (Fig 1D).

The low energy structures resulting from the stage 2 trajectories have near ideal backbone geometry but sidechains are not explicitly represented.

##### Stage 3
The Rosetta Monte Carlo combinatorial sidechain optimization method is used to build on sidechains, and the recently developed Rosetta “FastRelax” protocol is used to iteratively refine the sidechain and backbone conformations.
Annealing is carried out by ramping up and down the strength of the repulsive interactions and at each iteration repacking the sidechains and subjecting the whole structure to quasi-Newton optimization of the sidechain and backbone coordinates first in internal coordinates and then in Cartesian coordinates.
The Rosetta full atom energy function supplemented with the alignment derived restraint function is used in all calculations with the weight on the repulsive interactions varied as described above.
#### Results
A long-standing question in the structure prediction field is the extent to which comparative models improve over the available template structures. To assess the extent to which RosettaCM improves models beyond the best available template over a large and unbiased set of structures, we participated in the CAMEO project (Continuous Automated Model Evaluation http://www.cameo3d.org/) in which recently solved structures deposited in the PDB but not yet publicly released are made available to prediction servers; all models must then be submitted prior to the public release data. Analysis of statistics collected between 05/01/2012 and 03/31/2013 by the CAMEO experiment showed that RosettaCM consistently improves over the available templates in the aligned regions (Supplementary Fig S2-A).

To compare RosettaCM to the earlier Rosetta “rebuild and refine” protocol (LoopRelax), a benchmark set was selected from CAMEO to cover different ranges of modeling difficulties (Supplementary Table 2).


To evaluate the strengths and limitations of RosettaCM, we analyze its performance in the CASP10 structure prediction experiment. As noted above, the accuracy of comparative models depends not only on the quality of the model-building approach but also on the input templates and alignments.
To evaluate the model-building approach in RosettaCM independent of template recognition and alignment generation, we focused on the subset of closer homology targets for which most methods used the same templates and alignments.
We used the structural similarity (as measured by the GDT) between the first models submitted in the CASP10 experiment by the RosettaServer and the state of the art HHpredA(a widely used public server)and ZhangServer (the top-performing server in CASP10) as a measure of the extent of convergence on templates and alignments. The 63 domains for which the average GDT between the first models was 70% or above were selected for detailed
analysis to reduce the impact of differences in template selection.
To compare the performance of the methods, we utilized statistics computed and made publicly available by the
CASP10 organizers and the Zhang lab.
According to all three metrics, on the 63 targets for which template selection and alignment generation were straightforward, the RosettaServer models were better than those of other servers both on average and in having the most top models.
On the complete set of 127 domains, RosettaServer had the most top models (Figure 2, left), although the performance of the Zhang server was considerably better according to the standard CASP sum of Z scores metric because the RosettaServer did quite poorly on several targets due to errors in template identification and domain parsing.
![3.png](http://www.cattigers.review/usr/uploads/2018/01/760751388.png)
What is the origin of the improved model-building performance evident in Figure 2? To build a good model, a com-
parative modeling method should:
- (1) improve over the closest template in the aligned regions.
- (2) properly reconstruct the loops and other regions not present in the templates.

The per-residue changes in model accuracy relative to the closest available templates for RosettaCM and several other top methods are compared in Figure 3B.
Most residues are already quite close to the correct positions in the starting templates, and hence, most frequently, the deviations are close to zero. A subset of residues is in significantly different positions in the starting template and the actual structure, and for these residues, modeling methods can make substantial improvements. For this subset, RosettaCM produced the largest number of improvements over the target set as indicated by the greater number of decreases in deviation of more than 1.5 Å (Fig 3B, left most bars). Of residues that are
improved by over 1.5 Å, 27% are on a helix, 3% are on a strand, and 70% are either on a loop structure or at the junction between a loop and a helix or strand.
![4.png](http://www.cattigers.review/usr/uploads/2018/01/1851018307.png)

Examples of the improvements are shown in Figure 4. In Figures 4A–4C, the difference in model quality relative to the best template is shown along the linear sequence for the RosettaServer model and for several other top servers. The RosettaServer models show pronounced dips below the x axis, indicating improvement relative to the best template. The structural comparisons in the lower insets illustrate structural changes taking place during modeling for the regions indicated by the red arrows. The most-often observed scenario was improvement in loop regions (Figure 4A). Concerted improvements in secondary structure placement and loop geometry were also often observed (Figures 4B and 4C).

![5.png](http://www.cattigers.review/usr/uploads/2018/01/975688944.png)

Loop region improvement is illustrated by T0667. In target T0667, there is a deletion in the residue 161–163 loop in the closest template (2WTM). There is another template with a loop of the same length (1ISP), but the conformation is quite different (2 Å over the three loop residues; Figure 4D). The RosettaCM model is much closer to the native structure (0.9 Å over the loop region; Figure 4G) compared to the other server models (Figure 4J). The improvement in loop modeling lowers the rmsd for the residues indicated by the arrow in Figure 4A. The improvement in model accuracy comes from combining fragments from the lower-ranked template, and energy minimization after the fragment is superimposed.

Concerted backbone repositioning is illustrated by T0702 and T0685. In T0702, a nonconservative glycine-to-histidine substitution at position 5 results in a side-chain-side-chain hydrogen bond with Asn 59 that does not exist in the template, which is associated with a helix shift and loop structure change relative to the closest template, 2RCY (Figure 4E). The RosettaCM model recapitulates this hydrogen bond, and the associated helix shift and loop changes (Figure 4H). The other server models do not reproduce the hydrogen bond or the backbone structural changes (Figure 4K). These changes together improve the rmsd in the region indicated by the arrow in Figure 4B. Similarly in T0685, the interhelix interaction between a Phe and Tyr in the top template used by all three servers, 2C2A, is changed to Ala and Gly in the target, which causes two helices to collapse toward each other (Figure 4F). RosettaCM is able to model this change as well as the loop connecting the helix accurately (Figure 4I). In comparison, other methods either stayed close to the template structure or modeled the helix shift but not the conformational changes in the loop region (Figure 4L).
