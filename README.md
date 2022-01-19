# Opencast Deployment with Docker
We will use the official Opencast image for this purpouse with the difference of the /opencast/etc/ folder

On the [official repo](https://github.com/opencast/opencast) version 11.x. the folder /etc/ recived some important changes opositive of the 10.x version. 
The changes that we need to keep on the 11.x version that provides of the 10.x version are:
- /opencast/etc/workflows
- /opencast/etc/encoding

The only workflow that we need is the API Workflow, so, we can create a new Workflow from the `schedule-and-upload.xml` of the 10.x version with the needed changes.
The new workflow is named: `moodle-publish.xml`

We cleaned all the stuff form the schedule-and-upload workflow and prepare for the API upload by default.

For that reason we provide you the opencast/etc/ folder.

## How to connect Opencast with Moodle using the API

We need to install some plugins on Moodle to allow Opencast the inter-communication. There are 5 plugins that we need to install on Moodle:

![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted%20image%2020220110093805%201.png?raw=true)

And the plugins are:
-   **[Tool](https://moodle.docs.opencast.org/#tool/about.html)**
-   **[Block](https://moodle.docs.opencast.org/#block/about.html)**
-   **[Chunk Upload](https://moodle.docs.opencast.org/#chunkupload/about.html)**
-   **[Repository](https://moodle.docs.opencast.org/#repository/about.html)**
-   **[Filter](https://moodle.docs.opencast.org/#filter/about.html)** 

### Tool Plugin
The Tool plugin is the most important plugin. Tool manage the API communication between Opencast and Moodle. So, we need to create the API user on Opencast with the following roles:
```
ROLE_API
ROLE_API_*
ROLE_SUDO
ROLE_GROUP_MH_DEFAULT_ORG_EXTERNAL_APPLICATIONS
```

This user is the user we need to configure on the Moodle plugin configuration: `Administration/Plugins/Admin tools/Opencast API`

### Block plugin
Next plugin is the Block plugin, that allow us to get a Moodle Block where teachers could upload the videos to Opencast Repository. The official docs are available [here]()
Once we add the the Opencast Block teachers are able to access and *create a new serie* and select videos to upload this serie. The series management help us to keep the peaceful order on Opencast.

**Important tip: at this moment the size limit on the upload process is the limit that sysadmins configure on the PHP.ini of Moodle. If we need (obviously we need id) to upload biggers files, we need to use the chunkupload plugin.** 

### Chunk Upload plugin
That plugin provid us a mechanism to split the file in N smallers pieces and upload by parts to the server. So, with that method, we avoid the size limitation of PHP.ini.

### Repository plugin
After has the videos uploads on Opencast and linkeds to the course using the API, we are ready to reference the videos on two different ways:
- Using the Video directly as one moodle common block. That appears on moodle course as OpenCast icon:
![[./img/Pasted image 20220119140814.png]]
![[Pasted image 20220119141009.png]]
- Using the page (or another content) block and insert the Opencast Video as another media object.
![[Pasted image 20220119141235.png]]

On this config options, it's important to select "Opencast channelid" = "api"

### Filter plugin
Is an extension of functions of Repository Plugin that needs the LTI integration

...(Comming soon)


At this moment we have an stable Opencast with Docker that could connect to Moodle throught the API.
So the next steps are:
- SSL config for HTTPS connections
- LTI integration
- Scale with K8s
