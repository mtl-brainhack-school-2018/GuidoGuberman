Creating precise and reliable fMRI-based ROIs for tract filtering
================================================================================

## Background
For one of my projects, I am interested in using diffusion-weighted imaging to study the working-memory impairments caused by mild traumatic brain injuries (mTBIs) in children. Previous work from my lab has shown that as they perform a working-memory task, children who sustain mTBIs display decreased brain activity in several areas of the brain. However, it is not clear what gives rise to this altered activity. One possibility is that mTBIs damage the connections between areas of the brain involved in working memory.

## Methods
To test this possibility, I would like to use fMRI activation maps as ROIs to filter tracts that connect areas of the brain related to working-memory. I have 16 concussed individuals and 46 controls (after QA) with DWI and fMRI scans.

DWI parameters are the following:
* 2 scans
* 99 directions each
* 10 b0 images
* Single-shell, bmax=1000

Every subject has undergone preprocessing for DWI, which includes:

* Denoising: using NLSAM algorithm (St-Jean, S., Coupé, P., & Descoteaux, M. (2016))
* Inhomogeneity correction: using N4BiasFieldCorrection
* Eddy and motion correction: using FSL

For every subject, I have computed fODFs using constrained-spherical deconvolution, and I have performed probabilistic tractography using the Particle Filtering Tractography script generated by Maxime Descoteaux.

## The problem
Ideally, I would now apply the fMRI activation blobs onto each person's whole-brain tractogram in order to filter my tracts of interest. The problem is that there is no single solution for how to apply these fMRI blobs. If I use a high threshold, fMRI blobs will be compact, but they will be more restricted towards the grey matter. If I use a lower threshold, I will get more white-matter voxels, but I will now have huge blobs that may capture areas that are not as related to working-memory. I need to find the ideal trade-off between ROI size and precision.

## Idea
To create precise and reliable ROIs from the fMRI activation maps, it would be interesting to calculate the "reliability" of the tracts connecting each major fMRI activation cluster. If my fMRI blobs are large or noisy, presumably the tracts that they will filter will also be noisy. I could re-run tractography by seeding directly from the ROIs and calculate a ratio of the streamline counts and tract spread. I could repeat this process using incremental amounts of seeds per voxel. If the ROIs are unreliable (i.e.: they're not picking up any real tracts), the number of streamlines should increase and the tract spread should too (because you're adding streamlines that don't truly belong to a tract, so they don't necessary need to be huddled together), keeping the ratio of both numbers pretty similar. If the ROIs are reliable, the streamline count should increase but the tract spread should stay the same, making the ratio of the two numbers larger. Tract "reliablity" could be the average of these ratios across all the iterations of this process.

The idea would be to perform this process for different fMRI activation thresholds. The threshold that yields the most reliable ROIs is the optimal threshold.

## Structure of the script
The script should take as input:

1. The raw fMRI activation map
2. The fODF image
3. The map include/map exclude images (see Requirements section)

The script should also take the following arguments:

1. A -m flag that determines whether you want the script to generate the ROIs for you from a raw activation map or whether you want to feed in the ROIs yourself
2. The thresholds to try (depends on option -m)
3. The specific ROI images to try (depends on option -m)

The script should generate as output:

1. The average tract reliablity of each threshold tried, either on the command window or as a txt file.

## Requirements
The script would call the Particle Filtering Tractography algorithm developed by Maxime Descoteaux. That script requires an inclusion and exclusion map (generated by another one of Max's scripts).

The script would also need to call some fMRI tools in order to play around with the thresholds and generate masks from the activation peaks.

## EXTRA
Given this once in a lifetime opportunity to interact with a group of smart and distinguished individuals, I would like to take this script one step further. If the ratio of streamline count to tract spread is indeed a good measure of tract "reliablity", then it is possible to tweak the fMRI-based ROI to maximize this ratio. It would be interesting if this script (or a second one?) could generate a trimmed version of the ROIs that would be maximally "reliable".

## Deliverables
Ideally, I would like to have the script entirely written by the end of the the BrainHack School.
