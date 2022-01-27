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
#
## How to connect Opencast with Moodle using the API

We need to install some plugins on Moodle to allow Opencast the inter-communication. There are 5 plugins that we need to install on Moodle:

![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220110093805_1.png?raw=true)

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

All series and videos creates under one course will be available only on this course. Teacher can not access to another serie that exists in another course.****

**Important tip: at this moment the size limit on the upload process is the limit that sysadmins configure on the PHP.ini of Moodle. If we need (obviously we need id) to upload biggers files, we need to use the chunkupload plugin.** 

### Chunk Upload plugin
That plugin provid us a mechanism to split the file in N smallers pieces and upload by parts to the server. So, with that method, we avoid the size limitation of PHP.ini.

### Repository plugin
After has the videos uploads on Opencast and linkeds to the course using the API, we are ready to reference the videos on two different ways:
- Using the Video directly as one moodle common block. That appears on moodle course as OpenCast icon:

![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220119140814.png?raw=true)

![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220119141009.png?raw=true)

- Using the page (or another content) block and insert the Opencast Video as another media object.

![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220119141235.png?raw=true)

On this config options, it's important to select "Opencast channelid" = "api"

### Filter plugin
Is an extension of functions of Repository Plugin that needs the LTI integration

...(Coming soon)


#
## Configure HTTPS throught Nginx
First of all, we change the Docker-Compose structure
We create an internal and private network for the containers. That is the 172.16.0.0/24 network (For example, you can change it on the `docker-compose.yml`)

In addition, we delete the ports NAT of Opencast service because we don't need anymore to access directly to the Opencast service. We will use Nginx for that.

- `docker-compose.yml`
- `/opencast/etc/org.ops4j.pax.web.cfg`
  Change the default port HTTP: `80`
- `/opencast/etc/custom.properties`
  - `org.opencastproject.server.url=http://<your-domain>`
  - `org.opencastproject.download.url=http://<your-domain>/static`
- `/etc/nginx/templates/default.conf.template`
  - `server_name <your-domain>`


With this changes we are isolation the dockers connections out of the host network.

#
## Problems

### Videos don't show on Moodle
The URL still showing `http://localhost:8080` 
- I change the URIs from `localhost:8080` to `<your-domain>`
I add the Publish to Engage operation block but didn't work (I check again the publish to engage option on Moodle)

### Videos are not uploading to Engage
- "publish to engage" on Moodle -> Deactivate?


At this moment we have an stable Opencast with Docker that could connect to Moodle throught the API.


## [LTI Integration](https://docs.opencast.org/r/11.x/admin/#modules/ltimodule/)

At this moment, we have some troubles with the Opencast Blocks because it returns an 403 Forbiden error, so we are not able to access to the video from Moodle.
So, we are going to configure the LTI integration trying to solve the issue.

The Structure are the following:

Moodle will be the LTI consumer
Opencast will be the LTI producer

So, that means MoodleUsers try to access to Opencast resources.
- [https://subscription.packtpub.com/book/hardware_and_creative/9781783289714/9/ch09lvl1sec58/supporting-the-lti-consumers-and-producers](Test)

Go to Dashboard -> Site administration -> Plugins -> Activity modules -> External tool -> Manage tools
And create a new Tool:
![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0923.png?raw=true)
And use the Opencast Config parameters:
![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0924.png?raw=true)

Modify the following files:
- org.opencastproject.security.lti.LtiLaunchAuthenticationHandrler.cfg
  - Uncomment line: `lti.oauth.highly_trusted_consumer_key.1=CONSUMERKEY`
- mh_default_org.xml:
  - Uncomment line: `<ref bean="oauthProtectedResourceFilter" />` inside the _"authenticationFilters"_ bean 

### Use of LTI integration

From the Opencast Video Blocks on Moodle, teacher can upload videos to Opencast selecting a serie.
After that, teacher could add this video into diferent spaces at actual course. Adding a new activity:
![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126.png?raw=true)

We could choose:
- Video (Opencast):
  - This option embed a video into a new block of the course
  - Teacher could choose between a single video or a full video's series.
  ![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0904.png?raw=true)
  ![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0907.png?raw=true)

- Embed the video throught any other resource (for example, a new page):
  - Teacher needs to pick up the video from the OpenCast Repository on Moodle
  ![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0912.png?raw=true)
  ![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0913.png?raw=true)

  (If you are not using Firefox you will see the video on this page)
  ![alt text](https://github.com/mugenulnar/docker-opencast/blob/main/README/Pasted_image_20220126_0920.png?raw=true)

By adding the Opencast LTI integration once at your Moodle site, the LTI will be applied to all OpenCast content on your site.

## Future implementations
- ### Scale with K8s


## Adding Moodle to Docker-Compose
