---
title: "Synadia builds next generation pill verification systems with AWS IoT and ML"
url: "https://aws.amazon.com/blogs/iot/synadia-builds-next-generation-pill-verification-systems-with-aws-iot-and-ml/"
date: "Thu, 04 Aug 2022 21:10:48 +0000"
author: "Sounavo Dey"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/feed/"
---
<p>U.S. prescription medications costs are approaching $500 billion a year and growing up to 7% annually,&nbsp;according to a <a href="https://www.forbes.com/sites/sethjoseph/2020/10/21/a-prescription-for-americas-drug-pricing-problems/?sh=722970c2554c">House Ways and Means Committee report</a>. In this market, billions of dollars in unused medicines are still wasted annually due to traditional packaging that usually contains more pills or tablets than those prescribed by physicians. Automated pill dispensing is the process of dispensing pills into a pouch/container using an automated process. This is an important step in optimizing this supply chain and avoiding pill wastage. Pharmaceutical companies use visual inspection systems to identify potential packaging errors that are then manually corrected by skilled pharmacists.</p> 
<p>The introduction of these visual inspection systems for multiple pills in a single pouch introduced new challenges in this supply chain. Traditional machine vision applications often rely on rule-based inspection with static images. Over the last two decades, pharmaceutical companies have used these traditional image processing techniques to validate the contents of these pouches with mixed results. Static image validation created a high level of&nbsp;false negative and false positive&nbsp;results, which increased the need for additional manual controls and hardware calibration due to the sensitivity of the image validation. This lack of traceability and auditability&nbsp;proves that existing solutions do not achieve the high-standards the pharmaceutical market requires. The stand-alone nature of these visual inspection systems results in an inefficient process where pharmacists manually open and correct the contents of the prescription and generate higher waste in the process.</p> 
<div class="wp-caption alignnone" id="attachment_9097" style="width: 310px;">
 <img alt="Example of a Pill Pack" class="wp-image-9097 size-medium" height="219" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/PillPack-300x219.png" width="300" />
 <p class="wp-caption-text" id="caption-attachment-9097">Example of a Pill Pack</p>
</div> 
<p>This blog post covers how&nbsp;<a href="https://synadia.com/">Synadia Software b.v (Synadia)</a> and Amazon Web Services (AWS) developed a new cloud-based quality assurance solution for pill validation using machine learning (ML) capabilities. Using AWS&nbsp;technology, the next generation of pill-dispensing machines can verify dispensed pills using self-learning algorithms that automatically adjust for new pills and adapt to local conditions. We present a cloud-based solution that contains machine learning algorithms that leverage all the image history to automatically learn and upgrade the latest pill recognition models and deploy them to the pill-dispensing machines.</p> 
<h2>Current pill-dispensing challenges</h2> 
<p>Today, pill-dispensing machines require canisters to be loaded with pills prior to executing a batch job. De-blistering, which is the action to remove a pill from its blister, is a separate manual, error-prone process which takes place before batch order execution and is performed by a group of trained and licensed professionals.</p> 
<p>Machines take pills from canisters and, based on the order, package pills into plastic pouches. When a batch is ready, strings of pouches are loaded into a separate machine, which performs quality checks to confirm that each pouch has the correct pills and amount. Each quality assurance (QA) machine needs separate training to perform the necessary QA checks. The QA machines flag when they detect discrepancies, which requires an expensive human intervention to resolve. The error rate of such machines is approximately 13%.</p> 
<p>Synadia has developed&nbsp;an automatic pill-dispensing machine for the European market. The solution is comprised of a centrally managed network of connected machines with the capability to dynamically receive input and then dispense and package the required types of pills into pouches. The automated process aims to provide higher accuracy for the de-blistering process to achieve consistent results. Using ML models, Synadia can set up a centralized QA mechanism for pill distribution. This eliminates the need to maintain QA models in each location.</p> 
<h2>Solution walkthrough</h2> 
<div class="wp-caption alignnone" id="attachment_9098" style="width: 889px;">
 <img alt="Reference Architecture of the presented solution" class="wp-image-9098 size-full" height="491" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/RefArch.png" width="879" />
 <p class="wp-caption-text" id="caption-attachment-9098">Reference Architecture of the presented solution</p>
