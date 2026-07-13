# Composite Module Analyst (CMA)

The **Composite Module Analyst (CMA)** is a computational tool designed to identify complex regulatory signatures—specifically combinations of **transcription factor (TF)** binding sites—within genetic sequences. It utilizes a fitness-based approach combined with a genetic algorithm to define promoter models that best fit observed gene expression profiles.

**Background** The development of the CMA algorithm is rooted in the biological understanding that functionally related genes—those involved in the same biochemical, physiological, or molecular-genetic processes—are often coordinately regulated. This regulation is driven by the precise binding of multiple transcription factors to target sites (cis-elements) within the regulatory regions of the genes. It is the specific combinations of these TFs, rather than a single factor alone, that dictate the specificity of gene transcription and generate unique patterns of gene expression.

Prior to the introduction of CMA, researchers heavily utilized microarray measurements to reveal hundreds of genes whose expression changed in association with diseases or specific conditions. However, interpreting the actual molecular mechanisms driving these changes remained a significant challenge. While various predictive methods existed to find regulatory motifs, traditional "ab initio" motif-finding methods struggled to distinguish true regulatory signals from background noise, particularly within the long regulatory regions (10 Kb or more) typical of humans and other higher eukaryotes.

**Purpose** The primary purpose of the CMA algorithm is to overcome these limitations by **identifying "composite modules," which are stable combinations of TF binding sites common to the promoters of functionally related genes**. These composite modules are responsible for driving the major component of the gene expression patterns observed in these genes.

The **Genetic Algorithm (GA)** serves as the core optimization engine in the Composite Module Analyst software. Its primary role is to sift through massive amounts of biological data to identify the optimal composite promoter model—specific combinations of transcription factor (TF) binding sites—that best explains observed gene expression profiles.

**How the Genetic Algorithm Works**

The GA mimics the process of natural evolution to iteratively improve candidate promoter models until it finds the optimal regulatory signature.

    1. **Initialization and Encoding:** The algorithm first takes the candidate binding sites identified in the sequences and generates a random set of starting models based on predefined physical limits (like the minimum/maximum number of matrices and module width). Each candidate model acts as an **"organism"** and is mathematically encoded into "chromosomes." One chromosome encodes individual matrices and their parameters, while another encodes pairs of matrices and their spatial constraints.

    2. **Fitness Evaluation:** During each iteration, every organism in the population is evaluated using a **multicomponent fitness function**. This function scores how well the model discriminates the experiment track from the control track. The score is calculated based on a combination of statistical measures, including linear regression fit, Student's t-test values, specificity/sensitivity (error rates), the normality of the score distribution, and a penalty to prevent the model from becoming unnecessarily complex.

    3. **Selection and Elitism:** The organisms that score the highest are selected to survive and multiply into the next generation. To ensure that highly successful discoveries are not lost during the evolutionary process, an **elite size** parameter dictates a specific number of the absolute best organisms that will unconditionally survive.

    4. **Mutation and Recombination:** To continuously generate and test new variations, the algorithm applies genetic operators to the surviving population. **Mutations** occur randomly and can involve swapping a matrix for a different one from the library, or making small adjustments to cut-off values and impact parameters. **Recombination** involves exchanging parts of chromosomes between two different candidate models. A **mutation rate** controls how frequently and significantly these changes occur.

    5. **Termination:** The population size and evolution run for a **predefined number of iterations**. To conserve computational resources, a **non-change limit** can automatically halt the process early if the best score in the population fails to improve after a set number of rounds.

Once the evolution concludes, the algorithm outputs the "best" model—the one characterized by the highest fitness value, featuring fully optimized parameters and matrices that best characterize the regulatory logic of the input sequences.

## Initial Composite Module Analyst

Complete name: analyses/Methods/Site analysis/Construct composite modules on tracks

**Parameters**

- **Input and Output Tracks**
    These parameters define the datasets the algorithm will analyze and where the results will be stored.
    - Experiment track: The path to the experiment ('yes') track.
    - Control track: The path to the control ('no') track.
    - Output path: The destination path to store the final result.

- **Genetic Algorithm Parameters**
    These parameters control the evolutionary search process used to find the optimal composite module.
    - Number of iterations: The total number of evolutionary cycles the algorithm will run.
    - Population size: The number of candidate models (organisms) generated in each generation.
    - Non-change limit: The number of iterations after which the algorithm will automatically stop if the best score is not improved.
    - Elite size: The number of elite organisms (the absolute best-scoring models) that will unconditionally survive into the next generation.
    - Mutation rate: This parameter controls both how frequently mutations occur and how significant those changes are, scaled from 0 to 1.

