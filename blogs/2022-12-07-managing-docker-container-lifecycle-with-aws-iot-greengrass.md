---
title: "Managing Docker container lifecycle with AWS IoT Greengrass"
url: "https://aws.amazon.com/blogs/iot/managing-docker-container-lifecycle-with-aws-iot-greengrass/"
date: "Wed, 07 Dec 2022 19:09:15 +0000"
author: "Kai-Matthias Dickman"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/feed/"
---
<h2>Introduction</h2> 
<p>In this post, we will be discussing how to manage <a href="https://www.docker.com/">Docker</a> container lifecycle using an <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/develop-greengrass-components.html">AWS IoT Greengrass custom component</a>. There are five phases in a Docker container lifecycle: create, run, pause/unpause, stop, and kill. The custom component interacts with the Docker Engine via the <a href="https://docker-py.readthedocs.io/en/stable/">Docker SDK for Python</a> to manage processes based on your use case, such as user initiated commands or an application sending commands.</p> 
<p><a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html">AWS IoT Greengrass</a> is an open source Internet of Things (IoT) edge runtime and cloud service that helps you build, deploy and manage IoT applications on your devices. You can use AWS IoT Greengrass to build edge applications using pre-built software modules, called components, that can connect your edge devices to AWS services or third-party services.</p> 
<p>AWS IoT Greengrass <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/develop-greengrass-components.html"><em>components</em></a> can represent applications, runtime installers, libraries, or any code that you would run on a device. You can configure AWS IoT Greengrass components to run a <a href="https://www.docker.com/">Docker</a> container from images stored in the following locations:</p> 
<ul> 
 <li>Public and private image repositories in <a href="https://aws.amazon.com/ecr/">Amazon Elastic Container Registry</a> (Amazon ECR)</li> 
 <li>Public Docker Hub repository</li> 
 <li>Public Docker Trusted Registry</li> 
 <li>Amazon S3 bucket</li> 
</ul> 
<p>While Greengrass components have <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/ipc-component-lifecycle.html">lifecycles</a> of their own that you may interact with, these lifecycles do not support containerized processes. To start, stop, pause and resume a Docker container running on AWS IoT Greengrass, you can use commands such as Docker <a href="https://docs.docker.com/engine/reference/commandline/pause/">pause</a> and Docker <a href="https://docs.docker.com/engine/reference/commandline/unpause/">unpause</a> via a Greengrass component running on a Greengrass core device. The custom lifecycle component, which we will refer to as the ‘lifecycle component’, consists of a Python script that subscribes to an AWS IoT Core MQTT topic and interacts with the Docker Engine.</p> 
<h2>Solution Overview</h2> 
<p>Below is an example workflow and architecture for one possible implementation. With these building blocks in hand you can further expand to fit your specific use case.</p> 
<ol> 
 <li>A user deploys a Docker container component and the lifecycle component to the Greengrass core device.</li> 
 <li>The application publishes a MQTT message to an AWS IoT Core topic. The MQTT message specifies the container name and desired action to be performed. In this example, we send a start command to the container named <code>env</code>.</li> 
 <li>The custom lifecycle component is subscribed to the topic.</li> 
 <li>The lifecycle component receives the message and then interacts with the Docker Engine via the Docker SDK for Python and executes the desired command on the container name specified.</li> 
 <li>Based on the command received from the lifecycle component, the Docker Engine will pause, unpause, start, or stop the specified container.</li> 
</ol> 
<h2>Solution Diagram</h2> 
<p><img alt="Solution Diagram" class="alignnone wp-image-11145 size-full" height="527" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/09/solution-diagram.png" width="977" /></p> 
<h2>Implementation Instructions</h2> 
<h3>Prerequisites</h3> 
<ol> 
 <li>The AWS CLI is <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html">installed and configured</a>.</li> 
 <li>AWS Greengrass Core <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html">installed on your device</a>.</li> 
 <li>Docker Engine is <a href="https://docs.docker.com/engine/install/">installed and running</a>.</li> 
