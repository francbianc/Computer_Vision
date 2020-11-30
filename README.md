# Burglary Detection in Surveillance Videos
Implementation of a deep-learning model to detect burglary in surveillance videos, based on the following papers:
- W. Sultani, C. Chen and M. Shah, [Real-world Anomaly Detection in Surveillance Videos](https://arxiv.org/pdf/1801.04264.pdf), *2018 IEEE/CVF Conference on Computer Vision and Pattern Recognition*, Salt Lake City, UT, 2018, pp. 6479-6488, doi: 10.1109/CVPR.2018.00678.
- J. Carreira and A. Zisserman, [Quo Vadis, Action Recognition? A New Model and the Kinetics Dataset](https://arxiv.org/pdf/1705.07750.pdf), *2015 IEEE International Conference on Computer Vision (ICCV)*, Santiago, 2015, pp. 4489-4497, doi: 10.1109/ICCV.2015.510.


## Installing `requirements` (Python 3, Tensorflow 2.3.0) 

```shell script
pip install -r requirements.txt
```

## How to use this repo?

1. **FEATURE EXTRACTION**     
**From videos as `.mp4` to features as `.txt`**   
  * Using C3D Architecture: 
    - `c3d_extract.py`    = for each video in cfg.input_folder, extract the 32 features (32, 4096) and save them in a txt file located in cfg.C3D_path. To run, it needs the following file:
      1. `c3d.py`            = definition of the C3D model, derived by Sultani, Chen and Shah (2019).    
    (`c3d_extract_server.py` can be used if you run the codes on the entire UCF_Crimes (95 GB) train and test dataset)  
  
  * Using I3D Architecture:
    - `i3d_extract.py`    = for each video in cfg.input_folder, extract the 32 features (32, 1024) and save them in a txt file located in cfg.I3D_path. To run, it needs the following file:
      1. `i3d.py`            = definition of the I3D model, derived by Carreira and Zisserman (2018).     
    (`i3d_extract_server.py` can be used if you run the codes on the entire UCF_Crimes (95 GB) train and test dataset)  
  
  If you use as dataset only some videos, put them into the `Input` folder and run the extract code without the "_server" extension. Remember to put in the Input folder both the train and test videos. You don't have to change any variable or path. Results (features) are saved into `C3D_Features` or `I3D_Features`, depending on which architecture you choose. 

  If you use as dataset the entire **UCF_Crimes** dataset, you don't have to put any video in the Input folder, but change *cfg.path_all_videos* in `configuration.py`. This path should refer to the directory of the folder that contains a subfolder per each video category. Make sure the names of all these subfolders are listed in `video_paths.txt`. Then, run the extract codes with the "_server" extension. Results are saved in different folders, located in the directory specified by cfg.path_all_videos, such that all the features of a category are saved in the same folder. 


2. **TRAIN**    
**From features as `.txt` to model's weights as `.mat`, using training data**  
  * Using C3D or I3D Features to train a *Fully-Connected* or *Fully-Connected-with-LSTM* model:
    - `train.py`   = load features in a batch of 60 videos, pass them through a classifier model, compute the loss and perform a backpropagation step at each iteration (20,000 iterations are performed as suggested by Sultani, Chen and Shah(2019)).
    To run, it needs the following files: 
      1. `loss_function.py` = the loss function defined by Sultani, Chen and Shah. 
      2. `load_trainset.py` = function to load extracted features in a batch of 60 videos.
      3. `classifier.py` = the architecture of the training model (4 possibilities: NO_LSTM-C3D, NO_LSTM-I3D, LSTM-C3D, LSTM-I3D).
  
  Before running train.py, according to the experiment you want to implement, change the following variables in:
  >> configuration.py
  - *cfg.path_all_features*
  - *cfg.train_exp_name*
  - *cfg.use_i3d*
  - *cfg.use_lstm*
  - *cfg.train_0_path*
  - *cfg.train_1_path*
  - *cfg.num_0*
  - *cfg.num_1*
  >> train.py 
  - *num_iters*: we choose 20K iterations
  - *batch_size*: we choose 60

  Results (the model and its weights) are saved into a subfolder of the `trained_models` folder. This subfolder derives its name from the experiment name, defined by cfg.train_exp_name. 
  

3. **TEST**    
**From model's weights as `.mat` to predictions on test data as `.txt`** 
  * Using C3D or I3D Features, through a *Fully-Connected* or *Fully-Connected-with-LSTM* model, to get a burglary-score prediction:
  - `test.py` = load extracted features (32, 4096) or (32, 1024), load a pre-trained model, per each test video multiply the corresponding 32 features by the pre-trained model's weights and return 32 burglary scores. To run, it needs the following file:
    1. `classifier.py`
  (`test_server.py` can be used if you run the codes on the entire UCF_Crimes test dataset)

  If you use as test set only some videos, put their features .txt files in the `C3D_Features` or `I3D_Features` folder. Make sure to remove from these folders all the features of those videos that have been used for trainig. Before running test.py, choose which classifier model you want to use by changing the following variables in the configuration.py file:
  - *cfg.classifier_model_weigts*: weights of pre-trained model
  - *cfg.use_lstm*: include a LSTM layer in the classifier
  - *cfg.use_i3d*: input features have been extracted with I3D, hence dim = (32, 1024) instead of (32, 4096)
  Results (predictions for test videos) are saved into the `Scores` folder.  

  If you use as test set the entire *UCF_Crimes* test set, you have to run the test_server.py code. Before running the code, choose which classifier model you want to use by changing the following variables in the configuration.py file:
  - *cfg.classifier_model_weigts*
  - *cfg.use_lstm* 
  - *cfg.use_i3d* 
  - *cfg.train_exp_name*: name of the experiment (must be in line with the classifier_model_weights choice)
  - *cfg.path_all_features*: directory of the folder containing as subfolders features divided by category
  - *cfg.NamesAnn_path*: directory of a .txt file containing the names of the test videos that must be included for the experiment
  Results are saved into a folder, whose name refers to the defined experiment name, that is located in the directory specified by cfg.path_all_features.


4. **AUC - Area Under the Curve**    
**From predictions as `.txt` to AUC**
  * Using predicted burglary-scores to estimate the model accuracy: 
  - `AUC.py` = per each test video, load extracted features, load predictions, load ground truth and compute the Area Under the Curve. 
  (`AUC_server.py` can be used if you run the codes on the entire UCF_Crimes test dataset) 

  If you decide to use as test set only some videos, make sure they are in the `Input` folder, without other videos used for training. Moreover, put only their features in the `C3D_Features` or `I3D_Features` folder and make sure their temporal annotations appear in the txt file specified by *cfg.all_ann_path*. Before running AUC.py, change the following variable in the configuration.py file:
  - *cfg.use_i3d*
  Results (AUC value) are printed.

  If you decide to use as test set the entire *UCF_Crimes* test set, you have to run AUC_server.py. Before running this code, change the following variables in the configuration.py file: 
  - *cfg.train_exp_name*
  - *cfg.use_i3d*
  - *cfg.path_all_features*
  - *cfg.NamesAnn_path*
  - *cfg.Ann_path*: directory of a .txt file containing the names of test videos and their temporal annotations
  Results are printed.


5. **GIF**
**From predictions ad `.txt` to video gif as `.gif`**
  * Using predicted burglary-scores to plot them against the corresponding video's frames:
    - `GIF.py` = create a gif containing the original test video and its burglary-score trend

  Choose a test video, make sure it's in the `Input` folder and its prediction .txt file is in `Score`. Run GIF.py. Results (gif of video and its corresponding scores) are saved in the `GIF` folder. 


6. **VISUALIZE FILTERS / FEATURE MAPS**
**Visualize filters and feature maps of C3D convolutional layers**
  * Filters: choose a convolutional layer and select the number of the filter, specifying which are the units the filter is going to connect. These values are the inputs of the `plot_filter()` function in `c3d_filters_featmaps.ipynb`. Results (plot of one fiter) are displayed directly in the notebook. 

  * Feature maps: choose a video, save its path in the "video_path" variable and if you want to visualize a specific clip, save its number in the "num_clip" variable. If you don't change this variable, the clip would be randomly selected. The clip and the corresponding 16 frames would be saved in a subfolder of `Filters_FeatureMaps`. Then, choose the number of the unit in the first convolutional layer (there are 64 units) and the number of the frame (among the 16 frames of the clip). These last two values are the inputs of the `plot_featmap()` function in `c3d_filters_featmaps.ipynb`. Results (plot of one feature map) are displayed directly in the notebook. 

  ## Training set creation
  ...
