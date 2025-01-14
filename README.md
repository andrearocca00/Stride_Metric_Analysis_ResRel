# MFEL_ResRel project: Stride_Metric_Analysis_ResRel

## Impact of different metrics for the distance and different resolutions

**Aim of the work:** Look at how different choices of the metric used to compute the distance matrix affect the Resolution and Relevance framework.

**PLEASE NOTE:** Everything in this pipeline regards the case in which we completely neglect the Energy Distance matrix (case $\lambda$ = 1 for the general case). 

The main reference is the following article: [Information-theoretical measures identify accurate low-resolution representations of protein configurational space](https://pubs.rsc.org/en/content/articlelanding/2022/sm/d2sm00636g)

The pipeline requires as input only the structure file (first frame) and the trajectory and it computes the spatial distance matrices for every metric specified in the parameter file (with the script distances.py). Then the analysis is performed by the script analysis.py, and the results can be visualized running the notebook visual.ipynb

Every input is managed by the parameter file `par_dist.dat`, that EVERY script reads. Here the user has to specify:
- the path of the first frame and of the trajectory (the file is designed to insert them in the directory `input` cointained in `save_and_load`)
- the metrics which will be employed to build the distance matrices (in the file reported here all the implemented metrics are used)
- the different striding values for the trajectory which has to be integers, the number of cores to run the code in parallel (also this has to be an integer) 

**PLEASE NOTE:** Always include the value 1 to strides, which is the case of NO striding. The numbers reported here represent the number by which the frame is divided (so stride 10 means that we are keeping 1 frame out of 10).

The sintax of the parameter file should NOT be changed, so when multiple inputs are allowed (ONLY metrics and strides) the user has to write the different inputs separating them with a comma and a space.


## Analysis

First, for every metric chosen, the pipeline computes the Resolution and Relevance curves at different stridings of the trajectory (both uniform and random). To characterize the impact of striding in the different cases, the Multi-Scale Relevance (MSR) values, and the Inter/Intra-cluster covariance matrices traces are computed and plotted.

Then, cosidering only the case with full trajectories (no stride), the clusterings which maximize the Relevance for the different metrics are compared. The idea is to compare these different clusterings. To do so different kind of analysis are performed:

- Coarse Grained trajectories comparison and Adjust Rand Index (ARI) and Adjusted Mutual Information (AMI)
- Diffusion maps comparison
- Principal Component Analysis


### Diffusion maps

For every metric the code performs a Diffusion Coordinates analysis very similar to the one of the reference article. In this case we consider only the avarage linkage criterion to recostruct the Low-Dimensional representation of the distance matrix. The interesting observables from these plots are: the shape of the scatterplot of the first two DCs (if the LR representation mantains the same shape and how this varies for the different metrics) and the color gradient observed in the LR curve which is backmapped in the HR scatterplot (if it is present also in the HR representation this means that the clustering agglomerates frames which are close in the HR space).
Then the first DC are investigated more in depth for the different metrics, producing scatterplots between the different metrics and the confusion matrix heatmap. All these analysis can quantify how similar the various first DCs are.


### ARI and AMI

These two coefficients are a direct similarity measure of two clusterings partitions of the same dataset. They are standard evaluation tools in data science and machine learning fields. Here their standard implementation in the scikit-learn library is employed.
Further information can be found in the manual: [ARI](https://scikit-learn.org/dev/modules/generated/sklearn.metrics.adjusted_rand_score.html) and [AMI](https://scikit-learn.org/1.5/modules/generated/sklearn.metrics.adjusted_mutual_info_score.html)


### CG trajectories comparison and PCA

Every clustering obtained previously groups togheter frames of the trajectory.   
We can observe how some structural observables (basically the ones we use to compute the distances) behave in the CG trajectories respect to the High Resolution trajectory, by a sobstitution of the observable value at every frame with the corresponding cluster avarage value. We can observe directly this comparison, or thanks to histogram distribution 

Instead, to perform PCA it is necessary to re-build the whole trajectory. For every cluster it is possible to define a representative frame: this is the closest frame to the cluster centroid. So we re-build the trajectory substituing the frames with the representative frame of the cluster at which they belong to. These CG trajectories are stored in: `save_and_load/AA/PCA/Traj` (in the case atom selection is all, or `save_and_load/C_A/PCA/Traj` for name CA selection).
Then the space spanned by the first N Principal Components (N is choosen directly in the visualize notebook) can be compared for the different metrics with the Root Mean Square Inner Product (RMSIP). More info on the RMSIP can be found [here](https://nrgten.readthedocs.io/en/latest/principal_components.html).


## Data structure and scripts usage

The data structure proposed here allows to ordinately manage all the files produced by the pipeline and is ready to use.
If other selections (compatible with MDAnalysis) want to be used, it is sufficient to create a directory with identical structure of `C_A` (or, equivalently, `AA`) naming it as preferred, in this example `CUSTOM`. Then it is necessary to include this new save path at the beginning of ALL the scripts like this:
```
if sel == 'name CA':
    save_path = 'save_and_load/C_A'
if sel == 'all':
    save_path = 'save_and_load/AA'
if sel == 'custom':
    save_path = 'save_and_load/CUSTOM'
```


Notice that the scripts read automatically the parameter file, so to run them users can use the command:

```
python distances.py
```

```
python analysis.py
```

**PLEASE NOTE:** The two scripts has to be runned in the order just proposed, and they can consider ONLY ONE atom selection per time. Thanks to the folder structure the various distance matrices and datas produced for different atom selections are stored in different folders, accessible in save_and_load. So to avoid any kind of overwriting it is strongly disencouraged to modify the proposed structure, and if different atoms selections want to be used the previusly shown change of save_path has to be done in ALL the scripts.


### visual.ipynb
This notebook allows to visualize the resusts of the data analysis.
Thanks to the data structure created before, this notebook performs only very simple computations, so it can be runned in a few seconds on a laptop.

Like the other two scripts, the notebook reads from the parameter file the metrics analysed and the atom selection used to compute the distance matrices. If the user is interested in a direct comparison of different atom selections, this notebook could be run multiple times changing the `sel` variable in the parameter file (once of course the previous steps of the pipeline have been performed).



## Python Libraries required

- numpy
- scipy
- multiprocessing
- functools
- configparser
- matplotlib
- MDAnalysis
- mdtraj
- sklearn
