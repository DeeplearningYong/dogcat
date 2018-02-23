1) Enter your home folder, download this code repository, it will create a folder called dogcat: 
```bash
cd ~
git clone https://github.com/DeeplearningYong/dogcat.git
```
If it doesnt work, you can just click the green button clone or download, then download zip, then unzip it, rename the unzipped folder as dogcat 

2) create lmdb data for Caffe to use later
You need to download dog images and cat images from https://www.kaggle.com/c/dogs-vs-cats/data
you will download train and test zip files, unzip them and put them into dogcat/data/trainImgs and dogcat/data/testImgs,
You need to change the lines in code/create_lmdb.py accordingly 
```bash
......
train_lmdb = '/Data/dogcat/data/train_lmdb'
test_lmdb  = '/Data/dogcat/data/test_lmdb'
......
train_data = [img for img in glob.glob("/Data/dogcat/data/trainImgs/*jpg")]
test_data  = [img for img in glob.glob("/Data/dogcat/data/testImgs/*jpg")]
```
and then run: 
```bash
sudo pip install lmdb
python code/create_lmdb.py
```

3) Generate the mean image of training data:
```bash
/home/yourname/caffe/build/tools/compute_image_mean -backend=lmdb /path/to/your/train_lmdb /where-you-want-to-save/mean.binaryproto 
```
You also can skip this step since I have created one under /data

4) Change model definitions accordingly in caffe_models/caffe_model_1/caffenet_train_val_1.prototxt
```bash
......
mean_file: "/Data/dogcat/data/mean.binaryproto"
......
source: "/Data/dogcat/data/train_lmdb"
......
mean_file: "/Data/dogcat/data/mean.binaryproto"
......
source: "/Data/dogcat/data/test_lmdb"
```

Also modify caffe_models/caffe_model_1/solver_1.prototxt
```bash
......
net: "/Data/dogcat/caffe_models/caffe_model_1/caffenet_train_val_1.prototxt"
......
snapshot_prefix: "/Data/dogcat/caffe_models/caffe_model_1/caffe_model_1"
``` 

5) train model
```bash
/home/yourname/caffe/build/tools/caffe train --solver /Data/dogcat/caffe_models/caffe_model_1/solver_1.prototxt 2>&1 | tee /Data/dogcat/caffe_models/caffe_model_1/model_1_train.log
```
You can find trained models under dogcat/caffe_models/caffe_model_1/ 

6) make predictions with new data, you need to modify code/make_predictions_1.py accordingly:
```bash
......
with open('/Data/dogcat/data//mean.binaryproto') as f:
......
net = caffe.Net('/Data/dogcat/caffe_models/caffe_model_1/caffenet_deploy_1.prototxt',
                '/Data/dogcat/caffe_models/caffe_model_1/caffe_model_1_iter_3000.caffemodel',
                caffe.TEST)
......
test_img_paths = [img_path for img_path in glob.glob("/Data/dogcat/data/testImgs/*jpg")]
```

then run: 
```bash
python code/make_predictions_1.py
```
If everything work well, you will observe something like this:
```bash
......
/Data/dogcat/data/testImgs/10129.jpg
1
-------
/Data/dogcat/data/testImgs/1831.jpg
1
-------
/Data/dogcat/data/testImgs/6258.jpg
0
-------
/Data/dogcat/data/testImgs/9225.jpg
0
......
```
You can vidually check those images to see whether the predictions are correct or not.