</div> 
<p>QA is setup&nbsp;in two steps:</p> 
<ul> 
 <li>Train: learn from existing data. This step requires massive computing resources and needs to be centralized; therefore, it is implemented on AWS.</li> 
 <li>Inference: make decisions about data. This step needs a lot less computing power and needs near-real time (1 sec) processing. This is achieved by ML Inference on <a href="https://aws.amazon.com/greengrass/">AWS IoT Greengrass</a>.</li> 
</ul> 
<p>Every pill-dispensing machine has AWS IoT Greengrass installed.&nbsp;AWS IoT Greengrass has the ability to route messages locally among devices, between devices, and then the cloud, as well as run machine learning inferences on the device. A camera installed on the pill-dispensing machine takes pictures of the pills. To train the models, the images are sent to <a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a> through AWS IoT Greengrass and stored on <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3)</a>. The images are used by <a href="https://aws.amazon.com/sagemaker">Amazon SageMaker</a> to train the QA model.</p> 
<p>The model inferences get deployed to AWS IoT Greengrass and are executed through an <a href="https://aws.amazon.com/lambda/">AWS Lambda</a> function. Based on the outcome of the inference and predefined rules, an action is taken on whether the pill recognition is correct, providing a notification to the customer.</p> 
<p>Reporting on pill dispensing and supply chain is centralized and reported through <a href="https://aws.amazon.com/quicksight/">Amazon QuickSight</a>. Error codes and operating manuals are stored in Amazon S3 and available for quick search through <a href="https://aws.amazon.com/kendra/">Amazon Kendra</a>.</p> 
<h2>Pill dispensing machine hardware</h2> 
<div class="wp-caption alignnone" id="attachment_9106" style="width: 889px;">
 <img alt="Camera setup in the pill-dispensing machine" class="wp-image-9106 size-full" height="493" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/CameraSetup.png" width="879" />
 <p class="wp-caption-text" id="caption-attachment-9106">Camera setup in the pill-dispensing machine</p>
</div> 
<p>The initial setup consists of a camera connected to Programmable Logic Controller (PLC ) and local compute running AWS IoT Greengrass. To create ideal lighting conditions, a custom flashlight based on a Printed Circuit Board (PCB )that is positioned around the camera. When a pill is dropped at the camera position, the PLC sends an MQTT message to the broker at AWS IoT Greengrass,&nbsp;which executes a Lambda function to trigger the camera. When the image is received and processed, the PLC receives another MQTT message to start the next action.</p> 
<div class="wp-caption alignnone" id="attachment_9105" style="width: 889px;">
 <img alt="This is a model of a next generation pill-dispensing machine that can collect one or more pills from their primary containers placed in the square boxes and dispense them into a pouch into the central outlet." class="wp-image-9105 size-full" height="550" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/next_generation_pill-dispensing_machine.png" width="879" />
 <p class="wp-caption-text" id="caption-attachment-9105">This is a model of a next generation pill-dispensing machine that can collect one or more pills from their primary containers placed in the square boxes and dispense them into a pouch into the central outlet.</p>
</div> 
<div class="wp-caption alignnone" id="attachment_9104" style="width: 889px;">
 <img alt="This is a zoomed version of the pill racks showing the placement of the pills in their primary containers." class="wp-image-9104 size-full" height="510" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/PillRack.png" width="879" />
 <p class="wp-caption-text" id="caption-attachment-9104">This is a zoomed version of the pill racks showing the placement of the pills in their primary containers.</p>
</div> 
<div class="wp-caption alignleft" id="attachment_9103" style="width: 535px;">
 <img alt="Pill dispensing machine canister. A pill falls from the left-hand side conduct (01), and falls inside the canister (02), where a diaphragm waits to be opened for further processing (03)." class="wp-image-9103 size-full" height="445" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/machine_canister.png" width="525" />
 <p class="wp-caption-text" id="caption-attachment-9103">Pill dispensing machine canister. A pill falls from the left-hand side conduct (01), and falls inside the canister (02), where a diaphragm waits to be opened for further processing (03).</p>
