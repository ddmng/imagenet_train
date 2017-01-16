Retrained Imagenet classifier in Docker
---
This repository contains the instructions and script to train Imagenet on a set of custom classes, using Tensorflow.
Code and most of the instructions are from Siraj Raval wonderful [video](https://youtu.be/QfNvhPx5Px8) and [repository](https://github.com/llSourcell/tensorflow_image_classifier) and from Google CodeLabs [tutorial](https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/?utm_campaign=chrome_series_machinelearning_063016&utm_source=gdev&utm_medium=yt-desc#0)

# Instructions
So far, there is only a manual procedure. Steps are: create the directory tree and fill with the training and test pictures, create the Tensorflow container bind-mounting the tree, train the network, copy the classifier script in the container, run the classifier over the test images.

## Preparation
`git clone` this repository and create a `tf_files` directory that will contain tree subdirs: `new_pics`, `toScan` and `scanned`:

* `new_pics` will contain a subdirectory for every class you need to train for the new classifier; each subdirectory will contain the training images for that class
* `toScan` contains the pictures that will be submitted to the new classifier predicting function 
* `scanned` contains the classified pictures

So the path would be as follows:

```
cloned_directory
|- new_pics
|  |- class1
|  |- class2
|  |- ...
|- toScan
|- scanned
```

`new_pics` will be bind-mounted in the Tensorflow container where the train script will expect the new classes; `toScan` and `scanned` are used inside the  `label_dir.py` script.

## Steps

Step 1: Start classifier container
```sh
docker run -ti --name tensor_classifier -v $(pwd)/tf_files/toScan:/toScan/ -v \
  $(pwd)/tf_files/scanned:/scanned/ -v $(pwd)/tf_files/new_pics:/new_pics/ \
  gcr.io/tensorflow/tensorflow:latest-devel
```

Step 2: inside the container, launch retrain script
```sh
cd /tensorflow
python tensorflow/examples/image_retraining/retrain.py --bottleneck_dir=/ \
  tf_files/bottlenecks --how_many_training_steps 500 --model_dir /tf_files/ \
  inception --output_graph=/tf_files/retrained_graph.pb \
  --output_labels=/tf_files/retrained_labels.txt \
  --image_dir /new_pics
```

Step 3: after complete retrain, copy label_dir script in the container
```sh
exit

docker cp label_dir.py tensor_classifier:/root
```

Step 4: restart the container and run the classifier
```sh
docker start tensor_classifier
docker attach tensor_classifier
```

Step 5: run the classifier asking the script to put under `scanned` only a particular class you may be interested to.
```sh
python label_dir.py <interesting_class>
```

This will create a copy of each file located in `toScan` to `scanned` renaming it to: `<classname>--<score>`, e.g. `darthvader--58334.jpg` means that the picture matches the "darthvader" class with a 58% score.

The `interesting_class` parameter is mandatory but if you want that the script will copy to `scanned` all the input images with its score just modify a couple of python lines in the script.

## Results
I'm still experimenting with different classes and pictures combinations, so no results yet. Please share any interesting experience you make.
