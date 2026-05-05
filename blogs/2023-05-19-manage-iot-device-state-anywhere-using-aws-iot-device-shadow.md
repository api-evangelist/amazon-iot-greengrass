---
title: "Manage IoT device state anywhere using AWS IoT Device Shadow service and AWS IoT Greengrass"
url: "https://aws.amazon.com/blogs/iot/manage-iot-device-state-anywhere/"
date: "Fri, 19 May 2023 15:08:01 +0000"
author: "Feng Lu"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/feed/"
---
<h2>Introduction</h2> 
<p>Internet of Things (IoT) developers often need to implement a robust mechanism for managing IoT device state either locally or remotely. A typical example is a smart home radiator, an IoT device where you can use the built-in control panel to adjust the temperature (device state), or trigger temperature adjustment messages remotely from a software application running in the cloud.</p> 
<p>You can quickly build this mechanism by using the <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html">AWS IoT Device Shadow service</a>. The AWS IoT Device Shadow service can make a device’s state available to your business logic, even in the case of intermittent network connection.</p> 
<p>In addition, to efficiently manage your device’s software lifecycle and accelerate your development efforts, you can use <a href="https://aws.amazon.com/greengrass/">AWS IoT Greengrass</a> along with its pre-built components. AWS IoT Greengrass is an open-source edge runtime and cloud service for building, deploying, and managing device software. One of the components of AWS IoT Greengrass is the <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/shadow-manager-component.html">shadow manager</a>, which enables the local shadow service on your core device. The local shadow service allows components to use interprocess communication (IPC) to&nbsp;<a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/ipc-local-shadows.html">interact with local shadows</a>. The shadow manager component manages the storage of local shadow documents, and also handles synchronization of local shadow states with the AWS IoT Device Shadow service.</p> 
<p>In this blog post, I am using AWS IoT Device Shadow service and AWS IoT Greengrass together with a <a href="https://www.raspberrypi.com/">Raspberry Pi</a> and <a href="https://www.raspberrypi.com/products/sense-hat/">Sense HAT</a> hardware to simulate a smart home radiator. This demonstration uses a single digit number (0 – 9) to simulate the output power. This number is the device state that we want to manage from anywhere, local and remote. The user can change this number through a local hardware switch (the built-in joystick on the Sense HAT) as well as remotely from a cloud-based application.</p> 
<p>The Raspberry Pi shows the number on the Sense HAT LED display, indicating the radiator output power. The user can push up on the joystick on the Sense HAT to increase the number (or push down to decrease it).</p> 
<p><img alt="Raspberry Pi simulating home radiator " class="alignnone wp-image-12823" height="388" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Raspberry-Pi-simulating-home-radiator-300x233.png" width="500" /><br /> <em>Figure 1</em><em>. </em><em>Raspberry Pi – simulating home radiator </em></p> 
<p><em><img alt="Architecture overview" class="alignnone size-full wp-image-12844" height="523" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Device-shadow-architecture-1.png" width="951" /><br /> Figure 2. Architecture overview</em></p> 
<p>By following this blog post, you can quickly start building and testing your IoT solutions for managing your device’s state anywhere.</p> 
<h2>Prerequisites</h2> 
<p>To follow through this blog post, you will need:</p> 
<p>Hardware:</p> 
<ul> 
 <li>A <a href="https://www.raspberrypi.com/">Raspberry Pi</a> (This demo is using a Raspberry Pi 3B)</li> 
 <li>A Raspberry Pi <a href="https://www.raspberrypi.com/products/sense-hat/">Sense HAT</a></li> 
</ul> 
<p>Software:</p> 
<ul> 
 <li><a href="https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit">Raspberry Pi OS (64 bit)</a> installed on the Raspberry Pi (refer to <a href="https://www.raspberrypi.com/software/">this link</a> for instruction)</li> 
 <li><a href="https://pythonhosted.org/sense-hat/">Sense HAT SDK</a> installed on the Raspberry Pi</li> 
 <li>An&nbsp;<a href="https://console.aws.amazon.com/">AWS account&nbsp;</a>and permissions to access AWS IoT Core and AWS IoT Greengrass</li> 
</ul> 
<h2>Walkthrough</h2> 
<h3>Step 1: Install and configure the AWS IoT Greengrass core software on the Raspberry Pi.</h3> 
<p>In order to make your Raspberry Pi as an AWS IoT Greengrass core device, follow step 1 to step 3 in the AWS IoT Greengrass <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html">Getting started document</a>. I created the device with the following configuration:</p> 
<ul> 
 <li><strong>Core device name</strong>: PiWithSenseHat</li> 
 <li><strong>Thing group</strong>: RaspberryPiGroup</li> 
