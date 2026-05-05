---
title: "Training the Amazon SageMaker object detection model and running it on AWS IoT Greengrass – Part 3 of 3: Deploying to the edge"
url: "https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-3-of-3/"
date: "Tue, 03 Jan 2023 22:51:08 +0000"
author: "Angela Wang"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/feed/"
---
<p><em>Post by&nbsp;<a href="https://www.linkedin.com/in/angelaruohanwang">Angela Wang</a> and <a href="https://www.linkedin.com/in/tanner-mcrae-aa728358">Tanner McRae</a>, Senior Engineers on the AWS Solutions Architecture&nbsp;R&amp;D and Innovation team</em></p> 
<p>This post is the third in a series on how to build and deploy a custom object detection model to the edge using Amazon SageMaker and AWS IoT Greengrass. In the previous 2 parts of the series, we walked you through <a href="https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-1-of-3/">preparing training data</a> and <a href="https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-2-of-3/">training a custom object detection model</a> using&nbsp;the built-in SSD algorithm of Amazon&nbsp;SageMaker. You also converted the model output file into a deployable format.&nbsp;In this post, we take the output file and show you how to run the inference on an edge device using&nbsp;AWS IoT Greengrass.</p> 
<p>Here’s a reminder of the architecture that you are building as a whole:</p> 
<p><img alt="architecture diagram of the blog" class="alignnone size-large wp-image-3301" height="569" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/part-3-1024x569.png" width="1024" /></p> 
<h2>Following the steps in your AWS account</h2> 
<p>You are welcome to follow the upcoming steps in your own AWS account and on your edge device. Parts 1 and part 2 are not prerequisites for following this section. You can use either a custom model that you have trained or the example model that we provided (under the&nbsp;<a href="https://cdla.io/permissive-1-0/">CDLA Permissive</a>&nbsp;license).</p> 
<h2>Set up environment and AWS IoT Greengrass Core on&nbsp;an edge device</h2> 
<p>Before you get started installing&nbsp;AWS IoT Greengrass&nbsp;core on your edge device, make sure you check the <a href="https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html#gg-platforms">hardware and OS requirements</a>.&nbsp;For this post,&nbsp;we used an Amazon EC2 instance with the Ubuntu 18.04 AMI. Although it’s not a real&nbsp;edge device, we often find it helpful to use an EC2 instance as a testing and development environment for AWS IoT use cases.</p> 
<h3>Device setup</h3> 
<p>For GPU-enabled devices, make sure you have the&nbsp;GPU drivers, such as&nbsp;<a href="https://developer.nvidia.com/cuda-downloads">CUDA</a>, installed. If your device only has CPUs, you can still run the inference but with slower performance.</p> 
<p>Also make sure to install&nbsp;MXNet&nbsp;and&nbsp;OpenCV, which is required to run the model inference code,&nbsp;on your edge device. For guidance, see documentation <a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/tree/master/greengrass#set-up-environment-and-aws-iot-greengrass-core-on-edge-device">here</a>.</p> 
<p>Next, configure your device, and install the AWS IoT Greengrass core software following the steps in <a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/tree/master/greengrass#set-up-environment-and-install-aws-iot-greengrass-software">Set up environment and install AWS IoT Greengrass software</a>.</p> 
<p>Alternately, launch a test EC2 instance with the preceding setup completed by launching this AWS CloudFormation stack:</p> 
<p><a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=greengrass-object-detection-blog&amp;templateURL=https://greengrass-object-detection-blog.s3.amazonaws.com/cloudformation/ec2.yaml"><img alt="button to launch predefinied cloudformation stack" class="alignnone size-full wp-image-682" height="27" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2017/09/10/cloudformation-launch-stack.png" width="144" /></a></p> 
<h3>Create an AWS IoT Greengrass group</h3> 
<p>Now you are ready to create an&nbsp;AWS IoT Greengrass group in the AWS Cloud.&nbsp;There are few different ways to do this and configure AWS Lambda functions to run your model at the edge:</p> 
<ul> 
 <li>Using&nbsp;<a href="https://github.com/dzimine/greengo"><strong>Greengo</strong></a>, an open-source project for defining AWS IoT Greengrass resources in a config file and managing deployment through command line: this is the option detailed in this post.</li> 
 <li>Using the <strong>AWS IoT Greengrass console</strong>: For steps, see <a href="https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-config.html">Configure AWS IoT Greengrass on AWS IoT</a>.</li> 
 <li>Using <a href="https://aws.amazon.com/cloudformation/"><strong>AWS CloudFormation</strong></a>: For an example setup, see <a href="https://aws.amazon.com/blogs/iot/automating-aws-iot-greengrass-setup-with-aws-cloudformation/">Automating AWS IoT Greengrass Setup With AWS CloudFormation</a>.</li> 
