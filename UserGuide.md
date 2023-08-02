mkdir -p $SUBJECTS_DIR/${subject}/mri/orig
mri_convert ${subjT1} $SUBJECTS_DIR/${subject}/documents/users/name/3dbrain/freesurfer/7.4.1/subjects/bert/mri/orig/001.mgz
recon-all -subjid ${subject} -all -time -log logfile -nuintensitycor-3T -sd $SUBJECTS_DIR -parallel
source ~/.bashrc
export EXPERIMENT_DIR=~/documents/users/3dbrain
export SUBJECTS_DIR=$EXPERIMENT_DIR/freesurfer/7.4.1/subjects
cd $SUBJECTS_DIR
export subject=testjb
export subjT1=$SUBJECTS_DIR/Run.nii
mkdir -p $SUBJECTS_DIR/${subject}/mri/orig
mri_convert ${subjT1} $SUBJECTS_DIR/${subject}/mri/orig/001.mgz
recon-all -subjid ${subject} -all -time -log logfile -nuintensitycor-3T -sd $SUBJECTS_DIR -parallel
Ctrl z to pause
Fg to unpause
mri_convert $SUBJECTS_DIR/${subject}/mri/aseg.mgz $SUBJECTS_DIR/subcortical.nii
#Second, binarize all Areas that you're not interested and inverse the binarization
mri_binarize --i $SUBJECTS_DIR/subcortical.nii \
             --match 2 3 24 31 41 42 63 72 77 51 52 13 12 43 50 4 11 26 58 49 10 17 18 53 54 44 5 80 14 15 30 62 \
             --inv \
             --o $SUBJECTS_DIR/bin.nii
# Third, multiply the original aseg.mgz file with the binarized files
fslmaths $SUBJECTS_DIR/subcortical.nii \
         -mul $SUBJECTS_DIR/bin.nii \
         $SUBJECTS_DIR/subcortical.nii.gz


