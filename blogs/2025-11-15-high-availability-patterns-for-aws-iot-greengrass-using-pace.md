---
title: "High availability patterns for AWS IoT Greengrass using Pacemaker"
url: "https://aws.amazon.com/blogs/iot/high-availability-patterns-for-aws-iot-greengrass-using-pacemaker/"
date: "Sat, 15 Nov 2025 03:23:08 +0000"
author: "Yong JI"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/feed/"
---
<p>Edge computing downtime in industrial IoT environments can be both inconvenient and costly. Systems at the edge require continuous operation to maintain business continuity. While <a href="https://aws.amazon.com/greengrass/" rel="noopener noreferrer" target="_blank">AWS IoT Greengrass</a> delivers powerful edge computing capabilities, achieving true enterprise-grade high availability requires additional orchestration. This post shows how to use <a href="https://github.com/ClusterLabs/pacemaker" rel="noopener noreferrer" target="_blank">Pacemaker</a>, a cluster resource manager, to build resilient edge infrastructure with automated failover.</p> 
<p>In this walkthrough, you’ll learn to implement active/passive and active/active high availability patterns using Pacemaker with AWS IoT Greengrass, complete with automated failover, state replication, and monitoring integration.</p> 
<h2><strong>The high availability challenge for edge computing</strong></h2> 
<p>Traditional cloud applications benefit from built-in redundancy and auto-scaling, however, applications on the edge face unique challenges:</p> 
<ul> 
 <li><strong>Physical isolation</strong>: Edge devices operate in remote locations with limited connectivity</li> 
 <li><strong>Resource constraints</strong>: Unlike cloud environments, edge resources are finite and precious</li> 
 <li><strong>Service criticality</strong>: Edge failures can halt physical operations immediately</li> 
 <li><strong>Recovery complexity</strong>: Manual intervention at remote sites is expensive and slow</li> 
</ul> 
<p>AWS IoT Greengrass addresses many edge computing challenges, but high availability requires thoughtful architecture beyond a single device deployment.</p> 
<h2><strong>How Pacemaker enhances AWS IoT Greengrass</strong></h2> 
<p>Pacemaker helps you build highly available AWS IoT Greengrass deployments through cluster management capabilities:</p> 
<h3><strong>Proven reliability</strong></h3> 
<ul> 
 <li>Used in mission-critical environments for over a decade</li> 
 <li>Handles complex failure scenarios with sophisticated fencing mechanisms</li> 
 <li>Works in both active/passive and active/active configurations</li> 
</ul> 
<h3><strong>AWS IoT Greengrass-aware resource management</strong></h3> 
<ul> 
 <li>Monitors Greengrass service health and component states</li> 
 <li>Manages shared storage for seamless state transfer</li> 
 <li>Coordinates failover of dependent services and network resources</li> 
</ul> 
<h3><strong>Enterprise-ready integration</strong></h3> 
<ul> 
 <li>Integrates with existing <a href="https://en.wikipedia.org/wiki/Linux" rel="noopener noreferrer" target="_blank">Linux</a> infrastructure management</li> 
 <li>Supports complex dependency chains and resource constraints</li> 
 <li>Provides detailed logging and monitoring for compliance requirements</li> 
