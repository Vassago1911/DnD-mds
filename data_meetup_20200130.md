#meetup 2020-01-30
##ML assisted quality control - andre kochanke (go; reply)
full product cycle: use case, ml, postprocessing, application

production automatisation
conveyor belts and such, ideally: low error rate, highly standardised, high ev speed
how about image based ev?

image based evaluation
where do the images come from? how many? are they labelled?
can they be used for training and inference? 
image data preculiarities: crops, angles, illumination, gradients, resolution, 
identify relevant, ignore irrelevant parts (xkcd 1838)

the job:
pcb (printed circuit boards) industry, need to control surface integrity, 
want to detect corrosion
kpis: how many defects? what is the extent of these relative to the embedding layers?
do surface corrosions occur?

( => pitch: hängt ihr schon ewig daran alle regeln für einen prozess zu definieren? 
ml produkte, wo wirklich nur der learner n frontend kriegt?)

mask r-cnn

relevant parts:  rpn, classifier, pixelwise object mask
transfer learning via public datasets, e.g. coco, imagenet
arxiv: 1506.01497, 1703.06870
github: matterport masked r cnn

imagenet = non-commercial!

1512.03385
resnet 101 (residual neural networks)

50m parameters to train

1506.01497; towardsdatascience yolo object detection
faster r-cnn

1703.06870; 1411.4038

data acquisition:
labelled data from highly trained personnel; lengthy process; resources needed for data acquisition on the order of 
the final product (i.e. getting data needed same resources as the ml part)

model evaluation:
small test set with 50 images
training metrics (loss, acc) gave limited insight
manual ev of hyperparameter tuning
rigorous analysis by customer
also the customer had ev difficulty sometimes
reached trained-human-level performance

postprocessing
ml generates image masks 
corrosion and relevant layer can be identified
rule based approach
mask analysis with 'walker'
goal: statistical analysis of kpis

application
web application; hosted on google cloud platform (gcp)
ev history; user can upload image package

##marc päpper - serving ml models as an inference api in production
mindpeak - increase cancer diagnostic accuracy accessibly to those in need

( => ayasdi in den tda-talk!! ; tesseract / ocr mypdf docker container => digitalisierung consulting)

future cancer diagnosis not for everyone?
demand is increasing, number of qualified people is not

about mindpeak:
automation tools for visual diagnosis in pathology, support experts

training a dl-model: data collection, data annotation, training, evaluation, deployment, monitoring (and back to 1)

today: deployment!

goal: api integration of the model
send an image, to the mindpeak api, receive the json response

first option: serverless lambda function (half-life symbol? :D)

advantages: time to market, no server maintenance, scalable, cost efficient
disadvantages: no gpu, restrictions (small package sizes (meh with pytorch)), cold start problem, vendor lock-in

cool possibility: lambda layers; (like docker)
rough steps to create a lambda layer in python:
create a virtualenv + activate; install your packages; zip and upload to s3, create lambda layer; or better: use docker 

danger (snake with unicorn on top meme)

trick: lambda layer /tmp extraction

.. payload = {'file': base64.encode(image)...}..

minimal example code on cifar - mpaepper/pytorch-serverless (github)  + blog post

second option: gpu servers with docker

advantages: gpu, freedom from dependencies, same environment, quick provisioning
disadvantages: overhead (probably only troublesome in hft/fintech), rising complexity, needs servers, nvidia driver dep

but! nvidia docker images to the rescue! (with cuda and pytorch already in)

making docker access the gpu
before docker 19.03: nvidia-docker2
native gpu support since docker 19.03! migration from versions below docker 19.03 works rather well;
currently still isssues to specify the gpu usage properly in docker-compose files :S

inference architecture:
nginx -> gunicorn -> flask

github: docker-cifar

the real world: rising complexity
use an api gateway; authentication / authorisation / user login
inference speed: calibration, async offloading
stability/security: test, ci/cd, ssl encryption, monitoring notifications, collect and visualise metrics; 
scaling over multiple servers (docker swarm)

many other options:
use c++ to call the traced pytorch model
use a modelserver like redisai
transform it to caffe2 and use a framework which is able to serve that like tensorrt
kubeflow with kubernetes
use full cloud options like aws sagemaker

summary:
many factors to consider; seperate api gateways by environment; if inference is possible on cpu, then serverless is a viable option;
otherwise: docker on cpu or modelservers, docker swarm is a nice alternative to kubernetes; and log and visualise your metrics