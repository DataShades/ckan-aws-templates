# ckan-aws-templates

Author: shane.davis@linkdigital.com.au

Copyright 2016 Datashades 

**Cloud Formation templates for Datashades AWS OpsWorks Stack.**

**Contents**

* Datashades.VPCs.json: [Optional] Creates dev,uat and production VPCs for OpsWorks stack 
* Datashades-OpsWorks-CKAN-Stack.json: [Required] Creates AWS OpsWorks CKAN Stack
* Datashades-OpsWorks-CKAN-Instances.json: [Optional] Creates initial Instances within AWS OpsWorks CKAN Stack
* Datashades-OpsWorks-CKAN-Extensions.json: [Optional] Creates base CKAN extensions Application definitions within AWS OpsWorks CKAN Stack

**Features:**

* Single click Enterprise grade CKAN deployment
* User selectable VPC.
* HA DB, Solr and Web nodes.
* AWS RDS or EC2 for Postgres
* User selectable instance and data volume sizes

**Assumptions:**

It's assumed that you:

* have a pretty good working knowledge of AWS, CloudFormation, OpsWorks, and CKAN and it's requirements such as Solr and Postgres. Those will be necessary to troubleshoot builds when you haven't provided the correct parameters or some other obstacle gets in your way. 
* have built, installed and successfully run CKAN manually on some kind of single node configuration. If not, this stack isn't designed to be something to cut your teeth on. It's been designed to be relatively foolproof, but not completely so.
* know your way around the Linux command line reasonably well and know how to deal with error logs, dependency conflicts etc.

Installation
------------

***Overview***

* Use the Datashades.VPCs.json CloudFormation template to optionally create a Dev, UAT and Production VPC. You can simply use an existing VPC just fine.
We tend to use this template to create a preprod VPC stack that contains Dev and UAT. We typically use the AWS default VPC for production.
* Use Datashades-OpsWorks-CKAN-Stack.json CloudFormation template to create the actual AWS OpsWorks CKAN stack.
* Optionally use the Datashades-OpsWorks-CKAN-Instances.json CloudFormation template to create the initial instances to run within the stack once the OpsWorks stack is successfully provisioned. See detailed instructions and notes below regarding this template.
* Optionally use the Datashades-OpsWorks-CKAN-Extensions.json CloudFormation template to add CKAN extensions as OpsWorks Application definitions to stack once CKAN is successfully running. See detailed instructions and notes below regarding this template.
* Create Route53 records for Solr

**Datashades.VPCs**

From AWS Console: 

* Create a new Cloud Formation stack.
* Select the upload template option and upload the DatashadesVPCs.json template.

**Template Parameters:**

A recommended name for the Stack is [stack_version]-VPCs. EG: Preprod-VPCs

***General Options:***

* VPCPrefix: Name to prexfix VPC Name tag with. EG: CKAN
* AdminIPQty: How many admin IP addresses you want to supply. (see notes below)
* AdminIPs: Admin IPs in CIDR format. EG: x.x.x.x/x
* SubnetQty: How many subnets your VPC contains. (see notes below)
* SubnetAZs: What AZs you want your subnets to be provisioned in 

*Notes:*

* AdminIPs: The template creates a security group for you in each VPC that permits all traffic from admin IPs. Thus, you might allow all traffic from the static Public IP address of your office. You might also allow all traffic from a Bastion host you have elsewhere in the cloud. 
* SubnetQty: The template assumes a maximum of three availability zones even though some regions have more than three. The subnet quantity needs to match the number of subnets you choose for SubnetAZs

***Development VPC:***

InclDev: Boolean for whether or not to build a dev VPC.
DevCIDR: The CIDR for the Development VPC
DevSubnetCIDRs: Comma delimited list of CIDRs for selected subnets (see notes below)

*Notes:*

* DevSubnetCIDRs: The CIDR blocks for subnets must be within the range of the VPC CIDR.
When you select SubnetAZs, CIDRs will be allocated in sequencial order. Thus if you have a preference for AZ C as the primary subnet AZ 
because you have the most EC2 reserved instances there, AZ C will be the first subnet, and allocated the first CIDR block in the range.
To understand this further, you might select AZ C, AZ B, AZ A, in which case your CIDRs will be allocated in that order.
As much as anything this approach is about preferencing.

***UAT, Production VPC:***

