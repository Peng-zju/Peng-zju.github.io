---
layout: post
tile: How to Update OpenEdx Xblock and Solve Error: render_django_template()  'i18n_service'
---

## How to Update OpenEdx Xblock and Solve Error: render_django_template()  'i18n_service'

When you install an Xblock A and this error occurs:
> Error: render_django_template() got an unexpected keyword argument 'i18n_service'

It means that Xblock A depends on another Xblock called [*xblock-utils*](https://github.com/edx/xblock-utils), and the version of *xblock-utils* is too low to support rendering UI using Django templates for Xblock A. 

To fix the this error, you need to update the xblock-utils Xblock in your server.

### 1. List the current version of xblock-util

   `$ /edx/bin/pip.edxapp list | grep xblock-utils`
 
### 2. Uninstall current version if version < 1.1.1
 
   `$ /edx/bin/pip.edxapp uninstall xblock-utils`
   
### 3. Install new version, from [Lawrence McDaniel Blog](https://blog.lawrencemcdaniel.com/how-to-install-an-xblock/)
 
   `$ cd ~`
   
   `$ git clone https://github.com/edx/xblock-utils.git`
	  
 #Change the folder ownership and group to edxapp
 
 `$ sudo chown -R edxapp xblock-in-video-quiz`
 
 `$ sudo chgrp -R edxapp xblock-in-video-quiz`
	  
#To install the xblock: 

`$ sudo -H -u edxapp bash`

`$ source /edx/app/edxapp/edxapp_env `

`$ /edx/bin/pip.edxapp install ~/xblock-utils`
 
### 4. Restart servers
-   LMS 

`$ sudo /edx/bin/supervisorctl restart lms`
-   CMS 

`$ sudo /edx/bin/supervisorctl restart cms`
    
-   Workers 

`$ sudo /edx/bin/supervisorctl restart edxapp_worker:`

If the above command will not work, try rebooting the server.

Now you can proceed with your routine for adding a Xblock in Studio by adding the desired the Xblock name in the Advanced Module List. And the error message should be gone.

### Reference
https://openedx.atlassian.net/wiki/spaces/COMM/pages/753532962/Native+XBlocks+Morning+Session

https://github.com/edx/xblock-utils/pull/49