</ol> 
<h3>Deploy a Docker container to a Greengrass Core device</h3> 
<p>Follow the instructions on how to <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/run-docker-container.html">run a Docker container</a> with AWS Greengrass or optionally create and run a container with the Docker Engine itself. Be sure to provide a name for the container, in this example we use the name of <code>env</code>.</p> 
<p>Verify you have a running Docker container and that it has the desired name:</p> 
<pre><code class="lang-bash">docker container ls</code></pre> 
<h3>Create the custom lifecycle component</h3> 
<p>To create a Greengrass component we need to create the Python script that will contain our code and also a Greengrass recipe which will specify the deployment details when the component is deployed to a Greengrass Core device.</p> 
<ol> 
 <li>Create an empty folder and script file named <code>customlifecycle.py</code>. <pre><code class="lang-bash">mkdir -p ~/artifacts &amp;&amp; touch ~/artifacts/customlifecycle.py</code></pre> </li> 
 <li>In your favorite Integrated Development Environment (IDE), open <code>customlifecycle.py</code> and paste the following code. Be sure to save the file. Note: the code snippet below is under an MIT-0 license and is available on <a href="https://github.com/aws-samples/aws-iot-gg-docker-lifecycle">Github</a>. <pre><code class="lang-python">#Imports
import time
import json
import traceback
import docker
import subprocess
import awsiot.greengrasscoreipc
import awsiot.greengrasscoreipc.client as client
from awsiot.greengrasscoreipc.model import (
    IoTCoreMessage,
    QOS,
    SubscribeToIoTCoreRequest
)

TIMEOUT = 10
ipc_client = awsiot.greengrasscoreipc.connect()
topic = "docker"
qos = QOS.AT_MOST_ONCE

#IPC Stream Handler
class StreamHandler(client.SubscribeToIoTCoreStreamHandler):
    def __init__(self):
        super().__init__()

    def on_stream_event(self, event: IoTCoreMessage) -&gt; None:
        message = json.loads(event.message.payload.decode())
        
        try:
            client = docker.from_env()
            name = message["name"]
            command = message["command"]
        
            if command == "start":
                container = client.containers.get(name)
                container.start()
                print("Starting container: " + name)
        
            elif command == "pause":
                container = client.containers.get(name)
                result = json.loads(container.pause())
                print(result)
                print("Pausing container: " + name)
                
            elif command == "unpause":
                container = client.containers.get(name)
                print(container.unpause())
                print("Unpausing container: " + name)
                
            elif command == "stop":
                container = client.containers.get(name)
                container.stop()
                print("Stopping container: " + name)
                
            else:
                print("Error")
            
        except:
            with tempfile.TemporaryFile() as tmp:
            tmp.write("Docker Error")
                
    def on_stream_error(self, error: Exception) -&gt; bool:
        message_string = "Error!"

        return True

    def on_stream_closed(self) -&gt; None:
        pass
        
#Initiate Subscription
request = SubscribeToIoTCoreRequest()
request.topic_name = topic
request.qos = qos
handler = StreamHandler()
operation = ipc_client.new_subscribe_to_iot_core(handler)
future = operation.activate(request)
future_response = operation.get_response()
future_response.result(TIMEOUT)

while True:
    time.sleep(1)

operation.close()
</code></pre> </li> 
 <li>Create a bucket and retrieve your bucket name using the following command. <pre><code class="lang-bash">EPOCH_TIME=$(date +"%s") &amp;&amp; S3_BUCKET=lifecycle-component-$EPOCH_TIME &amp;&amp; aws s3 mb s3://$S3_BUCKET</code></pre> </li> 
 <li>Execute the following command to create a folder and a file to put the recipe into. <pre><code class="lang-bash">mkdir -p ~/recipes &amp;&amp; touch ~/recipes/customlifecycle-1.0.0.json</code></pre> </li> 
 <li>Open the created recipe file <code>customlifecycle-1.0.0.json</code> and paste the following contents. Replace [YOUR BUCKET NAME] with the bucket name retrieved in the step 3. <pre><code class="lang-json">{
    "RecipeFormatVersion": "2020-01-25",
    "ComponentName": "Docker-lifecycle-component",
    "ComponentVersion": "1.0.0",
    "ComponentType": "aws.greengrass.generic",
    "ComponentDescription": "A component that interacts with Docker daemon.",
    "ComponentPublisher": "Amazon",
    "ComponentConfiguration": {
      "DefaultConfiguration": {
        "accessControl": {
          "aws.greengrass.ipc.mqttproxy": {
            "docker_lifecycle:mqttproxy:1": {
              "policyDescription": "Allows access to subscribe to all topics.",
              "operations": [
                "aws.greengrass#SubscribeToIoTCore"
              ],
              "resources": [
                "*"
              ]
            }
          }
        }
      }
    },
    "Manifests": [
      {
        "Lifecycle": {
          "Install": "pip3 install awsiotsdk",
          "Run": "python3 -u {artifacts:path}/customlifecycle.py"
        },
        "Artifacts": [
          {
            "Uri": "s3://[YOUR BUCKET NAME]/customlifecycle.py"
          }
        ]
      }
    ]
  }