These essentially have exactly the same parameters as the development VPC.
It makes the most amount of sense to create preprod VPCs together, and Production separately. There's a stronger possibility you'd
want to scrap your Dev and UAT VPCs to rebuild them from scratch than your Production VPC. For the vast majority of people, they
wouldn't rebuild their VPCs at all. 

**Datashades-OpsWorks-CKAN-Stack.json**

From AWS Console: 

* Create a new Cloud Formation stack.
* Select the upload template option and upload the Datashades-OpsWorks-CKAN-Stack.json template.

**Template Parameters:**

A recommended name for the Stack is [stack_version]-CKAN. EG: DEV-CKAN

***OpsWorks Custom Cookbook:***

Used to define parameters for Datashades opswx-ckan-cookbook location which may be the public Datashades repo (default) or a private repo location such as GitHub Enterprise or AWS CodeCommit.

* Cookbook URL: Git repository URL. Default: Datashades public repo
* Cookbook SSH Key: Allows you to provide the private key for accessing private repos such as GitHub Enterprise.
* Cookbook Revision: Branch you wish to checkout. Default: master

***OpsWorks Stack Details:***

Allows you to define specific aspects of the AWS OpsWorks stack that will be built.

* CreateServiceRole: Boolean to determine whether an OpsWorks service role should be created. If not supplied, template attempts to use existing aws-opsworks-service-role created by OpsWorks when a initial OpsWorks stack is created via the console.
* StackVersion: Dropdown list for dev,uat,prod etc.
* StackTLD: The root domain for automated DNS management. EG: ckan.mydomain.com
* CreateTLD: Whether or not to create the Hosted Zone in Route53 for the StackTLD. If false, a Route53 Hosted Zone for StackTLD must already exist
* NFSEC2Size: What instance size you want for the GFS/NFS role
* WebEC2Size: What instance size you want for web nodes
* DefaultEC2Key: The name of the SSH key stored in AWS EC2 Key Pairs that should be assigned to instances in the stack by default 
* StackVPC: The VPC you want the OpsWorks stack to provision EC2 instances into.
* PrimarySubnet: The primary subnet within the StackVPC you want primary instances provisioned into. 
* SecondarySubnet: The secondary subnet within the StackVPC you want secondary instances provisioned into. For true HA, you should select subnets that are in seperate Availability Zones 
* AdminSG: The VPC security group that contains firewall rules for your admins to access stack instances. This allows you to adhere to the best practise of not opening SSH or other ports to the world, but instead limiting SSH to known static IP addresses such as a Bastion Host. 
* RDSSize: What instance size RDS should be provisioned on.

***OpsWorks Stack Details:***

* DataVolSize: What size the data volume for CKAN filestore and CKAN resources, logs etc should be. 
* PGDBPassword: The password ckan_default DB user should use. This will be a root Postgres user. 
* NFSCIDR: The CIDR range NFS/GFS should allow. Should match your VPC CIDR

***Application Sources:***

The application sources are important to keep up to date. You might want to change the default for Solr to something you know keeps archives of older versions so you don't have to keep changing it. The mirrors for example drop old versions. 

* SolrSource: Expects a zip archive. You'll want to change the default to a mirror that's much closer to you.
* ZooKeeperSource: Expects a tar.gz archive. You'll want to change the default to a mirror that's much closer to you.
* CKANSource: Expects a zip archive.

**NOTES:**

The template itself doesn't do a lot of error checking per se. When the template is run by CloudFormation, pay attention to errors. 
The OpsWorks stack itself can be created with no problems, but when you start to add instances, and run setup recipes, things fail because some of the parameters you provided weren't correct.
If you pay attention to the errors you receive and address them accordingly, you'll be successful. 
A very common issue is mirrors not supporting the version of solr you've stipulated. In this scenario, the template will successfully create the OpsWorks stack, but the reset recipe will fail for the solr layer complaining the archive is unavailable. In this circumstance you can modify the Solr application source URL in the OpsWorks stack created to resolve the issue.

Some of the parameters in this stack aren't technically required. This template originally provisioned instances as well, and the parameters were required for that. Since separation the parameters haven't been removed.
In future revisions however, the instances template may be reintroduced as a sub template, and the parameters may become relevant again. In the meanwhile they are somewhat harmless.