</ul> 
<p>Together, these tools keep your edge workloads running during hardware failures or network disruptions.</p> 
<h2><strong>Architecture overview: High availability patterns</strong></h2> 
<p>AWS IoT Greengrass high availability can be implemented using two primary patterns, each optimized for different use cases.</p> 
<h3><strong>Active/Passive configuration: Maximizing data consistency</strong></h3> 
<p>This mode maximizes data consistency and automated failover—ideal for mission-critical applications where data integrity and service continuity are paramount. One node runs Greengrass actively while the other stands ready in standby mode. A software-based, block-level data replication service like Distributed Replicated Block Device (DRBD) ensures instant state synchronization between nodes, enabling failover with zero data loss and maintaining device identity.</p> 
<p><img alt="Greengrass HA Active Passive" class="alignnone wp-image-17281 size-full" height="281" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/11/16/gg-ha-blog-ActivePassive.drawio.png" width="492" /></p> 
<h4><strong><em>Key benefits:</em></strong></h4> 
<p>This configuration ensures complete state preservation during failover with sub-minute downtime, zero data loss for in-flight transactions and critical operations, while maintaining device identity, certificates, and Stream Manager persistence seamlessly.</p> 
<h4><strong><em>Real-world use cases:</em></strong></h4> 
<p>Active/Passive configurations are essential in scenarios requiring zero or minimal data loss, such as in-flight entertainment systems that handle offline payment processing and battery manufacturing facilities where production lines depend on continuous data flow from critical manufacturing sensors and ML model outputs to maintain operational integrity and quality control.</p> 
<h3><strong>Active/Active: Maximum throughput and scalability</strong></h3> 
<p>This mode maximizes throughput and provides horizontal scaling for high-volume workloads. Multiple independent Greengrass instances run simultaneously across cluster nodes, with intelligent load balancing distributing work based on node health and capacity. Each node operates with its own unique device credentials and configurations.</p> 
<p><img alt="Greengrass HA Active Active" class="alignnone size-full wp-image-17280" height="351" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/11/16/gg-ha-blog-ActiveActive.drawio.png" width="501" /></p> 
<h4><strong><em>Key benefits:</em></strong></h4> 
<p>These configurations enable horizontal scaling for high-throughput scenarios, improve resource utilization across nodes, and provide graceful degradation under partial failures.</p> 
<h4><strong><em>Real-world use cases:</em></strong></h4> 
<p>Active/Active configurations are ideal for high-volume scenarios such as automotive parts manufacturing facilities and large-scale manufacturing operations with multiple production lines, where each node handles different line segments to provide both redundancy and increased processing capacity for real-time analytics and anomaly detection.</p> 
<h3><strong>Configuration selection guide</strong></h3> 
<p>Use Active/Passive for applications that require zero data loss, shared state, and device identity preservation. This pattern works well when you need a single point of control and can accept failover times under one minute.Use Active/Active when you need high throughput and horizontal scaling. This pattern suits applications that can operate independently without shared state, where load distribution provides operational benefits, and graceful degradation is preferable to complete failover.</p> 
<h2><strong>How to implement the solution</strong></h2> 
<p>The complete playbook, including detailed configuration examples and testing procedures, is available in the <a href="https://github.com/aws-samples/sample-greengrass-ha-pacemaker" rel="noopener noreferrer" target="_blank">GitHub respository</a>. This provides an Active/Passive implementation automation using <a href="https://github.com/ansible/ansible" rel="noopener noreferrer" target="_blank">Ansible</a> that you can customize for your specific requirements. Active/Active setup steps are also available in <a href="https://github.com/aws-samples/sample-ha-for-greengrass-with-pacemaker/tree/main/docs/MANUAL-SETUP-GUIDE.md" rel="noopener noreferrer" target="_blank">MANUAL-SETUP-GUIDE</a> within the same repository.</p> 
<h3>Setup steps</h3> 
<p><strong>1. Environment setup</strong></p> 
<p>Clone the repository and set up the development environment</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">git clone https://github.com/aws-samples/sample-greengrass-ha-pacemaker.git
cd sample-greengrass-ha-pacemaker
./scripts/setup-dev-env.sh &amp;&amp; source .venv/bin/activate
</code></pre> 
</div> 
<p><strong>2. Configure cluster secrets</strong></p> 
<p>Generate and encrypt cluster credentials using Ansible Vault</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># Create vault password file
echo "your_secure_password" &gt; .vault_pass
chmod 600 .vault_pass
# Auto-generate encrypted secrets
./scripts/setup-vault.sh</code></pre> 
</div> 
<p>This creates `vars/cluster-vault.yml` with encrypted credentials for cluster authentication and DRBD replication.</p> 
<p><strong>3. Prepare Greengrass credentials</strong></p> 
<p><em>Note: This approach is designed for testing and demonstration purposes only.</em></p> 
<p>Download Greengrass installation files from AWS IoT Console.</p> 
<ol> 
 <li>Navigate to AWS IoT Core console → Greengrass → Core devices</li> 
 <li>Click ‘Set up one core device’ → ‘Set up a device with installer download’</li> 
 <li>Name your device (e.g., ‘greengrass-ha-device’)</li> 
 <li>Select or create a Thing Group</li> 
 <li>Download both files and rename them: 
  <ol> 
   <li>Rename hash-setup.sh to greengrass-setup.sh</li> 
   <li>Rename hash.zip to greengrass-certs.zip</li> 
  </ol> </li> 
 <li>Place files in `files/greengrass/` directory</li> 