</ul> 
<p>Now you should be able to see this device in your AWS console.</p> 
<p><em><img alt="AWS IoT Greengrass core device in console" class="alignnone wp-image-12831 size-full" height="1294" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/1.-Greengrass-core-device-in-console.png" width="2010" /><br /> Figure 3. AWS IoT Greengrass core device in console</em></p> 
<h3>Step 2: Deploy prebuilt AWS IoT Greengrass components to the device</h3> 
<p>The next step is to deploy <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/public-components.html">prebuilt AWS IoT Greengrass components</a> to the device. AWS IoT Greengrass provides and maintains a set of prebuilt components that can accelerate our development. In this demonstration, I am deploying the following components:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-cli-component.html">greengrass.Cli</a>:<br /> Provides a local command-line interface that you can use on core devices to develop and debug components locally</li> 
</ul> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/shadow-manager-component.html">greengrass.ShadowManager</a><br /> Enables the local shadow service on your core device and handles synchronization of local shadow states with the AWS IoT Device Shadow service</li> 
</ul> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/local-debug-console-component.html">greengrass.LocalDebugConsole</a> (optional)<br /> Provides a local dashboard that displays information about your AWS IoT Greengrass core devices and its components</li> 
</ul> 
<h4>Steps:</h4> 
<ol> 
 <li>Go to <a href="https://console.aws.amazon.com/iot/home">AWS IoT Greengrass console</a></li> 
 <li>Navigate to <strong>Deployment</strong> in <strong>Greengrass devices</strong>, create a new deployment</li> 
 <li><strong>Deployment target </strong>could be either <strong>Thing group </strong>RaspberryPiGroup, or <strong>Core device </strong></li> 
 <li>Select these 3 components from <strong>Public components</strong></li> 
</ol> 
<p><em><img alt="Select the prebuilt components" class="alignnone wp-image-12837 size-full" height="748" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Deploy-prebuilt-components.png" width="1112" /><br /> Figure 4. Select the prebuilt components</em></p> 
<ol start="5"> 
 <li>Configure aws.greengrass.ShadowManager component</li> 
</ol> 
<p>In <strong>Configure components</strong> step, select aws.greengrass.ShadowManager, then click <strong>Configure component</strong></p> 
<p><em><img alt="Configure aws.greengrass.ShadowManager " class="alignnone size-full wp-image-12839" height="501" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/3.-Configure-ShadowManager.png" width="1135" /><br /> Figure 5. Configure aws.greengrass.ShadowManager </em></p> 
<ol start="6"> 
 <li>Set up component version and configuration json of aws.greengrass.ShadowManager</li> 
</ol> 
<p><em><img alt="Configure aws.greengrass.ShadowManager details" class="alignnone size-full wp-image-12840" height="917" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Configure-shadowmanager-part-2.png" width="1269" /><br /> Figure 6. Configure aws.greengrass.ShadowManager – details</em></p> 
<ul> 
 <li><strong>Version: </strong>2.3.1</li> 
 <li><strong>Configuration to merge</strong>:<strong> &nbsp;</strong></li> 
</ul> 
<pre style="padding-left: 40px;"><code class="lang-json">{
  "synchronize": {
    "coreThing": {
      "classic": true,
      "namedShadows": [
        "NumberLEDNamedShadow"
      ]
    },
    "shadowDocuments": [],
    "direction": "betweenDeviceAndCloud"
  },
  "rateLimits": {
    "maxOutboundSyncUpdatesPerSecond": 100,
    "maxTotalLocalRequestsRate": 200,
    "maxLocalRequestsPerSecondPerThing": 20
  },
  "shadowDocumentSizeLimitBytes": 8192
}
</code></pre> 
<p>The json configuration synchronizes a <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html">named shadow</a>, called <em>NumberLEDNamedShadow</em> in this example, in both directions, <em>betweenDeviceAndCloud</em> option. In your real-world application, you could use multiple named shadows, and with 1 way or bi-directional synchronization. Check the details of the aws.greengrass.ShadowManager in <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/shadow-manager-component.html">its document</a>.</p> 
<ol start="7"> 
 <li>Complete the <strong>Create Deployment</strong> wizard to finish the deployment.</li> 
