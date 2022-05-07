# Digital Voicing of Silent Speech

This is the repository for the papers "Digital Voicing of Silent Speech" at EMNLP 2020 and "An Improved Model for Voicing Silent Speech" at ACL 2021.
The current commit contains only the model for the more recent paper, but the 2020 model can be found in the commit history.
On an ASR-based open vocabulary evaluation, the latest model achieves a WER of approximately 42%.

## trained networks

--pretrained_wavenet_model ./models/wavenet_model/wavenet_model_50_of_50.pt https://drive.google.com/file/d/1_x5Ath-6CRtjoiGXrkTqz1jhhYrAISX_/view?usp=sharing

--models ./models/transduction_model/model_768_07_of_80.pt https://drive.google.com/file/d/1cHkkUC8xbwbCnV76ewwxU2t_GPr5r-jj/view?usp=sharing

--models ./models/transduction_model/model_384_73_of_80.pt https://drive.google.com/file/d/16aTmQDKlkkIYMEc3Eve0UxFCrJdzEIuJ/view?usp=sharing
epoch 73 - validation loss: 0.7192 training loss: 0.4744 phoneme accuracy: 79.60

--models ./models/transduction_model/model_384_76_of_80.pt https://drive.google.com/file/d/16fmwWWdy4VU6bTLs7whmIttjohFZVAoJ/view?usp=sharing
epoch 76 - validation loss: 0.7243 training loss: 0.4647 phoneme accuracy: 79.44

## Data

The EMG and audio data can be downloaded from <https://doi.org/10.5281/zenodo.4064408>.  The scripts expect the data to be located in a `emg_data` subdirectory by default, but the location can be overridden with flags (see the top of `read_emg.py`).

Force-aligned phonemes from the Montreal Forced Aligner can be downloaded from <https://github.com/dgaddy/silent_speech_alignments/raw/main/text_alignments.tar.gz>.
By default, this data is expected to be in a subdirectory `text_alignments`.
Note that there will not be an exception if the directory is not found, but logged phoneme prediction accuracies reporting 100% is a sign that the directory has not been loaded correctly.

## Environment Setup

This code requires Python 3.6 or later.
We strongly recommend running in a new Anaconda environment.

First we will do some conda installs.  Your environment must use CUDA 10.1 exactly, since DeepSpeech was compiled with this version.
```
conda install cudatoolkit=10.1
conda install pytorch -c pytorch
conda install libsndfile=1.0.28 -c conda-forge
```

Pull nv-wavenet into the `nv_wavenet` folder and follow build intructions provided in the repository.
```
git clone https://github.com/NVIDIA/nv-wavenet.git nv_wavenet
# follow build instructions in nv_wavenet/pytorch
```
If you have an older GPU, you may need to use <https://github.com/dgaddy/nv-wavenet> instead, which removes code for 16-bit floating point that only works on newer GPUs.

The rest of the required packages can be installed with pip.
```
pip install absl-py librosa soundfile matplotlib scipy numba jiwer unidecode deepspeech==0.8.2 praat-textgrids
```

Download pre-trained DeepSpeech model files.  It is important that you use DeepSpeech version 0.7.0 model files to maintain consistency of evaluation.  Note that the DeepSpeech pip package we recommend is version 0.8.2 (which uses a more up-to-date CUDA), but this is compatible with version 0.7.x model files.
```
curl -LO https://github.com/mozilla/DeepSpeech/releases/download/v0.7.0/deepspeech-0.7.0-models.pbmm
curl -LO https://github.com/mozilla/DeepSpeech/releases/download/v0.7.0/deepspeech-0.7.0-models.scorer
```

## Running

To train a WaveNet model, use
```
python wavenet_model.py --output_directory "./models/wavenet_model/" --silent_data_directories ""
```

To train an EMG to speech feature transduction model, use
```
python transduction_model.py --pretrained_wavenet_model "./models/wavenet_model/wavenet_model.pt" --output_directory "./models/transduction_model/"
```
At the end of training, an ASR evaluation will be run on the validation set.

Finally, to evaluate a model on the test set, use
```
python evaluate.py --models ./models/transduction_model/model.pt --pretrained_wavenet_model ./models/wavenet_model/wavenet_model.pt --output_directory evaluation_output
```

By default, the scripts now use a larger validation set than was used in the original EMNLP 2020 paper, since the small size of the original set gave WER evaluations a high variance.  If you want to use the original validation set you can add the flag `--testset_file testset_origdev.json`.
