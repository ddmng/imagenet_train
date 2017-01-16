# Docker Tensorflow retrained Imagenet classifier

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

cd /root

python label_dir.py
```