</ol> 
<p>At the end of the Step 2, the Raspberry Pi is ready to synchronize a named shadow <em>NumberLEDNamedShadow</em> between device and cloud, by using AWS IoT Greengrass core software and the prebuilt component.</p> 
<h3>Step 3: Create AWS IoT Greengrass components for simulating smart home radiator with local control</h3> 
<p>Now create two AWS IoT Greengrass components for simulating a smart home radiator with local control. We can leverage <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/interprocess-communication.html">interprocess communication</a> (IPC) for the internal communication between the components. If you are not familiar with how to build custom AWS IoT Greengrass components, please follow step 4 in the <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html">Getting started document</a>. In this blog, we create and test them locally.</p> 
<ol> 
 <li>Component <em>example.sensehat.joystick</em>: Capture the events from the joystick and publish the events to an IPC topic <em>“ipc/joystick” </em>(It was defined as a variable in the recipe).</li> 
 <li>Component <em>example.sensehat.led</em>: Subscribe the IPC topic <em>“ipc/joystick”</em>, update the local shadow and the Sense HAT LED display.</li> 
</ol> 
<h4>3.1 Create component <em>com.example.sensehat.joystick</em></h4> 
<p>This component is publishing events of the built-in joystick to AWS IoT Greengrass core IPC. The event is like:</p> 
<pre><code class="lang-json">{
   "timemillis":1669202845134,
   "direction":"down",
   "action":"released"
}</code></pre> 
<p>You can find the component <a href="https://github.com/aws-samples/manage-iot-device-using-device-shadow-blog/blob/main/Components/recipes/com.example.sensehat.joystick-1.0.0.json">recipe</a> and <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/artifacts/com.example.sensehat.joystick/1.0.0/joystick.py">artifact</a> from&nbsp;<a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog">blog source code repo</a>. Instead of hard coding the IPC topic in the source code, it is <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/recipes/com.example.sensehat.joystick-1.0.0.json#L9">defined in the recipe</a> as a variable.</p> 
<h4>3.2 Create component <em>com.example.sensehat.led</em></h4> 
<p>Now create the second component named <em>com.example.sensehat.led</em>. You can find the component <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/recipes/com.example.sensehat.led-1.0.0.json">recipe</a> and <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/artifacts/com.example.sensehat.led/1.0.0/led.py">artifact</a> in the source code repo. In the recipe it <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/recipes/com.example.sensehat.led-1.0.0.json#L9">defines</a> the access permission to IPC and shadow documents.</p> 
<p>This component:</p> 
<ul> 
 <li>Maintains a number as device state, and displays it on LED</li> 
 <li>Subscribes to joystick event topic via IPC</li> 
 <li>Based on the received joystick event, increase/decrease the number</li> 
 <li>Periodically checks the shadow. If there is a new number in the shadow document, update the device with that number.</li> 
</ul> 
<p><em><img alt="Workflow of com.example.sensehat.led component " class="alignnone wp-image-12846 size-full" height="292" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/led-component-lifecycle.png" width="636" /><br /> Figure 7. Workflow of com.example.sensehat.led component </em></p> 
<h2>Demo: Manage the device state in action</h2> 
<p>Now the Raspberry Pi as simulator is ready for use.</p> 
<p>It responds to 2 types of events:</p> 
<ul> 
 <li>New joystick event: Control the device locally</li> 
 <li>New cloud shadow document: Control the device remotely</li> 
</ul> 
<p><img alt="Logic to control the device either locally or remotely" class="alignnone size-full wp-image-12848" height="502" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Logic-to-control-the-device-either-locally-or-remotely.png" width="566" /><br /> Figure 8: Logic to control the device either locally or remotely</p> 
<p>To see the device shadow in action:</p> 
<ol> 
 <li>Go to <a href="https://console.aws.amazon.com/iot/home">AWS IoT Greengrass console</a></li> 
 <li>Navigate to <strong>Things</strong>, select <em>PiWithSenseHat</em></li> 
 <li>In <strong>Device Shadows</strong>, you can find the <em>NumberLEDNamedShadow</em>. Note that you do not need to manually create this shadow. When device reports back the shadow for the first time, it will create it for you if it is missing.</li> 
</ol> 
<p><em><img alt="locate the device shadow in AWS console" class="alignnone wp-image-12849 size-full" height="673" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/locate-the-device-shadow-in-AWS-console.png" width="1427" /><br /> Figure 9. locate the device shadow in AWS console</em></p> 
<h3>Demo 1: Update the device locally by using joystick</h3> 
<ol> 
 <li>Use the joystick to increase/decrease the number locally (The initial number was 6. I firstly deceased it to 0, then increased it to 2).</li> 
 <li>Observe the device shadow document is updated in real time in AWS console. The change is sync to the cloud shadow in the real time.</li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>status is changed to “device updated by local”</li> 
   <li>number is changed to the new value from local</li> 
  </ul> </li> 