**Datashades-OpsWorks-CKAN-Instances.json**

From AWS Console: 

* Create a new Cloud Formation stack.
* Select the upload template option and upload the Datashades-OpsWorks-CKAN-Instances.json template.

**IMPORTANT**

Until you are somewhat familar with building this stack it's highly recommended that you turn the CloudFormation rollback option OFF 
when provisioning instances.
When OpsWorks instances are provisioned via CloudFormation, they are started, and the setup and deploy recipes are run on boot.
There's quite a lot that can fail on initial builds. Most of them are small repairable errors, but with rollback enabled, you won't be able to
review the deployment logs easily and determine what went wrong.
9 times out of 10 you can run the setup command a second time and have the instance come online correctly.
The Instances template is provided to illustate the order in which instances within the stack need to be provisioned.
Essentially you create the first services node, wait for it to come online. Add the second services node, get it online with SolrCloud running
correctly, then you will have a smooth CKAN Web node install.
 
**Template Parameters:**

A recommended name for the Stack is [stack_version]-CKAN-INSTANCES. EG: DEV-CKAN-INSTANCES

***OpsWorks Stack Details:***

This aspect of the template could do with some improvement either by making it a sub-template so these parameters aren't required, or using a somewhat
different approach. None the less, the template is important for illustrating how instances should be allocated to layers. In theory each layer should be
able to have it's own instances, but in this initial implementation there's a necessity and assumption that Zookeeper will run on the same instances
as Solr, and SolrCloud will run on the same instances as GFS.
In the longer run, AWS Elastic File Store or S3 would be used in prefence of Gluster. Gluster, and NFS before that were used for pure convience of wanting
and needing at the time a block volume.
The fastest easiest way to grab these parameters is to view the Outputs from the Datashades-OpsWorks-CKAN-Stack.json CloudFormation stack. Have them displayed
in one browser window while you provision this stack in another browser window.
The Redis layer is screaming out to be replaced by the AWS Redis service.

* OpsWorksStack: ID for the OpsWorks stack instances should be provisioned in.
* GFSLayerID: ID of the GFS Layer.
* SolrLayerID: ID of the Solr Layer.
* RedisLayerID: ID of Redis Layer.
* WebLayerID: ID of the Web Layer.

***Stack Instance Details:***

* DefaultEC2Key: The name of the SSH key stored in AWS EC2 Key Pairs that should be assigned to instances in the stack by default 
* NFSEC2Size: What instance size you want for the GFS/NFS role
* WebEC2Size: What instance size you want for web nodes
* PrimarySubnet: The primary subnet within the StackVPC you want primary instances provisioned into. 
* SecondarySubnet: The secondary subnet within the StackVPC you want secondary instances provisioned into. For true HA, you should select subnets that are in seperate Availability Zones 

**Datashades-OpsWorks-CKAN-Extensions.json**

From AWS Console: 

* Create a new Cloud Formation stack.
* Select the upload template option and upload the Datashades-OpsWorks-CKAN-Extensions.json template.

**IMPORTANT**

It's strongly recommended that you get CKAN working in it's own right before adding extension definitions. Once you're more
comfortable with deploying the stack, and know all the gotcha's and quirks, it's simple enough to add extensions from the get
go. Initially however, it's easier to deal with one problem at a time. Some people might prefer of course to trouble shoot
everything in a single shot. I suspect seasoned CKAN admins will go with the second option.

**Template Parameters:**

A recommended name for the Stack is [stack_version]-CKAN-EXTENSIONS. EG: DEV-CKAN-EXTENSIONS

***Stack Parameters:***

* OpsWorksStack: ID for the OpsWorks stack CKAN extensions Application definitions should be provisioned in.

*NOTES*

Once this template has been provisioned, navigate to the stack in the AWS console and deploy extensions to the Web Layer.

As things stand the recipes deploy ALL extensions all at once. This is both desired and undesired dpending on your perspective.
In a perfectly ideal world, some would prefer extensions to be independently deployed one by one. This is can still be accomplished
by adding the Application definition for extensions one by one, and deploying after each addition.

In most environments, when deploying new instances to the Web Layer, you really do want all the extensions to be deployed all at once.

**Solr Route53 Records**

There's probably a much better way of allowing CKAN to talk to SolrCloud, but this is an approach we know works, and isn't difficult to
implement via Route53 even if it has Ninja and Hacker stamped all over it.

