# Source code is written in bash so set terminal from zsh to bash
```
source ~/.bashrc
# Set proper file directory. The variable is stored in the folders documents > users > 3dbrain. 
export EXPERIMENT_DIR=~/documents/users/3dbrain
# Create a new directory which stores the first one. 
export SUBJECTS_DIR=$EXPERIMENT_DIR/freesurfer/7.4.1/subjects
# $EXPERIMENT_DIR ends at the 3dbrain folder which contains the freesurfer folder and the following folders within it.
# Folders and directories are arbritarly names as long as each segement leads to the correct files 
```
# Change directory using cd to $SUBJECTS_DIR and set the "subject" to folder for where the model is stored(ex.testjb).  
```
cd $SUBJECTS_DIR
export subject=testjb
export subjT1=$SUBJECTS_DIR/Run.nii
```

# Create initial surface model via Freesurfer.
```
mkdir -p $SUBJECTS_DIR/${subject}/mri/orig
mri_convert ${subjT1} $SUBJECTS_DIR/${subject}/mri/orig/001.mgz
recon-all -subjid ${subject} -all -time -log logfile -nuintensitycor-3T -sd $SUBJECTS_DIR -parallel
```
# The above process can take up to 9 hours to complete. Closing machine can restart progress. Apply these commands to pause
```
Ctrl z to pause
Fg to unpause
```
# The following code creates the 3D model from the right and left hemispheres
```
mris_convert --combinesurfs $SUBJECTS_DIR/${subject}/surf/lh.pial $SUBJECTS_DIR/${subject}/surf/rh.pial \
             $SUBJECTS_DIR/cortical.stl
```
# The model can be viewed in MeshLab and smoothed using the ScaleDependent Laplacian Smooth
```
Smoothing the model contriputes to more efficeint printing as the surface is not as complex.
  - Export the model without binary encoding.
```

# From here, the subcortical regions are created.
```
It should be noted that the subcortical model from Freesurver does not provide as much detail compared to the coritcal one.
Thankfully, the model is created using Freesurfer's aseg.mgz segmentation file.
```
# Begin with converting the segmentation file into NIfTI format
```
mri_convert $SUBJECTS_DIR/$subject/testjb/mri/aseg.mgz $SUBJECTS_DIR/$subject/subcortical.nii
```

# Second, binarize all Areas that you're not interested then inverse the binarization
```
mri_binarize --i $SUBJECTS_DIR/$subject/testjb/subcortical.nii \
             --match 2 3 24 31 41 42 63 72 77 51 52 13 12 43 50 4 11 26 58 49 10 17 18 53 54 44 5 80 14 15 30 62 \
             --inv \
             --o $SUBJECTS_DIR/$subject/testjb/bin.nii
```
# Finally, multiply the original aseg.mgz file with the binarized files
```
fslmaths $SUBJECTS_DIR/$subject/testjb/subcortical.nii \
         -mul $SUBJECTS_DIR/$subject/testjb/bin.nii \
         $SUBJECTS_DIR/$subject/testjb/subcortical.nii.gz
```