</code></pre> </li> 
 <li>Upload the component artifacts to Amazon Simple Storage Service. <pre><code class="lang-bash">aws s3 cp --recursive ~/artifacts/ s3://$S3_BUCKET/</code></pre> </li> 
 <li>Next, we will publish the Greengrass component by running the following command. <pre><code class="lang-bash">cd ~/recipes &amp;&amp; aws greengrassv2 create-component-version --inline-recipe fileb://customlifecycle-1.0.0.json</code></pre> </li> 
 <li>You can now see this has been added to your <a href="https://console.aws.amazon.com/iot/home">AWS IoT Console</a> -&gt; Greengrass -&gt; Components -&gt; My Components.<img alt="AWS Greengrass components" class="alignnone wp-image-11142 size-large" height="406" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/09/component-inage-1024x406.png" width="1024" /></li> 
</ol> 
<h3>Deploy the custom lifecycle component</h3> 
<p>Now we will deploy the custom lifecycle component to your Greengrass Core device using the AWS CLI. Deployments may be applied to Things or Thing Groups. In this case, we will apply the deployment directly to the Greengrass Core thing entity.</p> 
<ol> 
 <li>Create a deployment manifest folder and file using the command below. <pre><code class="lang-bash">mkdir -p ~/deployments &amp;&amp; touch ~/deployments/gg_deployment.json</code></pre> </li> 
 <li>In your IDE, copy and paste the below into the <code>gg_deployment.json</code> file. Update the [targetARN] with your Thing ARN. You may retrieve your Thing ARN from the AWS IoT Core console. Be sure to save the file. <pre><code class="lang-json">{
  "targetArn": "[targetArn]",
  "deploymentName": "Deployment for Custom Docker Lifecycle",
  "components": {
    "Docker-lifecycle-component": {
      "componentVersion": "1.0.0"
    }
  }
}
</code></pre> </li> 
 <li>Create the deployment with the following command. <pre><code class="lang-bash">cd ~/deployments &amp;&amp; aws greengrassv2 create-deployment --cli-input-json file://gg_deployment.json</code></pre> </li> 
 <li>Verify that the component is now running on your Greengrass Core device. It may take several minutes for it to instantiate. <pre><code class="lang-bash">sudo /greengrass/v2/bin/greengrass-cli component list</code></pre> </li> 
</ol> 
<h3>Test the Custom Lifecycle component</h3> 
<ol> 
 <li>Go to &nbsp;<a href="https://console.aws.amazon.com/iot/home">AWS IoT Core</a> console, select the <strong>MQTT test client</strong>.</li> 
 <li>Select <strong>Publish to topic</strong>.</li> 
 <li>In the <strong>Topic name</strong>, enter <code>docker</code></li> 
 <li>In the <strong>Message payload</strong>, copy in the message below. The command syntax will depend on the name and current state of your container. <pre><code class="lang-json">{
  "command":"start",
  "name":"env"
}</code></pre> </li> 
 <li>Verify that the state of your container has changed. <pre><code class="lang-bash">docker container ls</code></pre> </li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this blog post, we explored how to use AWS IoT Greengrass to control a Docker container’s lifecycle. This was achieved using a custom component that subscribes to an AWS IoT Core MQTT topic and uses the message contents to execute commands against the Docker daemon with the Docker SDK for Python.</p> 
<p>To take a deeper dive with AWS IoT Greengrass, including building Greengrass components, check out our <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/5ecc2416-f956-4273-b729-d0d30556013f/en-US">AWS IoT Greengrass V2 Workshop</a>!</p>
