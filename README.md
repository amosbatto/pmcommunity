# pmcommunity
Code for the ProcessMaker community

I have created this code to help users of ProcessMaker when answering questions
at http://forum.processmaker.com

Most of this code was created in version 3.2 or 3.2.1. It may work in previous
versions of ProcessMaker, but I haven't tested it.

All code is Public Domain and can be used without any restrictions.

Author: Amos Batto
Email: amos@processmaker.com
ProcessMaker Forum Manager since 2009

## Contents

* **extraRest-*X*.*X*.tar**  Plugin to provide extra REST endpoints. 
   For more information, untar the plugin and examine the **extraRest/README.txt** file or see the source code
in **extraRest/src/Services/Api/ExtraRest/Extra.php**.

* **extraFunctions-*X*.*X*.tar** Plugin to provide extra PM functions to be used triggers and plugins. 
   For more information, untar the file and examine the **extraFunctions/README.txt** file or see the source code
in **extraFunctions/classes/class.pmFunctions.php**.

## Installing Plugins in ProcessMaker
To install these plugins, download the **.tar** file and login to ProcessMaker as
a user with the PM_SETUP permission in her role (such as the "admin" user). 
Then, go to **Admin > Plugins > Plugin Manager** and click on the **Import** button 
and select the **.tar** file to upload it to ProcessMaker. Once uploaded, then select 
the plugin in the list and click on **Enable**.