</ul> 
<p><em><img alt="Using joystick to update the number" class="alignnone wp-image-12851" height="353" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/use-joystick-to-update-the-number.gif" width="650" /><br /> Figure 10: Using joystick to update the number, and report the new value to cloud in real time</em></p> 
<h3>Demo 2: Update the device remotely by updating device shadow document in cloud</h3> 
<ol> 
 <li>In the beginning of this demo, the device LED was displaying 0, and device shadow document was <pre><code class="lang-json">{
  "state": {
    "desired": {
      "number": 0
    },
    "reported": {
      "status": "device is updated by local",
      "number": 0
    }
  }
}
</code></pre> </li> 
 <li>In AWS IoT Core console, <strong>Edit</strong> the shadow document with the following Json (you can skip the “reported” section), then click <strong>Update<br /> </strong><p></p> <pre><code class="lang-json">{
  "state": {
    "desired": {
      "number":9
    }
  }
}
</code></pre> </li> 
</ol> 
<ol start="3"> 
 <li>Observe the Raspberry Pi LED updates the number. The change is pushed from cloud to local device. Now the device is displaying number 9:</li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>status is changed to “device updated by shadow”</li> 
   <li>number is changed from 0 to 9.</li> 
  </ul> </li> 
</ul> 
<p><img alt="Update the device remotely by updating device shadow document in cloud" class="alignnone wp-image-12861" height="459" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/16/Update-the-device-remotely-by-updating-device-shadow-document-in-cloud.gif" width="650" /><br /> Figure 11. Update the device remotely by updating device shadow document in cloud</p> 
<p>As the display number can be updated either by local joystick or remotely from AWS console, the latest update takes precedence. Therefore when the update is done locally, it is important <a href="https://github.com/aws-samples/manage-IoT-device-using-device-shadow-blog/blob/main/Components/artifacts/com.example.sensehat.led/1.0.0/led.py#L105">to set the “desired” value</a> back to remote shadow in cloud, so the remote shadow knows the new “desired” value and will not update it in the next shadow sync cycle. See more at the document <a href="https://docs.aws.amazon.com/iot/latest/developerguide/device-shadow-document.html#device-shadow-empty-fields">device-shadow-empty-fields</a>.</p> 
<h2>Cleaning up</h2> 
<ul> 
 <li>Delete/disable IAM user which you used for installing AWS IoT Greengrass core software in Raspberry Pi</li> 
 <li>Under AWS IoT console, navigate to <strong>Greengrass devices</strong> 
  <ul> 
   <li>In <strong>Core Device</strong> select the device PiWithSenseHat and Hit delete on top right.</li> 
   <li>In <strong>Thing groups</strong> delete RaspberryPiGroup</li> 
  </ul> </li> 
 <li>Remove these two custom components from Raspberry Pi 
  <ul> 
   <li>Run the following commands in the terminal on Raspberry Pi 
    <ul> 
     <li><code>sudo /greengrass/v2/bin/greengrass-cli --ggcRootPath /greengrass/v2 deployment create --remove "com.example.sensehat.led"</code></li> 
     <li><code>sudo /greengrass/v2/bin/greengrass-cli --ggcRootPath /greengrass/v2 deployment create --remove "com.example.sensehat.joystick"</code></li> 
    </ul> </li> 
   <li>Uninstall AWS IoT Greengrass core software from Raspberry Pi 
    <ul> 
     <li>Follow the steps in <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/uninstall-greengrass-core-v2.html">this document</a> to uninstall the AWS IoT Greengrass core software from your Raspberry Pi.</li> 
    </ul> </li> 
  </ul> </li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post, you learned how to use AWS IoT Device Shadow service and AWS IoT Greengrass to build a robust solution for managing IoT device state, whether it’s done locally or remotely. You can now focus on your own business logic, and let these two AWS services to do the heavy lifting for managing device state anywhere. Currently these two custom components are created and deployed locally in the device. The next step could be making them available in AWS IoT Greengrass, so you can deploy them to more devices. In order to that, you can follow step 5 and step 6 in AWS IoT Greengrass <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html#upload-first-component">document</a>.</p> 
<h2>About the author</h2> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="lufng-1.jpeg"><img alt="Feng Lu" class="alignleft size-full wp-image-5363" height="125" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/05/19/lufng-1.jpeg" width="125" /></p> 
 <p><a href="https://www.linkedin.com/in/linkcd/">Feng Lu</a> is a Senior Solutions Architect in AWS with 18 years professional experience. He is passionate about helping organizations to craft scalable, flexible, and resilient architectures that address their business problems. Currently his focus is on connecting the physical world and cloud with IoT technologies, and uniting computing/AI capacity to make our physical environment smarter and better.</p> 
</div>