</ol> 
<p><strong>4. Deploy and configure</strong></p> 
<p>This will deploy <a href="https://aws.amazon.com/ec2/" rel="noopener noreferrer" target="_blank">AWS EC2</a> and necessary resources to test on AWS.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># Deploy infrastructure
make cdk-deploy &amp;&amp; make cdk-inventory
# Retrieve SSH private key
./scripts/get-ssh-key.sh
# Configure HA cluster
ansible-playbook playbooks/setup/system-prerequisites.yml -i inventory/cdk-dev-hosts
ansible-playbook playbooks/setup/configure-ha.yml -i inventory/cdk-dev-hosts --vault-password-file .vault_pass</code></pre> 
</div> 
<p><strong>5. Validate and test</strong></p> 
<p>Check cluster status and optionally, run an automated failover test.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># Check cluster status
ansible node-1 -i inventory/cdk-dev-hosts -m shell -a "sudo pcs status" --become
# Test failover (optional)
ansible-playbook playbooks/testing/test-failover-simulation.yml -i inventory/cdk-dev-hosts --vault-password-file .vault_pass</code></pre> 
</div> 
<p>The automated tests validate resource migration, DRBD promotion, and data consistency during failover.</p> 
<h3>Cleanup</h3> 
<p>This will destroy the resources created by CDK.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># Destroy infrastructure
make cdk-destroy</code></pre> 
</div> 
<h2><strong>Conclusion: Enterprise-ready edge computing</strong></h2> 
<p>AWS IoT Greengrass and Pacemaker together provide the high availability needed for mission-critical edge deployments. By using Pacemaker’s cluster management capabilities, organizations can confidently deploy Greengrass where reliability is essential.Whether you’re managing industrial control systems, processing real-time analytics, or orchestrating edge AI workloads, this architectural pattern provides the foundation for resilient, scalable edge computing that your business can depend on.</p> 
<h3>Next steps</h3> 
<p>Ready to implement enterprise-grade high availability for your AWS IoT Greengrass deployments? Here’s your path forward:</p> 
<p>Repository: <a href="https://github.com/aws-samples/sample-greengrass-ha-pacemaker" rel="noopener noreferrer" target="_blank">sample-greengrass-ha-pacemaker</a></p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/greengrass" rel="noopener noreferrer" target="_blank">AWS IoT Greengrass documentation</a></li> 
 <li><a href="https://clusterlabs.org/pacemaker/" rel="noopener noreferrer" target="_blank">Pacemaker documentation</a></li> 
 <li><a href="https://linbit.com/drbd-user-guide" rel="noopener noreferrer" target="_blank">DRBD user’s guide</a></li> 
 <li><a href="https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html-single/configuring_and_managing_high_availability_clusters/index" rel="noopener noreferrer" target="_blank">High Availability cluster best practices</a></li> 
</ul> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-4649 size-full" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/11/15/yongjiff-headshot.png" /><strong>Yong Ji </strong>Yong Ji is a Senior Solutions Architect at Amazon Web Services (AWS), helping enterprises build innovative cloud-based solutions. With over 25 years of experience in cloud architecture, analytics and data engineering, Yong brings deep technical expertise and a passion for solving complex business challenges. Outside of work, Yong is a passionate table tennis player.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-4648 alignleft" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2025/11/15/sidsriv-headshot.png" /><strong>Siddhant Srivastava </strong>Siddhant Srivastava is a Software Development Engineer with AWS IoT Greengrass. He has 3+ years of experience in edge computing with focus on building resilient, scalable distributed systems. Outside work, Siddhant participates in soccer leagues and billiards tournaments.</p>
