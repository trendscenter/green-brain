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

# From here, areas of interest of the subcortical regions are extracted.
```
It should be noted that the subcortical regions from Freesurver does not provide as much detail compared to the coritcal one.
Thankfully, the model is created using Freesurfer's aseg.mgz segmentation file.
```
# Begin with converting the segmentation file into NIfTI format
```
mri_convert $SUBJECTS_DIR/$subject/testjb/mri/aseg.mgz $SUBJECTS_DIR/$subject/subcortical.nii
```

# Second, binarize all areas of interested then inverse the binarization
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

# To create the surface model of the subcortical regions, a temporary file is created, binaraized, and converted into an Stl file
```
# Binarized file is copied to temporary file
cp $SUBJECTS_DIR/$subject/testjb/subcortical.nii.gz $SUBJECTS_DIR/$subject/testjb/subcortical_tmp.nii.gz

# Unzip temp file
gunzip -f $SUBJECTS_DIR/$subject/testjb/subcortical_tmp.nii.gz

# Check all areas of interest for wholes and fill them out if necessary
for i in 7 8 16 28 46 47 60 251 252 253 254 255
do
    mri_pretess $SUBJECTS_DIR/$subject/testjb/subcortical_tmp.nii \
    $i \
    $SUBJECTS_DIR/$subject/testjb/mri/norm.mgz \
    $SUBJECTS_DIR/$subject/testjb/subcortical_tmp.nii
done
```
# The same smoothing filter and binary export settings from the cortical Stl is applied to this model in MeshLab.
```
# Binarize the volume
fslmaths $SUBJECTS_DIR/$subject/testjb/subcortical_tmp.nii -bin $SUBJECTS_DIR/$subject/testjb/subcortical_bin.nii

# Create a surface model of the binarized volume with mri_tessellate
mri_tessellate $SUBJECTS_DIR/$subject/testjb/subcortical_bin.nii.gz 1 $SUBJECTS_DIR/$subject/testjb/subcortical

# Convert surface into stl format
mris_convert $SUBJECTS_DIR/$subject/testjb/subcortical $SUBJECTS_DIR/$subject/testjb/subcortical.stl
```

# Concatenate cortical and subcotical regions to create final brain model.
```
echo 'solid '$SUBJECTS_DIR'/$subject/testjb/final.stl' > $SUBJECTS_DIR/$subject/testjb/final.stl
sed '/solid vcg/d' $SUBJECTS_DIR/$subject/testjb/cortical.stl >> $SUBJECTS_DIR/$subject/testjb/final.stl
sed '/solid vcg/d' $SUBJECTS_DIR/$subject/testjb/subcortical.stl >> $SUBJECTS_DIR/$subject/testjb/final.stl
echo 'endsolid '$SUBJECTS_DIR'/$subject/testjb/final.stl' >> $SUBJECTS_DIR/$subject/testjb/final.stl
```