</div> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2></h2> 
<h2>Ingesting data into AWS</h2> 
<p>Data ingestion is done through MQTT protocol using AWS IoT Core. The main AWS IoT Greengrass and AWS Lambda application takes snapshots of pills, runs these through a classification model, and then sends this information via MQTT to AWS IoT Core.</p> 
<p>The payload consists of a pill identification coupled with the classification probability. In scenarios where the probability is lower than a predefined threshold, the device can then upload the image to an Amazon S3 bucket for further investigation.</p> 
<h2>Running ML training in the cloud</h2> 
<p>There are many ways to identify the type of pill captured in the image. While the obvious choice would be to use an object detection model, we re-framed the solution to use an image classification model. Images are always expected to contain exactly one pill in a small canister. Hence, by setting up the camera so that it frames only the pill inside the canister large enough to be seen, an image classification model is able to recognize the pill features to discern among pill types. This enables us to use a well-known classification neural network model such as <a href="https://arxiv.org/abs/1512.03385?context=cs">ResNet-50</a> to identify the pills.</p> 
<p>To train the model, we take advantage of <a href="https://en.wikipedia.org/wiki/Transfer_learning">transfer learning</a> to achieve high accuracy with very few samples. We work with a small sample of 200 images, split into 120 images for training, 40 images for validation, and the remaining 40 images for test, representing 8 different pill categories. Transfer learning carries most of the low-level feature detection, thanks to being trained on over 14 million images from the <a href="https://www.image-net.org/">ImageNet dataset</a>, containing 1,000 categories. We train the top portion of the network to learn the specific classifier layers, while freezing the remaining layers with the ImageNet-trained parameters.</p> 
<p>The pill-dispensing machine has metadata about the pill type about to be dispensed, hence we use this as the label for our ground truth annotations. In order to avoid over-fitting on the small set of 120 training images, we use an augmentation protocol that will generate new data to help the model become more robust. After carefully analyzing the data, we observed that the pills were positioned on a circular canister centered in the image, so rotating the image by any angle would generate a new image with the same-looking canister and pill, but with the pill in a different position. We also considered a mirroring flip for robustness. With this simple augmentation protocol, we generated a few thousand images that would help train a more robust model.</p> 
<p>We trained the model using only 5 epochs (iterations over data) with a learning rate of 0.0001, quickly achieving a training and validation accuracy of 100%. We could optionally increase the performance of the model by fine-tuning some of the frozen layers. It’s possible to improve upon a 100% accurate model because models are not optimized against accuracy, but instead against a loss function that measures the confidence of the responses of the model, called categorical cross-entropy (e.g., this is ibuprofen with an 84% confidence). We wanted to improve these confidence percentage results to make the model more robust against images where a pill might look ambiguous and its confidence of prediction is low.</p> 
<p>In order to fine tune the model, we unfroze the last 26 layers of the model and set a slower learning rate of 0.00001. We ran our training script for 5 more epochs, reducing the original validation loss of 0.0079 to 0.0016. The model was still 100% accurate, but became more confident in its predictions.</p> 
<h2>Pill identification with ML inference on the edge</h2> 
<p>There are two ways of deploying a model. In a cloud-based deployment, the input data (an image) is sent from the IoT device upstream, where the model runs inference and returns the result back downstream. This can be a costly and slow solution, since large files need to be sent and processed, increasing latency and costs related to data volume. An edge deployment, however, places the model in the IoT device itself. This way, latency and the costs related to data volume vanish, as images can be processed within the device, and only reporting upstream the responses of the model.</p> 
<p>We deployed the trained model using AWS IoT Greengrass. In order to make inference faster on the edge, we optimize the model using&nbsp;<a href="https://aws.amazon.com/sagemaker/neo/">Amazon SageMaker Neo</a>, an AWS service that is able to compress the model parameters and allows for faster inference without losing performance.&nbsp;Amazon SageMaker Neo requires a much lighter framework to be installed in the edge device, allowing for a simpler setup. Using&nbsp;Amazon SageMaker Neo, we were able to improve the inference speed from 0.1 to 0.03 seconds, preserving the aforementioned 100% accuracy.</p> 
<p>We also considered the inference on the edge as a source of information for continuously improving the model. Since the pill-dispensing machine can provide metadata with the pill type in the canister, we proposed the following approach to identify and improve wrong detections. First, we collected images predicted incorrectly and uploaded them to Amazon S3 with the correct label. Second, we collected images predicted correctly, but with confidence below a certain threshold.</p> 
<p>After collecting enough new images (e.g.,1000), we re-triggered a training process, re-using the latest network parameters to transfer all the pill classification learning to date. This helps the system correct future misclassification, while at the same time improve the confidence on low-scoring predictions. The following architecture illustrates the full process of continuously learning and improving the model by collecting the pill labels from the dispenser.</p> 
<div class="wp-caption alignnone" id="attachment_9102" style="width: 987px;">
 <img alt="AWS architecture of the re-training process for pill recognition model improvement" class="wp-image-9102 size-full" height="727" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/retraining_process-.png" width="977" />
 <p class="wp-caption-text" id="caption-attachment-9102">AWS architecture of the re-training process for pill recognition model improvement</p>