</ul> 
<p>In this post, we walk you through setting up this object detection model using&nbsp;Greengo.&nbsp;Our team prefers the Greengo project to manage AWS IoT Greengrass deployment, especially for use in a rapid prototyping and development setting. We recommend using AWS CloudFormation for managing production environments.</p> 
<p>On a macOS or Linux computer, use&nbsp;<code>git clone</code>&nbsp;to download&nbsp;the <a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/tree/master/greengrass">sample code</a>&nbsp;that we provided if you haven’t done so already. The commands shown here have not been tested on Windows.</p> 
<p>In the&nbsp;<code>greengrass/</code>&nbsp;folder, you see a&nbsp;<code>greengo.yaml</code>&nbsp;file, which defines configurations and Lambda functions for an AWS IoT Greengrass group. The top portion of the file defines the name of the AWS IoT Greengrass group&nbsp;and AWS IoT Greengrass cores:</p> 
<pre><code class="lang-yaml">Group:
  name: GG_Object_Detection
Cores:
  - name: GG_Object_Detection_Core
    key_path: ./certs
    config_path: ./config
    SyncShadow: True</code></pre> 
<p>For the initial setup of the AWS IoT Greengrass group&nbsp;resources in AWS IoT, run the following command in the folder in which you found&nbsp;<code>greengo.yaml</code>.</p> 
<pre><code class="lang-bash">pip install greengo
greengo create</code></pre> 
<p>This creates all AWS IoT Greengrass group artifacts in AWS and places the certificates and&nbsp;<code>config.json</code>&nbsp;for AWS IoT Greengrass Core in&nbsp;<code>./certs/</code>&nbsp;and&nbsp;<code>./config/</code>.</p> 
<p>It also generates a state file in&nbsp;<code>.gg/gg_state.json</code>&nbsp;that references all the right resources during deployment:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">├── .gg
│   └── gg_state.json
├── certs
│   ├── GG_Object_Detection_Core.key
│   ├── GG_Object_Detection_Core.pem
│   └── GG_Object_Detection_Core.pub
├── config
│   └── config.json</code></pre> 
</div> 
<p>Copy the&nbsp;certs&nbsp;and&nbsp;config&nbsp;folder to the edge device (or test EC2 instance) using&nbsp;scp, and then copy them to the&nbsp;<code>/greengrass/certs/</code>&nbsp;and&nbsp;<code>/greengrass/config/</code> directories on the device.</p> 
<pre><code class="lang-bash">sudo cp certs/* /greengrass/certs/
sudo cp config/* /greengrass/config/</code></pre> 
<p>On your device, also download the&nbsp;root CA certificate compatible with the certificates Greengo generated to the&nbsp;<code>/greengrass/certs/</code>&nbsp;folder:</p> 
<pre><code class="lang-bash">cd /greengrass/certs/
sudo wget -O root.ca.pem https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem</code></pre> 
<h3>Start AWS IoT Greengrass core</h3> 
<p>Now you are ready to start the AWS IoT Greengrass core daemon on the edge device.</p> 
<pre><code class="lang-bash">$ sudo /greengrass/ggc/core/greengrassd start
Setting up greengrass daemon
Validating hardlink/softlink protection
Waiting for up to 1m10s for Daemon to start
...
Greengrass successfully started with PID: 4722</code></pre> 
<h3>Initial AWS IoT Greengrass group deployment</h3> 
<p>When the AWS IoT Greengrass daemon is up and running, return to where you have downloaded the code repo from GitHub on your laptop or workstation. Then, go to the <code>greengrass/</code> folder (where <code>greengo.yaml</code> resides) and run the following command:</p> 
<pre><code class="lang-bash">greengo deploy</code></pre> 
<p>This deploys the configurations you define in <code>greengo.yaml</code> to the AWS IoT Greengrass core on the edge device. So far, you haven’t defined any Lambda functions yet in the Greengo configuration, so this deployment just initializes the AWS IoT Greengrass core. You add a Lambda function to the AWS IoT Greengrass setup after you do a quick sanity test in the next step.</p> 
<h2>MXNet inference code</h2> 
<p>At the end of the last post, you used the following inference code on a Jupyter notebook. In the <a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/tree/master/greengrass/run_model"><code>run_model/</code></a>&nbsp;folder, review how you put this into a single Python class&nbsp;<code>MLModel</code>&nbsp;inside&nbsp;<a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/blob/399f953cd0a3a6b13decec9c4d785e7840ab0d99/greengrass/run_model/src/model_loader.py#L26"><code>model_loader.py</code></a>:</p> 
<pre><code class="lang-python">class MLModel(object):
    """
    Loads the pre-trained model, which can be found in /ml/od when running on greengrass core or
    from a different path for testing locally.
    """
    def __init__(self, param_path, label_names=[], input_shapes=[('data', (1, 3, DEFAULT_INPUT_SHAPE, DEFAULT_INPUT_SHAPE))]):
        # use the first GPU device available for inference. If GPU not available, CPU is used
        context = get_ctx()[0]
        # Load the network parameters from default epoch 0
        logging.info('Load network parameters from default epoch 0 with prefix: {}'.format(param_path))
        sym, arg_params, aux_params = mx.model.load_checkpoint(param_path, 0)

        # Load the network into an MXNet module and bind the corresponding parameters
        logging.info('Loading network into mxnet module and binding corresponding parameters: {}'.format(arg_params))
        self.mod = mx.mod.Module(symbol=sym, label_names=label_names, context=context)
        self.mod.bind(for_training=False, data_shapes=input_shapes)
        self.mod.set_params(arg_params, aux_params)

    """
    Takes in an image, reshapes it, and runs it through the loaded MXNet graph for inference returning the top label from the softmax
    """
    def predict_from_file(self, filepath, reshape=(DEFAULT_INPUT_SHAPE, DEFAULT_INPUT_SHAPE)):
        # Switch RGB to BGR format (which ImageNet networks take)
        img = cv2.cvtColor(cv2.imread(filepath), cv2.COLOR_BGR2RGB)
        if img is None:
            return []

        # Resize image to fit network input
        img = cv2.resize(img, reshape)
        img = np.swapaxes(img, 0, 2)
        img = np.swapaxes(img, 1, 2)
        img = img[np.newaxis, :]

        self.mod.forward(Batch([mx.nd.array(img)]))
        prob = self.mod.get_outputs()[0].asnumpy()
        prob = np.squeeze(prob)

        # Grab top result, convert to python list of lists and return
        results = [prob[0].tolist()]
        return results</code></pre> 
<h3>Test inference code on device directly (optional)</h3> 
<p>Although optional, it’s always helpful to run a quick test on the edge device to verify that the other dependencies (MXNet and others) have been set up properly.</p> 
<p>We have written a unit test&nbsp;<code>test_model_loader.py</code>&nbsp;that tests the preceding&nbsp;<code>MLModel</code>&nbsp;class. Review the code&nbsp;in this GitHub repository.</p> 
<p>To run the unit test, download the code and machine learning (ML) model artifacts to the edge device and kick off the unit test:</p> 
<pre><code class="lang-bash">git clone&nbsp;https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model.git
cd&nbsp;amazon-sagemaker-aws-greengrass-custom-object-detection-model/greengrass/run_model/resources/ml/od
wget&nbsp;https://greengrass-object-detection-blog.s3.amazonaws.com/deployable-model/deploy_model_algo_1-0000.params
cd&nbsp;../../..
python&nbsp;-m unittest test.test_model_loader.TestModelLoader</code></pre> 
<p>After the unit tests pass, you can now review how this code can be used inside of an AWS IoT Greengrass Lambda function.</p> 
<h2>Creating your inference pipeline in AWS IoT Greengrass core</h2> 
<p>Now that you have&nbsp;started AWS IoT Greengrass and tested the inference code&nbsp;on the edge device, you are ready to put it all together: create an&nbsp;AWS IoT Greengrass Lambda&nbsp;function that runs the inference code inside the AWS IoT Greengrass core.</p> 
<p>To test the AWS IoT Greengrass Lambda function for inference, create the following pipeline:</p> 
<p><img alt="architecture diagram of greengrass core inference" class="alignnone size-large wp-image-3307" height="294" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/blog-greengrass-testing-1024x294.png" width="1024" /></p> 
<ul> 
 <li>A Lambda function that contains the object detection inference code is running in AWS IoT Greengrass core&nbsp;<strong>BlogInfer</strong>.</li> 
 <li>The AWS&nbsp;IoT topic <strong>blog/infer/input</strong>&nbsp;provides input to the <strong>BlogInfer&nbsp;</strong>Lambda function for the location of the image file on the edge device to do inference on.</li> 
 <li>The IoT topic&nbsp;<strong>blog/infer/output</strong> publishes the prediction output of the&nbsp;<strong>BlogInfer</strong>&nbsp;Lambda function to the AWS IoT message broker in the cloud.</li> 
</ul> 
<h3>Choose lifecycle configuration for AWS IoT Greengrass Lambda function</h3> 
<p>There are two types of AWS IoT Greengrass Lambda functions:&nbsp;<a href="https://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-functions.html#lambda-lifecycle">on-demand or&nbsp;long-lived</a>. For doing ML inference, you must run the model in a&nbsp;long-lived function&nbsp;because loading an ML model into memory can often take 300 ms or longer.</p> 
<p>Running ML inference code in a long-lived AWS IoT Greengrass Lambda function allows you to incur the initialization latency only one time. When the AWS IoT Greengrass core starts up, a single container for a long-running Lambda function is created and stays running. Every invocation of the Lambda function reuses the same container and uses the same ML model that has already been loaded into memory.</p> 
<h3>Create Lambda&nbsp;function code</h3> 
<p>To turn the preceding inference code into a Lambda function, you created a&nbsp;main.py&nbsp;as the entry point for Lambda function. Because this is a long-lived function, initialize the&nbsp;MLModel&nbsp;object outside of the&nbsp;lambda_handler.&nbsp;Code inside the&nbsp;lambda_handler&nbsp;function gets called each time new input is available for your function to process.</p> 
<pre><code class="lang-python">import greengrasssdk
from model_loader import MLModel
import logging
import os
import time
import json

ML_MODEL_BASE_PATH = '/ml/od/'
ML_MODEL_PREFIX = 'deploy_model_algo_1'

# Creating a Greengrass Core sdk client
client = greengrasssdk.client('iot-data')
model = None

# Load the model at startup
def initialize(param_path=os.path.join(ML_MODEL_BASE_PATH, ML_MODEL_PREFIX)):
    global model
    model = MLModel(param_path)

def lambda_handler(event, context):
    """
    Gets called each time the function gets invoked.
    """
    if 'filepath' not in event:
        logging.info('filepath is not in input event. nothing to do. returning.')
        return None

    filepath = event['filepath']
    logging.info('predicting on image at filepath: {}'.format(filepath))
    start = int(round(time.time() * 1000))
    prediction = model.predict_from_file(filepath)
    end = int(round(time.time() * 1000))

    logging.info('Prediction: {} for file: {} in: {}'.format(prediction, filepath, end - start))

    response = {
        'prediction': prediction,
        'timestamp': time.time(),
        'filepath': filepath
    }
    client.publish(topic='blog/infer/output', payload=json.dumps(response))
    return response

# If this path exists, then this code is running on the greengrass core and has the ML resources to initialize.
if os.path.exists(ML_MODEL_BASE_PATH):
    initialize()
else:
    logging.info('{} does not exist and you cannot initialize this Lambda function.'.format(ML_MODEL_BASE_PATH))

</code></pre> 
<h3>Configure ML resource in AWS IoT Greengrass using Greengo</h3> 
<p>If you followed the step to run the unit test on the edge device, you had to copy the ML model parameter files manually to the edge device. This is not a scalable way of managing the deployment of the ML&nbsp;model artifacts.</p> 
<p>What if you regularly retrain your ML model and continue to deploy newer versions of your ML model? What if you have multiple edge devices that all should receive the new ML model? To simplify the process of deploying a new&nbsp;ML&nbsp;model artifact to the edge,&nbsp;AWS IoT Greengrass supports the management of&nbsp;<a href="https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-inference.html">machine learning resources</a>.</p> 
<p>When you define an ML resource in&nbsp;AWS IoT Greengrass,&nbsp;you add the resources to an AWS IoT Greengrass group. You define how Lambda functions in the group can access them. As part of AWS IoT Greengrass group deployment, AWS IoT Greengrass downloads the ML artifacts from Amazon S3 and extracts them to directories inside the Lambda runtime namespace.</p> 
<p>Then your AWS IoT Greengrass Lambda function can use the locally deployed models to perform inference. When your ML model artifacts have a new version to be deployed, you must redeploy the AWS IoT Greengrass group. The AWS IoT Greengrass service automatically checks if the source file has changed and only download the new version if there is an update.</p> 
<p>To define the machine learning resource in your AWS IoT Greengrass group, uncomment this section in your&nbsp;<code>greengo.yaml</code>&nbsp;file. (To use your own model, replace the&nbsp;<code>S3Uri</code>&nbsp;with your own values.)</p> 
<pre><code class="lang-yaml">Resources:
  - Name: MyObjectDetectionModel
    Id: MyObjectDetectionModel
    S3MachineLearningModelResourceData:
      DestinationPath: /ml/od/     
      S3Uri: s3://greengrass-object-detection-blog/deployable-model/deploy_model.tar.gz</code></pre> 
<p>Use the following command to deploy the configuration change:</p> 
<pre><code class="lang-bash">greengo update &amp;&amp; greengo deploy</code></pre> 
<p>In the AWS IoT Greengrass console, you should now see an ML resource created. The following screenshot shows the status of the model as <strong>Unaffiliated</strong>. This is expected, because you haven’t attached it to a Lambda function yet.</p> 
<p><img alt="greengrass console screenshot showing unaffiliated ml resource" class="alignnone size-large wp-image-3309" height="460" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/greengrass-console-ml-resource-1024x460.png" width="1024" /></p> 
<p>To troubleshoot an ML resource deployment, it’s helpful to remember that AWS IoT Greengrass has a containerized architecture. It uses&nbsp;filesystem overlay when it deploys resources such as ML model artifacts.</p> 
<p>In the preceding example, even though you configured the ML model artifacts to be extracted to&nbsp;<code>/ml/od/</code>, AWS IoT Greengrass actually downloads it to something like&nbsp;<code>/greengrass/ggc/deployment/mlmodel/&lt;uuid&gt;/</code>. To your AWS IoT Greengrass local Lambda function that you declare to use this artifact, the extracted files appear to be stored in<code>/ml/od/</code> due to the&nbsp;filesystem overlay.</p> 
<h3>Configure the Lambda function with Greengo</h3> 
<p>To configure your Lambda function and give it access to the machine learning resource previously defined, uncomment&nbsp;your&nbsp;<code>greengo.yaml</code> file:</p> 
<pre><code class="lang-yaml">Lambdas:
  - name: BlogInfer
    handler: main.lambda_handler
    package: ./run_model/src
    alias: dev
    greengrassConfig:
      MemorySize: 900000 # Kb
      Timeout: 10 # seconds
      Pinned: True # True for long-lived functions
      Environment:
        AccessSysfs: True
        ResourceAccessPolicies:
          - ResourceId: MyObjectDetectionModel
            Permission: 'rw'</code></pre> 
<p>You didn’t specify the language runtime for the Lambda function. This is because, at the moment, the Greengo project only supports Lambda functions running python2.7.</p> 
<p>Also, if your edge device doesn’t already have&nbsp;<strong>greengrasssdk</strong> installed, you can install&nbsp;<strong>greengrasssdk</strong> to the <code>./run_model/src/</code>&nbsp;directory and have it included in the Lambda deployment package:</p> 
<pre><code class="lang-bash">cd run_model/src/
pip install greengrasssdk&nbsp;-t&nbsp;.</code></pre> 
<h3>Using GPU-enabled devices</h3> 
<p>If you are using a CPU-only device, you can skip to the next section.</p> 
<p>If you are using an edge device or instance with GPU, you must enable the Lambda function to access the GPU devices, using the&nbsp;<a href="https://docs.aws.amazon.com/greengrass/latest/developerguide/access-local-resources.html">local resources feature</a>&nbsp;of AWS IoT Greengrass.</p> 
<p>To define the device resource in&nbsp;<code>greengo.yaml</code>, uncomment the section under&nbsp;<code>Resources</code>:</p> 
<pre><code class="lang-yaml">  - Name: Nvidia0
    Id: Nvidia0
    LocalDeviceResourceData:
      SourcePath: /dev/nvidia0
      GroupOwnerSetting:
        AutoAddGroupOwner: True
  - Name: Nvidiactl
    Id: Nvidiactl
    LocalDeviceResourceData:
      SourcePath: /dev/nvidiactl
      GroupOwnerSetting:
        AutoAddGroupOwner: True
  - Name: NvidiaUVM
    Id: NvidiaUVM
    LocalDeviceResourceData:
      SourcePath: /dev/nvidia-uvm
      GroupOwnerSetting:
        AutoAddGroupOwner: True
  - Name: NvidiaUVMTools
    Id: NvidiaUVMTools
    LocalDeviceResourceData:
      SourcePath: /dev/nvidia-uvm-tools
      GroupOwnerSetting:
        AutoAddGroupOwner: True</code></pre> 
<p>To enable the inference Lambda function to access the device resources, uncomment the following section inside the&nbsp;ResourceAccessPolicies of the Lambda function.</p> 
<pre><code class="lang-yaml">         - ResourceId: Nvidia0
           Permission: 'rw'
         - ResourceId: Nvidiactl
           Permission: 'rw'
         - ResourceId: NvidiaUVM
           Permission: 'rw'
         - ResourceId: NvidiaUVMTools
           Permission: 'rw'</code></pre> 
<h3>Configure topic subscriptions with Greengo</h3> 
<p>Lastly, to test invoking the inference Lambda function and receiving its outputs, create subscriptions for inputs and outputs for the&nbsp;inference Lambda function. Uncomment this section in your&nbsp;<code>greengo.yaml</code> file:</p> 
<pre><code class="lang-yaml">Subscriptions:
# Test Subscriptions from the cloud
- Source: cloud
  Subject: blog/infer/input
  Target: Lambda::BlogInfer
- Source: Lambda::BlogInfer
  Subject: blog/infer/output
  Target: cloud</code></pre> 
<p>To deploy these configurations to AWS IoT Greengrass, run the following command.</p> 
<pre><code class="lang-bash">greengo update &amp;&amp; greengo deploy</code></pre> 
<p>When the deployment is finished, you can also&nbsp;review the subscription configuration in the AWS IoT Greengrass console:</p> 
<p><img alt="screenshot of greengrass console showing subscriptions" class="alignnone size-large wp-image-3313" height="241" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/greengrass-subscriptions-1024x241.png" width="1024" /></p> 
<p>And if you check the Lambda function that was deployed, you can see that the ML resource is now affiliated with it:</p> 
<p><img alt="screenshot of greengrass console showing affliated ML resource" class="alignnone size-large wp-image-3312" height="487" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/greengrass-console-ml-resource-affliated-1024x487.png" width="1024" /></p> 
<h3>Test AWS IoT Greengrass Lambda function</h3> 
<p>Now you can test invoking the Lambda function. Ensure that you download an image (<a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model/raw/master/greengrass/run_model/resources/img/yellow_box_1_000086.jpg">example</a>)&nbsp;to your edge device, so that you can use it to test the inference.</p> 
<p>In the AWS IoT Greengrass console, choose&nbsp;<strong>Test</strong>, and subscribe to the <strong>blog/infer/output</strong>&nbsp;topic. Then, publish a message&nbsp;<strong>blog/infer/input</strong>&nbsp;specifying the path of the image to do inference on the edge device:</p> 
<p><img alt="screenshot of IOT console testing inference on lambda" class="alignnone size-large wp-image-3315" height="535" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/test-lambda-inference-1024x535.png" width="1024" /></p> 
<p>You should have gotten back bounding box prediction results.</p> 
<h2>Take it to real-time video inference</h2> 
<p>So far, the implementation creates&nbsp;a&nbsp;Lambda function that can do inference on an image file on the edge device. To have it perform real-time inference on a video source, you can extend the architecture. Add another long-lived Lambda function that captures video from a camera, extracts frames from video (similar to what we did in <a href="https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-1-of-3/">part 1</a>), and passes the reference of the image to the ML inference function:</p> 
<p><img alt="greengrass architecture diagram for real-time video inference" class="alignnone size-large wp-image-3316" height="257" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2019/11/26/blog-greengrass-prod-1024x257.png" width="1024" /></p> 
<p>&nbsp;</p> 
<h2>Resource cleanup</h2> 
<p>Keep in mind that you are charged for resources running on AWS, so remember to clean up the resources if you have been following along:</p> 
<ul> 
 <li>Delete the AWS IoT Greengrass group by running&nbsp;<code>greengo remove</code>.</li> 
 <li>Shut down the Amazon SageMaker notebook instance.</li> 
 <li>Delete the AWS CloudFormation stack if you used a test EC2 instance to run AWS IoT Greengrass.</li> 
 <li>Clean up S3 buckets used.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post series, we walked through the process of training and deploying an object detection model to the edge from end to end. We started from capturing training data and shared best practices in selecting training data and getting high-quality labels. We then discussed tips for using the Amazon SageMaker built-in object detection model to train your custom model and&nbsp;convert&nbsp;the output to a deployable format. Lastly, we walked through setting up the edge device and using AWS IoT Greengrass to simplify code deployment to the edge and sending the prediction output to the cloud.</p> 
<p>We see so much potential for running object detection models at the edge to improve processes in manufacturing, supply chain, and retail. We are excited to see what you build with this powerful combination of ML and IoT.</p> 
<p>You can find all the code that we covered at <a href="https://github.com/aws-samples/amazon-sagemaker-aws-greengrass-custom-object-detection-model">this&nbsp;GitHub repository</a>.</p> 
<p>Other posts in this three-part series:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-1-of-3/">Part 1: Preparing training data</a></li> 
 <li><a href="https://aws.amazon.com/blogs/iot/sagemaker-object-detection-greengrass-part-2-of-3/">Part 2: Training a custom object detection model</a></li> 
</ul>