* Create a CNAME record [version]solr.[tld] that is an Alias to [version]solr1.[tld] IE: uatsolr.mydomain.com ALIAS of uarsolr1.mydomain.com
* Change the Routing Policy to failover
* Failover Record Type Primary
* SetID: leave as suggested default
* Evaluate Health Target yes
* Create a CNAME record [version]solr.[tld] that is an Alias to [version]solr2.[tld]
* Change the Routing policy to failover, Record Type Secondary
* Repeat step 4 and 5 for this record.

Known Shortcomings
------------------

The templates at this point don't create an ELB for the Web Layer. This is intentional.
We use OpsWorks to provison a "master" web node, then snapshot this, create an AMI, and use the AMI in a launch config and ASG.
Our prime reason for doing this in Production environments in particular is because when environments scale, they usually need
to do so quite quickly due to load. 

When you dynamically provision hosts on the fly, there's always a moderately high risk a package has changed, repo is offline,
or any number of other things out of your control in complex builds aren't available when you need them to be. AMI's and Launch configs
absolutely eliminate this risk.

The component we're most likely to add to the mix is a script that automagically creates the AMI for you, creates a new launch
config, and optionally updates the ASG. Anyone who's run mission critical production environments will know, even though automagic
is a live saver, such things don't always work flawlessly in the real world. The ultimate answer would probably be something like
AWS Lambda that can use intelligence to carry out much of this work risk free. In the meanwhile, it is what it is.

The templates in their current form don't create the Route53 records you need for Solr. That's on the coming very soon list.

Known Issues
------------

**Critical**

There is an obscure issue we've made Amazon aware of regarding OpsWorks user management.
When users such as solr or ckan are created by OpsWorks recipes, they're automatically allocated an UID as you'd expect. Unfortunately
the UIDs allocated are in the same range OpsWorks uses for managing stack users permitting ssh access.
The upshot of all this, is when you add an OpsWorks stack user via the console, the AWS OpsWorks user recipe effectively hijacks the solr/ckan users
and turns them into OpsWorks stack users.

This is alarmingly bad and we've suggested to Amazon they perhaps limit this behavior to users within the UID range that also belong to
the opsworks group. You can identify when this bug has struck you because solr stops working, and ckan is deleted altogether.
The configure event in the OpsWorks stack reports an error - it's hijacking fails, but after all the damage is done.

You will also see a OpsWorks user with solr or ckan group membership.

We're working on a number of solutions to this prior to a Version 1 release.

The problem doesn't occur if you don't add any more ssh users to OpsWorks after the stack is provisioned. 

**Minor**

Zookeeper, GFS and SolrCloud aren't the easiest services to provision in an automated manner. At least not in highly dynamic stacks.
The recipes try as hard as they can to wait for DNS to resolve. The approach at the moment isn't bulletproof or perfect.
As a result, you may have GFS/ZooKeeper/SolrCloud node one come online, but node 2 fails it's setup step. 99% of the time if you repeat 
setup on node 2 a second time, everything will work like clock work.

**Minor**

The current recipes for GFS, Zookeeper and Solr partially cater for (n)Nodes. That said, it's assumed you'll only actually have two "service" nodes, 
and we know if you added a third node and expected this to just run flawlessly, chances are it won't.
Ultimately we'd like to be using AWS ElasticSearch and Elastic File Store if not S3 or a combination of those two. I'm certain both those
things will come. EFS is in limited release, from a regional perspective. The search in CKAN needs changes to support ElasticSearch.

Final Notes
-----------

The Datashades stack such as it is, is in active use by Link Digital and forms the basis on which Link Digital builds and supports
it's data platforms. As such it should be relatively well maintained.

We've done an initial release early more so people can see how to resolve particular issues when deploying CKAN. Especially in large scalable 
deployments.

I've personally seen a lot of "CKAN in a box" releases using Docker and other approaches. On the whole, these are fantastic for getting your
feet wet without drowning in the deep end. Datashades isn't an attempt to complete or replace any of these approachs. Rather, Datashades is
more of a high availability, "enterprise" stack built to best practise, with independantly scalable layers, easily adpated to CI work flows
and automated system maintenance.

Our hope and expectation is that it benefits the wider Public Data community and progresses the Open Data ideal.
 
