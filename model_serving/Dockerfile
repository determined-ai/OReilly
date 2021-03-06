FROM tensorflow/tensorflow:1.13.2-py3-jupyter
RUN apt-get update && apt-get install -y \
  git \
  nano \
  vim \
  curl \
  wget

WORKDIR /
ENV DEBIAN_FRONTEND=noninteractive

# configure locale
RUN apt-get update
# make sure that locales package is available
RUN apt-get install --reinstall -y locales
# uncomment chosen locale to enable it's generation
RUN sed -i 's/# pl_PL.UTF-8 UTF-8/pl_PL.UTF-8 UTF-8/' /etc/locale.gen
# generate chosen locale
RUN locale-gen pl_PL.UTF-8
# set system-wide locale settings
ENV LANG pl_PL.UTF-8
ENV LANGUAGE pl_PL
ENV LC_ALL pl_PL.UTF-8
# verify modified configuration
RUN dpkg-reconfigure locales

RUN echo "deb [arch=amd64] http://storage.googleapis.com/tensorflow-serving-apt stable tensorflow-model-server tensorflow-model-server-universal" | tee /etc/apt/sources.list.d/tensorflow-serving.list && curl https://storage.googleapis.com/tensorflow-serving-apt/tensorflow-serving.release.pub.gpg | apt-key add -
RUN apt-get update

# install the required libraries
RUN apt-get install -y protobuf-compiler \
  python-pil \
  python-lxml \
  python-tk \
	tensorflow-model-server \
	libkrb5-dev

RUN pip install Cython \
 contextlib2 \
 jupyter \
 jupyterlab \
 matplotlib \
 scipy \
 requests

# clone the models repo
RUN git clone https://github.com/tensorflow/models.git

WORKDIR /
RUN git clone https://github.com/cocodataset/cocoapi.git
WORKDIR cocoapi/PythonAPI
RUN make
RUN cp -r pycocotools /models/research

WORKDIR /models/research

RUN protoc object_detection/protos/*.proto --python_out=.
RUN echo "export PYTHONPATH=${PYTHONPATH}:`pwd`:`pwd`/slim" >> ~/.bashrc
RUN python setup.py install

# BACKUP Model just in case
WORKDIR /faster_rcnn_inception_v2_coco
RUN wget -nv -O model.tar.gz http://download.tensorflow.org/models/object_detection/faster_rcnn_inception_v2_coco_2018_01_28.tar.gz
RUN tar -xvf model.tar.gz
RUN mkdir -p object-detect/1
RUN find -name saved_model -exec mv {}/saved_model.pb {}/variables object-detect/1/ \;

# TensorFlow Serving Base Directory
RUN mkdir -p /serving/object_detection

EXPOSE 8501
EXPOSE 8888

WORKDIR /
ADD . .

ENTRYPOINT /start-notebook.sh
