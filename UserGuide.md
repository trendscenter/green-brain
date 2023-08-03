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
```
mkdir -p $SUBJECTS_DIR/${subject}/mri/orig
mri_convert ${subjT1} $SUBJECTS_DIR/${subject}/mri/orig/001.mgz
recon-all -subjid ${subject} -all -time -log logfile -nuintensitycor-3T -sd $SUBJECTS_DIR -parallel
```
# This process can take up to 9 hours to complete. Closing machine can restart progress. Apply these commands to pause
```
Ctrl z to pause
Fg to unpause
mri_convert $SUBJECTS_DIR/${subject}/mri/aseg.mgz $SUBJECTS_DIR/subcortical.nii
```
# Second, binarize all Areas that you're not interested and inverse the binarization
mri_binarize --i $SUBJECTS_DIR/subcortical.nii \
             --match 2 3 24 31 41 42 63 72 77 51 52 13 12 43 50 4 11 26 58 49 10 17 18 53 54 44 5 80 14 15 30 62 \
             --inv \
             --o $SUBJECTS_DIR/bin.nii
# Third, multiply the original aseg.mgz file with the binarized files
fslmaths $SUBJECTS_DIR/subcortical.nii \
         -mul $SUBJECTS_DIR/bin.nii \
         $SUBJECTS_DIR/subcortical.nii.gz