</div> 
<h2>Key learning’s</h2> 
<ol> 
 <li>Initially, the sample size was small. Also, the sampling of pills was not uniform. To improve sample variance, we used data augmentation techniques to increase the amount of data by adding slightly modified copies of already existing data, or newly created synthetic data from existing data. This also helped us remove data bias towards pill categories with more initial samples.</li> 
 <li>Initially, the image captures were zoomed out, which meant that the object of interest (i.e., the pill pack) was not in focus and rather small. After experimenting with the camera position and focus, we found the right level of depth for the captured image, which showed a much larger pill for the machine learning model to recognize its relevant features.</li> 
 <li>Amazon SageMaker Neo allowed us to achieve real time inference while at the same time reduce the footprint of the model artifact and the inference framework in the target device, allowing for a faster and simpler deployment.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>The automated pill-dispensing machine provides enhanced operational efficiency through a growing use of machine learning. Transparent data flow from lower-level physical devices to data analytics in the cloud enables real-time responses from remote locations or by executing inference on the edge, thereby improving prescription accuracy for end customer.</p> 
<p>Using data to improve prescription filling accuracy and operations empowers pharmaceutical companies to deliver new pills and manage the supply chain more effectively. The interconnected systems of pill-dispensing machines and machine learning in cloud are forecast-ed to reduce the burden of cost on patients, increase patient compliance, and leverage the advantages of smart devices that can provide instantaneous responsive healthcare.</p> 
<p>To learn more about AWS IoT and AWS machine learning go to the <a href="https://aws.amazon.com/iot/">AWS IoT documentation</a> and/or <a href="https://aws.amazon.com/machine-learning/">AWS machine learning documentation</a>.</p> 
<h2>About the authors</h2> 
<table style="height: 534px;" width="727"> 
 <tbody> 
  <tr> 
   <td style="width: 20%;"><img alt="" class="alignnone wp-image-9101 size-full" height="141" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/Sounavo.png" width="122" /></td> 
   <td> <p><strong>Sounavo Dey</strong> is Sr Solutions Architect Manufacturing in AWS, focused on IoT and manufacturing helping manufacturers as they transform to Industry 4.0. He helps drive technology innovations helping manufacturers plan future success, deliver solution and systematically transform and ensure incremental business value along the journey.</p> <p>He has wide experience in Industrial IoT and Cloud adoption</p></td> 
  </tr> 
  <tr> 
   <td><img alt="" class="alignnone size-full wp-image-9100" height="145" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/raul.png" width="127" /></td> 
   <td><strong>Raul Diaz Garcia</strong> is Data Scientist in AWS and works with customers across EMEA, where he helps customers enable solutions related to Computer Vision and Machine Learning in the IoT space.</td> 
  </tr> 
  <tr> 
   <td><img alt="" class="alignnone size-full wp-image-9099" height="145" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/08/04/Sebastiaan.png" width="108" /></td> 
   <td><strong>Sebastiaan Wijngaarden</strong> is CDA Data Analytics in AWS and works as CDA in the Professional Services organization focusing on Manufacturing and Supply Chain customers. With over 15 years of experience working in Manufacturing (discrete &amp; process) and other Industrial Customers (Healthcare &amp; Life Sciences, CPG, Energy, Power &amp; Utilities, Chemical, etc.).</td> 
  </tr> 
 </tbody> 
</table> 
<p></p>
