# Image Captioning Model in TensorFlow

This repo follows the blog post from here: 
https://vanishingcodes.wordpress.com/2017/03/20/using-tensorflow-to-build-image-to-text-deep-learning-application/

You can find Image Captioning model implemented using TensorFlow in this repo. The model trains on MSCOCO data set which is downloadable from link: 
http://mscoco.org/dataset/#download

The model is a simplified version of Google's ShowAndTell model: https://github.com/tensorflow/models/tree/master/im2txt#prepare-the-training-data

Basically, the model will extract all image features and save in numpy arrays to local first, and then build the LSTM to train on those features. However, this turned out to be time saving (at the cost of some accuracy, as we are not fine tuning the image model, the encoder, while training LSTM as opposed to Google's approach)

In addition, there are some other differences such as: 
1. Not using emsembling. 
2. Not using partially guided training 
3. Not using BLEU score to monitor validation 

## How to run the scripts

1. Download captions_train2014.json, captions_val2014.json from the link above, as well as train2014(80K) and val2014(40K) images. Save the json files into a folder named ../COCO_captioning/, and train and val images into ../train2014/ and ../val2014/ separately

2. Run command line below to prepare data sets. 
```shell
    python prepare_captions.py --file_dir /home/ubuntu/COCO/dataset/COCO_captioning/ --total_vocab 2000 --padding_len 25
```
3. Run command lines below to extract features using Inception V3 pretrained model, and save them to train2014_v3_pool_3.npy and val2014_v3_pool_3.npy
```shell
    python extract_features.py --model_dir /tmp/imagenet --image_dir /home/ubuntu/COCO/dataset/train2014 --save_dir     /home/ubuntu/COCO/dataset/COCO_captioning/train2014_v3_pool_3 

    python extract_features.py --model_dir /tmp/imagenet --image_dir /home/ubuntu/COCO/dataset/val2014 --save_dir /home/ubuntu/COCO/dataset/COCO_captioning/val2014_v3_pool_3 
```
4. Build and train the model 
```shell
    python train.py -–savedSession_dir [dir where your sessions will be saved] –-data_dir [dir where all training data are saved, which was generated in step 2] –-glove_vocab [path to GloVe word vectors, none if not needed] --sample_dir [dir to save all intermediate validation sample images during training] –-print_every [num of steps to print training loss. 0 for not printing] –-sample_every [num of steps to generate captions on validation images. 0 for not sampling] –-saveModel_every [num of steps to save the model checkpoint. 0 for not saving.]
```
5. Inference on test data folder - using beam search
```shell
    python inference_on_folder_beam.py –-pretrain_dir [path to pretrained v3 model; if not found, will download from web] –-test_dir [path to dir of test images you want to inference on] –-results_dir [path to dir where the test results will be saved] –-saved_sess [saved check point] –-dict_file [path to dictionary file generated in step 2, e.g. coco2014_vocab.json]
```