- **Score Calculation Parameters**
    These settings influence how the fitness score of each candidate composite module is calculated.
    - Penalty rate: The specific penalty value assigned to penalize sub-optimal model traits during scoring.
    - Site models in focus: Defines specific models that must be present in the resulting composite model to achieve a non-zero score.
    - Site models in focus table: The table containing the designated site models in focus.
    - Sequence column: The specific column used to pull sequences from the data.

- **Model Parameters**
    These constrain the macroscopic architecture of the composite models.
    - Min modules: The minimum allowed number of distinct modules in the final output.
    - Max modules: The maximum allowed number of distinct modules in the final output.

- **Gaussian Model Parameters**

    These parameters specifically constrain the internal structure and spatial distribution of the binding sites within each individual module.
    - Min models: The minimal number of distinct site models allowed within a single module.
    - Max models: The maximal number of distinct site models allowed within a single module.
    - Min sites to account: The minimal number of actual site occurrences to account for.
    - Max sites to account: The maximal number of actual site occurrences to account for.
    - Min module width: The minimal allowable width (measured as the standard deviation or sigma) of the spatial distribution of the module.
    - Max module width: The maximal allowable width (sigma) of the spatial distribution of the module.
    - Pairs mode: A toggle that, when activated, dictates whether sites from at least two different site models (which both satisfy their respective cut-off conditions) must be present together on the sequence.


## Composite Module Analyst with correlation

Complete name: analyses/Methods/Site analysis/Construct composite modules on track (correlation)

This modification operates on a single dataset of experimental data, omitting the need for a control track. This aligns with the "alternative input" approach described in the CMA literature, where the algorithm evaluates a single set of promoters that already have assigned expression values.

In this correlation-based method, the algorithm utilizes linear regression as its objective function. The genetic algorithm calculates how well the fuzzy promoter scores—which represent how strongly a promoter sequence matches the generated composite model—fit the actual continuous expression values assigned to the corresponding genes. It performs this linear regression by fitting the promoter scores to a curve and computing the regression value (R<sup>2</sup>). This regression component evaluates how accurately the predicted composite module explains the observed differential expression in your experiment.

**Parameters**

- This method shares the same parameters as initial CMA. Except next two parameters:
    - Experiment track – Path to experiment ('yes') track
    - Correlation column – Column to correlate with

## Composite Module Analyst weighted

Complete name: analyses/Methods/Site analysis/Construct composite modules on track (weighted correlation)

Like the correlation method, this variant relies exclusively on experimental data and works on a single set of promoters that already carry per-gene values, so no control track is required. It extends the correlation approach in two ways: it **weights each promoter by the reliability of its measurement**, and it **rewards composite modules assembled from transcription factors that are known to interact**. Together these bias the genetic algorithm toward statistically well-supported and biologically plausible modules.

**Weighted rank correlation.** As in the correlation method, the fitness measures how well the fuzzy promoter scores—how strongly each promoter matches the generated composite model—agree with the continuous values assigned to the corresponding genes. The weighted variant differs in three respects:

- Both the promoter scores and the values of the *Correlation column* are converted to **ranks** before fitting, giving a Spearman-like association that is robust to outliers.
- The regression is a **weighted linear regression**, and its regression value (R<sup>2</sup>) is used as the correlation component of the fitness.
- The weights are taken from the *Weight column*: each promoter's weight is computed as −log<sub>10</sub>(value). A low p-value/FDR (high statistical significance) therefore yields a large weight and pulls the fit more strongly, while non-significant promoters are down-weighted and contribute little.

**Interaction prior.** In addition to the fit, the fitness is multiplied by a factor that grows with the number of **distinct known-interacting transcription-factor pairs present in the model**. The interacting pairs are read from the *Interactions file*, and each unordered pair is counted at most once—repeating the same pair across several modules gives no extra credit. This steers the search toward composite modules built from cooperating TFs rather than arbitrary combinations of matrices.

**How the score is computed.** For each candidate model the three components are combined multiplicatively:

- a **complexity penalty** that discourages unnecessarily large models (as in the initial CMA);
- the **weighted rank-correlation** term, −log<sub>10</sub>(1 − |R<sup>2</sup>|), which rewards a tight fit between the module scores and the gene values;
- an **interaction factor** that increases with the number of known-interacting TF pairs contained in the model.

The genetic algorithm maximises this combined fitness, so the best model is one that is compact, whose score co-varies strongly (in a p-value-weighted sense) with the experimental values, and that is composed of transcription factors known to act together.

**Parameters**

- This method shares the same parameters as the initial CMA, with the following differences:
    - Experiment track – Path to the experiment ('yes') track.
    - Interactions file – Table of known transcription-factor interactions (pairs); used to reward modules built from cooperating TFs.
    - Correlation column – Column holding the per-gene value that the module score is correlated with (e.g. logFC).
    - Weight column – Column whose values (e.g. p-value or FDR) are converted into regression weights via −log<sub>10</sub>.
     