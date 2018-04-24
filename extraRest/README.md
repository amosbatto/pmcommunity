# Extra REST endpoints plugin 

The extraRest plugin includes extra REST endpoints to be used in ProcessMaker.
Some of these endpoints get around security restrictions in ProcessMaker's official
endpoints. Others provide functionality not provided by the official endpoints.

**Plugin:**    [extraRest-1.3.tar](extraRest-1.3.tar) (*right click* on link and select **Save Link As** in the context menu)  
**Author:**    Amos Batto (amos@processmaker.com)  
**Version:**   1.3 (2018-04-23) 
**Tested in:** ProcessMaker 3.2.1 Community in Debian 8.4 (probably will work in all 3.*X* versions)  
**License:**   Public Domain  

## Install this plugin in ProcessMaker
  * _Right click_ on the **extraRest-1.*X*.tar** file in Github and select "Save Link As" 
from the context menu to save the file to your local computer.   
  * Then, login to ProcessMaker as a user with the PM_SETUP permission in her role 
(such as the "admin" user).  
  * Then, go to **Admin > Plugins > Plugin Manager** and click on the **Import** button 
and select the **.tar** file to upload it to ProcessMaker. 
  * Once uploaded, then select the plugin in the list and click on **Enable**.

## More Information
For more information, untar the plugin and examine the source code in 
**extraRest/src/Services/Api/ExtraRest/extra.php**. 

## Included REST endpoints:
* [Claim case: `POST extrarest/case/{app_uid}/claim`](#claim-case-post-extrarestcaseapp_uidclaim)
* [Set case status: `PUT extrarest/case/status/{app_uid}`](#set-case-status-put-extrarestcasestatusapp_uid)
* [Get case info: `GET extrarest/case/{app_uid}`](#get-case-info-get-extrarestcaseapp_uid)
* [Get case and task info: `GET extrarest/case/{app_uid}/{del_index}`](#get-case-and-task-info-get-extrarestcaseapp_uiddel_index)
* [Get supervisor's list of review cases: `GET extrarest/cases/review`](#get-supervisors-list-of-review-cases-get-extrarestcasesreview)
* [Get logged-in user info: `GET extrarest/login-user`](#get-logged-in-user-info-get-extrarestlogin-user)
* [Get system language: `GET extrarest/language`](#get-system-language-get-extrarestlanguage)
* [Set system language: `PUT extrarest/language/{lang}`](#set-system-language-put-extrarestlanguagelang)
* [Get login session ID: `GET extrarest/session-id`](#get-login-session-id-get-extrarestsession-id)
* [Execute database query: `POST extrarest/sql`](#execute-database-query-post-extrarestsql)
* [Get user's default menu: `GET extrarest/user/{usr_uid}/config`](#get-users-default-menu-get-extrarestuserusr_uidconfig)
* [Set user's default menu: `PUT extrarest/user/{usr_uid}/config`](#set-users-default-menu-put-extrarestuserusr_uidconfig)
* [Get user's case list: `GET extrarest/cases/user/{user_uid}?{param=option}`](#get-users-case-list-get-extrarestcasesuseruser_uidparamoption)
* [Append rows to a PM Table: `PUT extrarest/pmtable/{pmt_uid}/append`](#append-rows-to-a-pm-table-put-extrarestpmtablepmt_uidappend)
* [Overwrite a PM Table: `PUT extrarest/pmtable/{pmt_uid}/overwrite`](#overwrite-a-pm-table-put-extrarestpmtablepmt_uidoverwrite)
* [Get document folders: `GET extrarest/documents/{fdr_uid}/folders`](#get-document-folders-get-extrarestdocumentsfdr_uidfolders)
* [Get documents in folder: `GET extrarest/documents/{fdr_uid}/contents`](#get-documents-in-folder-get-extrarestdocumentsfdr_uidcontents)

[Version Control](#version-control)

-------------------
### Claim case: `POST extrarest/case/{app_uid}/claim`

Claim a case for a user where a task is unassigned because the task 
is Self Service or Self Service Value Based Assignment. 

`POST http://{domain-or-ip}/api/1.0/{workspace}/extrarest/case/{app_uid}/claim`

**URL parameters:**  
  * `app_uid`:  Unique ID of case to claim.

**POST parameters:**  
  * `del_index`: *Optional.* The delegation index of the task to claim. Only needs 
             to be included if there are multiple open tasks in the case.
  * `usr_uid`:  *Optional.* Unique ID of the user to assign to case. Only include 
             if the logged-in user is a process supervisor assigning another user.
  
**Response:**  
HTTP status code is `200` and no response if successful.

**Example 1:**  
Assign the logged-in user to a Self Service Task where there is only one open task in case:
```php
$caseId = '2554682895ac25995666e24055342045';
$url = "/api/1.0/workflow/extrarest/case/$caseId/claim";
$aVars = array();
$oRet = pmRestRequest("POST", $url, $aVars, $oToken->access_token);
```

**Example 2:**  
Assign the logged-in user to a Self Service Task where there are 2 open tasks in case:
```php
$caseId = '2554682895ac25995666e24055342045';
$url = "/api/1.0/workflow/extrarest/case/$caseId/claim";
$aVars = array(
   'del_index' => 3
);
$oRet = pmRestRequest("POST", $url, $aVars, $oToken->access_token);
```

**Example 3:**  
Assign another user to Self Service Task when the logged-in user is a Process Supervisor:
```php
$caseId = '2554682895ac25995666e24055342045';
$url = "/api/1.0/workflow/extrarest/case/$caseId/claim";
$aVars = array(
  'del_index' => 2,  
  'usr_uid'   => '10654575559caec5e953104064429578' //unique ID of user to assign to task
);
$oRet = pmRestRequest("POST", $url, $aVars, $oToken->access_token); 
```

--------------------
### Set case status: `PUT extrarest/case/status/{app_uid}`

Change the case's status from TO_DO to DRAFT or from DRAFT to TO_DO. 
The logged-in user must either be the assigned user to the specified 
delegation index or a process supervisor for the case's process. 

`PUT http://{domain-or-ip}/api/1.0/{workspace}/extrarest/case/status/{app_uid}`

**URL parameters:**  
  * `app_uid`:  Unique ID of case to claim.

**POST parameters:**  
  * _string_ `status`: Case status to set, which can be `"TO_DO"` or `"DRAFT"`
  * _int_  `del_index`: Optional delegation index. If not set, then the current delegation in case.   
  
**Response:**  
HTTP status code is `200` and no response if successful.

**Example:**  
Change a case's status from DRAFT to TO_DO:
```php
$caseId = '7647950535a9f46ade01ab3092163527';
$url = "http://pm.example.com/api/1.0/workflow/extrarest/case/status/$caseId";
$aVars = array(
   'status' => 'TO_DO' 
);
$oRet = pmRestRequest("PUT", $url, $aVars, $oToken->access_token); 
```

--------------------
### Get case info: `GET extrarest/case/{app_uid}`

Get case information (without its task). Unlike the official endpoint 
GET cases/{app_uid}, this endpoint doesn't check whether the logged-in user had 
rights to access the case.  

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/case/{app_uid}`

**URL parameters:**  
  * `app_uid`:  Unique ID of case.

**Response:**  
If successful, the HTTP status code is set `200` and the response is a JSON
object with information about the case.

**Example:**  

*Request:*  
`http://pm.example.com/api/1.0/workflow/extrarest/case/72415796659f95bf89a59e2011357239`

*Response:*  
```javascript
{
    "APP_UID":             "72415796659f95bf89a59e2011357239",
    "APP_TITLE":           "#79",
    "APP_DESCRIPTION":     "",
    "APP_NUMBER":          79,
    "APP_PARENT":          "",
    "APP_STATUS":          "TO_DO",
    "APP_STATUS_ID":       2,
    "PRO_UID":             "73387376659f92ecf7a9382090189764",
    "APP_PROC_STATUS":     "",
    "APP_PROC_CODE":       "",
    "APP_PARALLEL":        "N",
    "APP_INIT_USER":       "00000000000000000000000000000001",
    "APP_CUR_USER":        "00000000000000000000000000000001",
    "APP_CREATE_DATE":     "2017-11-01 01:30:32",
    "APP_INIT_DATE":       "2017-11-01 01:30:32",
    "APP_FINISH_DATE":     null,
    "APP_UPDATE_DATE":     "2017-11-03 19:42:06",
    "APP_DATA": {
        "SYS_LANG":        "en",
        "SYS_SKIN":        "big",
        "SYS_SYS":         "workflow",
        "APPLICATION":     "72415796659f95bf89a59e2011357239",
        "PROCESS":         "73387376659f92ecf7a9382090189764",
        "TASK":            "32685606259f92ed00fc0d0080958682",
        "INDEX":           "2",
        "USER_LOGGED":     "00000000000000000000000000000001",
        "USR_USERNAME":    "admin",
        "accountNo":       "RS-321",
        "orderAmount":     342.50,
        "APP_NUMBER":      "79",
        "PIN":             "Z7OX",
        "__VAR_CHANGED__": "PROCESS,PROCESS,SYS_SYS,SYS_LANG,SYS_SKIN,htmlTable,htmlTable,htmlTable",
        "typeCurrentTask": "1"
    },
    "APP_PIN":             "63d21dfd49d86fb7650efbcd24ca4010",
    "APP_DURATION":        0,
    "APP_DELAY_DURATION":  0,
    "APP_DRIVE_FOLDER_UID":"",
    "APP_ROUTING_DATA":    "a:0:{}",
    "STATUS":              "To do",
    "TITLE":               "#79",
    "DESCRIPTION":         "",
    "CREATOR":             "Administrator admin",
    "CREATE_DATE":         "2017-11-01 01:30:32",
    "UPDATE_DATE":         "2017-11-03 19:42:06"
}
```

--------------------
### Get case and task info: `GET extrarest/case/{app_uid}/{del_index}`

Get case and task information. Unlike the official endpoint 
GET cases/{app_uid}/{del_index}, this endpoint doesn't check whether the logged-in user had 
rights to access the case.  

`GET http://{domain-or-ip}/api/1.0/workflow/extrarest/case/{app_uid}/{del_index}`

**URL parameters:**  
  * `app_uid`:  Unique ID of case.
  * `del_index`: Delegation index of a task in the case. If a task wasn't reassigned to another user,
  then the first task will usually have an index of 1, the second task will be 2, etc. 

**Response:**  
If successful, the HTTP status code is set `200` and the response is a JSON
object with information about the case and task.
  
**Response:**  
If successful, the HTTP status code is set to `200` and the response is an object
containing informtion about the case and its task.

**Example:**  
*Request:*  
```
http://pm.example.com/api/1.0/workflow/extrarest/case/72415796659f95bf89a59e2011357239/3
```
*Response:*  
```javascript
{
    "APP_UID":             "72415796659f95bf89a59e2011357239",
    "APP_TITLE":           "#79",
    "APP_DESCRIPTION":     "",
    "APP_NUMBER":          79,
    "APP_PARENT":          "",
    "APP_STATUS":          "TO_DO",
    "APP_STATUS_ID":       2,
    "PRO_UID":             "73387376659f92ecf7a9382090189764",
    "APP_PROC_STATUS":     "",
    "APP_PROC_CODE":       "",
    "APP_PARALLEL":        "N",
    "APP_INIT_USER":       "00000000000000000000000000000001",
    "APP_CUR_USER":        "00000000000000000000000000000001",
    "APP_CREATE_DATE":     "2017-11-01 01:30:32",
    "APP_INIT_DATE":       "2017-11-01 01:30:32",
    "APP_FINISH_DATE":     null,
    "APP_UPDATE_DATE":     "2017-11-03 19:42:06",
    "APP_DATA": {
        "SYS_LANG":        "en",
        "SYS_SKIN":        "big",
        "SYS_SYS":         "workflow",
        "APPLICATION":     "72415796659f95bf89a59e2011357239",
        "PROCESS":         "73387376659f92ecf7a9382090189764",
        "TASK":            "32685606259f92ed00fc0d0080958682",
        "INDEX":           "2",
        "USER_LOGGED":     "00000000000000000000000000000001",
        "USR_USERNAME":    "admin",
        "accountNo":       "RS-321",
        "orderAmount":     342.50,
        "APP_NUMBER":      "79",
        "PIN":             "Z7OX",
        "__VAR_CHANGED__": "PROCESS,PROCESS,SYS_SYS,SYS_LANG,SYS_SKIN,htmlTable,htmlTable,htmlTable",
        "typeCurrentTask": "1"
    },
    "APP_PIN":             "63d21dfd49d86fb7650efbcd24ca4010",
    "APP_DURATION":        0,
    "APP_DELAY_DURATION":  0,
    "APP_DRIVE_FOLDER_UID":"",
    "APP_ROUTING_DATA":    "a:0:{}",
    "STATUS":              "To do",
    "TITLE":               "#79",
    "DESCRIPTION":         "",
    "CREATOR":             "Administrator admin",
    "CREATE_DATE":         "2017-11-01 01:30:32",
    "UPDATE_DATE":         "2017-11-03 19:42:06"
    "TAS_UID":             "89054932459f92ed0438d57068674497",
    "DEL_INDEX":           1,
    "DEL_PREVIOUS":        0,
    "DEL_TYPE":            "NORMAL",
    "DEL_PRIORITY":        "3",
    "DEL_THREAD_STATUS":   "CLOSED",
    "DEL_THREAD":          1,
    "DEL_DELEGATE_DATE":   "2017-11-01 01:30:32",
    "DEL_INIT_DATE":       "2017-11-01 01:30:33",
    "DEL_TASK_DUE_DATE":   "2017-11-01 17:00:00",
    "DEL_FINISH_DATE":     "2017-11-01 01:31:17",
    "CURRENT_USER_UID":    "00000000000000000000000000000001",
    "TASK":                "89054932459f92ed0438d57068674497",
    "INDEX":               1,
    "PRO_ID":              29,
    "CURRENT_USER":        "Administrator admin"
}
```

--------------------
### Get supervisor's list of review cases: `GET extrarest/cases/review`

Returns the list of cases found under **Home > Review** for Process Supervisors.
The logged-in user must have the PM_SUPERVISOR permission in role to use this endpoint.

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/cases/review`
  
**Response:**  
if successful, the HTTP status code is set to `200` and the response is an 
array of objects holding information about cases, like the following:
```javascript
[
  {
    "APP_UID":               "2554682895ac25995666e24055342045",
    "DEL_INDEX":             "2",
    "DEL_LAST_INDEX":        "1",
    "APP_NUMBER":            "368",
    "APP_STATUS":            "TO_DO",
    "USR_UID":               "10654575559caec5e953104064429578",
    "PREVIOUS_USR_UID":      "00000000000000000000000000000001",
    "TAS_UID":               "9002452665ac24b3bc92243059239995",
    "PRO_UID":               "9399997805ac24afc9c3a52000021391",
    "DEL_DELEGATE_DATE":     "2018-04-02 12:26:14",
    "DEL_INIT_DATE":         "2018-04-02 12:27:29",
    "DEL_FINISH_DATE":       null,
    "DEL_TASK_DUE_DATE":     "2018-04-03 12:26:14",
    "DEL_RISK_DATE":         "2018-04-03 10:50:14",
    "DEL_THREAD_STATUS":     "OPEN",
    "APP_THREAD_STATUS":     "OPEN",
    "APP_TITLE":             "#368",
    "APP_PRO_TITLE":         "Customer complaint process",
    "APP_TAS_TITLE":         "Review complaint",
    "APP_CURRENT_USER":      "Batto Amos",
    "APP_DEL_PREVIOUS_USER": "admin Administrator",
    "DEL_PRIORITY":          "NORMAL",
    "DEL_DURATION":          "0",
    "DEL_QUEUE_DURATION":    "0",
    "DEL_DELAY_DURATION":    "0",
    "DEL_STARTED":           "0",
    "DEL_FINISHED":          "0",
    "DEL_DELAYED":           "0",
    "APP_CREATE_DATE":       "2018-04-02 12:25:57",
    "APP_FINISH_DATE":       null,
    "APP_UPDATE_DATE":       "2018-04-02 12:28:30",
    "APP_OVERDUE_PERCENTAGE":"0",
    "USR_FIRSTNAME":         "Amos",
    "USR_LASTNAME":          "Batto",
    "USR_USERNAME":          "amos",
    "APPDELCR_APP_TAS_TITLE":"Task 2",
    "USRCR_USR_UID":         "10654575559caec5e953104064429578",
    "USRCR_USR_FIRSTNAME":   "Amos",
    "USRCR_USR_LASTNAME":    "Batto",
    "USRCR_USR_USERNAME":    "amos",
    "PREVIOUS_USR_FIRSTNAME":"Administrator",
    "PREVIOUS_USR_LASTNAME": "admin",
    "PREVIOUS_USR_USERNAME": "admin",
    "APP_STATUS_LABEL":      "To do"
  },
  //the rest of the cases here
]
```
If the logged-in user does not have the PM_SUPERVISOR permissions in her role to 
access the **Home > Review** submenu, then the HTTP status code is set to `400` and the 
response is the following error object: 
```javascript
{
  "error": {
    "code":    400,
    "message": "Bad Request: Logged-in user lacks the PM_SUPERVISOR permission in role."
  }
}
```

--------------------
### Get logged-in user info: `GET extrarest/login-user`

Get information about the logged-in user.

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/login-user`
  
**Response:**  
If successful, the HTTP status code is set to `200` and the response an object
holding information about the logged-in user.

**Example:** 
*Request:*
```
http://pm.example.com/api/1.0/workflow/extrarest/login-user
```

*Response:*
```javascript
{
  "uid":                "10654575559caec5e953104064429578",
  "username":           "amos",
  "firstname":          "Amos",
  "lastname":           "Batto",
  "mail":               "amos@processmaker.com",
  "address":            "45 Elm St.",
  "zipcode":            "53827",
  "country":            "United States",
  "state":              "North Carolina",
  "location":           "Ellenboro",
  "phone":              "765-654-6523",
  "fax":                "",                    //deprecated
  "cellular":           "",                    //deprecated
  "birthday":           "2017-09-26",          //deprecated
  "position":           "Accounting",
  "role":               "PROCESSMAKER_OPERATOR",
  "replacedby":         "24141295159e5658e2e0fd9028990395",
  "replacedbyfullname": "Arturo Lopez",
  "duedate":            "2018-09-26",
  "calendar":           "00000000000000000000000000000001",
  "calendarname":       "Default Calendar",
  "status":             "ACTIVE",
  "department":         "13726536259e566c09db6f6075845729",
  "departmentname":     "Sales",
  "reportsto":          "92729925059e5652bebac55017512864",
  "userexperience":     "NORMAL",
  "photo":              "/opt/pm3.2.1/shared/sites/workflow/usersPhotographies/10654575559caec5e953104064429578.gif"
}
```

--------------------
### Get system language: `GET extrarest/language`

Get the system language of the logged-in user's session. This language is used 
by ProcessMaker when translating elements such as the case status. 

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/language`
  
**Response:**  
If successful, the HTTP status code is `200` and the response is the 
system language in "*xx*" or "*xx*-*CC*" format, such as `"es"` (Spanish) 
or `"pt-BR"` (Brazilian Portuguese). 

**Example:**  
*Request:*  
```
GET http://pm.example.com/api/1.0/workflow/extrarest/language
```
*Response:*  
```
200 (OK)
"pt-BR" 

```

--------------------
### Set system language: `PUT extrarest/language/{lang}`

Set the system language of the login session. This language will be used by 
subsequent REST calls. In an ordinary login in the graphical interface, 
the user can select the system language, but the oAuth2 login doesn't provide 
this option when using REST, so this endpoint can be used instead. 

`PUT http://{domain-or-ip}/api/1.0/{workspace}/extrarest/language/{lang}`

**URL parameters:**  
  * _string_ `lang`: The language in *xx* or *xx*-*CC* format, 
  such as `es` (Spanish) or `pt-BR` (Brazilian Portuguese). 

**POST parameters:**  
None. 
  
**Response:**  
HTTP status code is `200` and no response if successful.

**Example:**  
*Request:*  
```
PUT http://pm.example.com/api/1.0/workflow/extrarest/language/fr
```
*Response:*  
```
200
```


--------------------
### Get login session ID: `GET extrarest/session-id`

Get a login session ID that can be attached to URLs to access pages and files inside ProcessMaker 

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/session-id`
  
**Response:**  
If successful, the HTTP status code is set to `200` and the response is a session ID, which is a string consisting of 32 hexidecimal characters.

**Note:** The session ID can be attached to the end of URLs in ProcessMaker to access those
pages with a login session:  
`http://{domain-or-ip}/sys{workspace}/{lang}/{skin}/{folder}/{method}.php?sid={session-id}`  

Example opening a case:  
`http://example.com/sysworkflow/en/neoclassic/cases/cases_Open?APP_UID=9777918985a8dddbf884236088281261&DEL_INDEX=3&action=todo&sid=1234567890abcde1234567890abcde`  

Example accessing case's process map:  
`http://example.com/sysworkflow/en/neoclassic/cases/designer?prj_uid=6135378175a8dcdca9a2641037096319&prj_readonly=true&app_uid=9777918985a8dddbf884236088281261&sid=1234567890abcde1234567890abcde`  

Example accessing an Input Document file:  
`http://example.com/sysworkflow/en/neoclassic/cases/cases_ShowDocument?a=4699401854d8262f569e9a1070221206&sid=1234567890abcde1234567890abcde`  

**Example:**  
*Request:*  
```
GET http://pm.example.com/api/1.0/workflow/extrarest/session-id
```
*Response:*  
```javascript
200 (OK)
"2554682895ac25995666e24055342045"
```

--------------------
### Execute database query: `POST extrarest/sql`

Execute an SQL SELECT query in the current workspace's workflow database, 
which by default is named "wf_workflow". 

`POST http://{domain-or-ip}/api/1.0/{workspace}/extrarest/sql`

**POST parameters:**  
  * _string_ `sql`: SQL SELECT statement to execute.

**Note 1:** For security reasons, the code for this endpoint is commented out, but it can
be enabled if removing the `/* ... */` around the code. If needing to executing SQL 
queries, it is recommended to adapt the code for your specific purpose and 
include the specific query in the code of the endpoint and only pass the
parameters that need to be changed to the endpoint. If this endpoint is allowed to 
execute any SQL query, a hacker could use it to steal all the information from the
ProcessMaker database. 

**Note 2:** If thinking of modifying this endpoint to allow UPDATE, INSERT and DELETE
statements, then make sure to change the ProcessMaker configuration files. See:
http://wiki.processmaker.com/3.0/Consulting_the_ProcessMaker_databases#Protecting_PM_Core_Tables
 
**Response:**  
If successful, the HTTP status code is set to `200` and the response contains
an array of objects which hold each record retrieved from the database,  
such as in the following example:
```javascript
[
  {
    "USER":        "Wilson Jane",
    "NUM_OVERDUE": "8"
  },
  {
    "USER":        "Batto Amos",
    "NUM_OVERDUE": "6"
  },
  {
    "USER":        "Gomez Freddy",
    "NUM_OVERDUE": "1"
  }
]
```

**PHP example:**  
The database is queried to find the number of open cases assigned to each user 
since the beginning of the year 2018, which are overdue:
```php
$url = "/api/1.0/workflow/extrarest/sql";
$sql = "SELECT APP_CURRENT_USER AS USER, COUNT(*) AS NUMBER_OVERDUE 
   FROM APP_CACHE_VIEW WHERE DATE(DEL_INIT_DATE) >= '2018-01-01' AND 
   DEL_FINISH_DATE IS NULL AND DEL_TASK_DUE_DATE < NOW() 
   GROUP BY USR_UID ORDER BY APP_CURRENT_USER";
$oRet = pmRestRequest("POST", $url, array("sql"=>$sql), $oToken->access_token);

if ($oRet->status == 200) {
   foreach ($oRet->response as $aRow) {
      print "User ".$aRow->USER." has ".$aRow->NUMBER_OVERDUE.
         " cases since the start of the year 2018.\n";
   }
}
```
This script will produce the following output:
```
User Wilson Jane has 8 cases since the start of the year 2018.
User Batto Amos has 6 cases since the start of the year 2018.
User Gomez Freddy has 1 cases since the start of the year 2018.
```
This example is only provided to show what is possible with a REST endpoint. 
If needing to use this endpoint in production, remember to modify the source 
code of this endpoint to only execute the specific SQL query that you need, 
and not use it to execute any SQL query as shown in this example. Otherwise,
you are providing a way for hackers to attack your installation of ProcessMaker.

For example, instead of using the above endpoint, it is recommended to edit the
source code of `workflow/engine/plugins/extraRest/src/Services/Api/ExtraRest/extra.php`
and create the specific endpoint that is needed with its SQL query like this:
```php
    /**
     * Get the number of open cases assigned to each user which are overdue in the workspace.
     * The logged-in user needs the PM_ALLCASES permission in his/her role.
     * 
     * @url GET /cases/number-overdue-by-user
     * @access protected
     *
     * @param string $date_from Optional. Task assigned after date in 'YYYY-MM-DD' format  {@from query}
     * @param string $date_to   Optional. Task assigned before date in 'YYYY-MM-DD' format {@from query}
     *   
     * @return array
     * 
     * @author Amos Batto <amos@processmaker.com>
     * @copyright Public Domain
     */
    public function getNumberCasesOverdueByUser($date_from=null, $date_to=null) {
        try {
            if ($this->userCanAccess('PM_ALLCASES') == 0) {
                throw new \Exception("Logged-in user lacks the PM_ALLCASES permission in role.");
            }
            
            $sqlLimitDate = '';
            
            if (!empty($date_from)) {
               if (!preg_match('/^\d{4}-[0-2]\d-[0-3]\d$/', $date_from)) {
                  throw new \Exception("Bad date in date_from. Use format: date_from=YYYY-MM-DD");
               }
               
               $sqlLimitDate = "DATE(DEL_INIT_DATE) >= '$date_from' AND ";
            }
            
            if (!empty($date_to)) {
               if (!preg_match('/^\d{4}-[0-2]\d-[0-3]\d$/', $date_to)) {
                  throw new \Exception("Bad date in date_to. Use format: date_to=YYYY-MM-DD");
               }
               
               $sqlLimitDate .= "DATE(DEL_INIT_DATE) <= '$date_to' AND ";
            }
               
            $g = new \G();
            $g->loadClass("pmFunctions");
            
            $sql = "SELECT USR_UID AS USER_ID, 
                APP_CURRENT_USER AS USER, 
                COUNT(*) AS NUMBER_OVERDUE 
                FROM APP_CACHE_VIEW 
                WHERE $sqlLimitDate 
                DEL_FINISH_DATE IS NULL AND DEL_TASK_DUE_DATE < NOW() 
                GROUP BY USR_UID ORDER BY APP_CURRENT_USER";
        
            $aResult = executeQuery($sql);
            
            $aRows = array();
            foreach ($aResult as $aRow) {
                $aRows[] = $aRow;
            }
            return $aRows;
        } 
        catch (\Exception $e) {
            throw new RestException(Api::STAT_APP_EXCEPTION, $e->getMessage());
        }  
   }
```
Notice how this endpoint allows optional dates to be specified to limit the 
scope of the SQL query, but it uses 
[preg_match()](http://php.net/manual/en/function.preg-match.php) to 
check that the dates have a specific format to prevent SQL injection attacks. 
It is a good idea to check that the input to an endpoint matches a 
specific pattern with `preg_match()` or to pass the input through 
[mysqli_real_escape_string()](http://php.net/manual/en/mysqli.real-escape-string.php)
sanitize it.  

After editing the plugin's source code to add a new endpoint, login to
ProcessMaker as a user such as "admin" with the PM_SETUP_ADVANCED permission 
in her role. Go to **Admin > Settings > Plugins > Plugins Manager** and disable
the **extraRest** plugin. Then, reenable it so it will recreate the list
of available endpoints stored in the `shared/sites/{workspace}/routes.php` file.

Then, the custom endpoint can be executed, as shown in this example in PHP:
```php
$url = "/api/1.0/workflow/extrarest/cases/number-overdue-by-user?date_from=2018-01-01";
$oRet = pmRestRequest("GET", $url, null, $oToken->access_token);

if ($oRet->status == 200) {
   foreach ($oRet->response as $aRow) {
      print "User ".$aRow->USER." has ".$aRow->NUMBER_OVERDUE.
         " cases since the start of the year 2018.\n";
   }
}
``` 

--------------------
### Get user's default menu: `GET extrarest/user/{usr_uid}/config`

Get a user's default menu, which is stored in a serialized array in the 
`CONFIGURATION.CFG_VALUE` field in the database. This endpoint can
only be called by users who have the PM_USERS permission in their role.

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/user/{usr_uid}/config`

**URL parameters:**  
  * _string_ `usr_uid`:  Unique ID of user.
  
**Response:**  
If successful, the HTTP status code is set to `200` and the response contains an
object holding the user's configuration, such as:
```javascript
{
   "DEFAULT_LANG":       "",          
   "DEFAULT_MENU":       "PM_CASES",
   "DEFAULT_CASES_MENU": "CASES_SENT"
}
```
Note that the `DEFAULT_LANG` is deprecated and no longer used by ProcessMaker.

**PHP Example:**  
```php
$url = "/api/1.0/workflow/extrarest/user/10654575559caec5e953104064429578/config";
$oRet = pmRestRequest("GET", $url, null, $oToken->access_token);
if ($oRet->status == 200) {
   $userMenu = $oRet->response['DEFAULT_MENU'];
}
```

--------------------
### Set user's default menu: `PUT extrarest/user/{usr_uid}/config`

Set a user's default menu, which is stored in a serialized array in the 
`CONFIGURATION.CFG_VALUE` field in the database. This endpoint can
only be called by users who have the PM_USERS permission in their role.

`POST http://{domain-or-ip}/api/1.0/{workspace}/extrarest/user/{usr_uid}/config`

**URL parameters:**  
  * _string_ `usr_uid`:  Unique ID of user.

**POST parameters:**  
* _string_ `default_lang`: *Optional.* Default interface language in "*xx*" or "*xx*-*CC*" format, 
such as `"es"` (Spanish) or `"pt-BR"` (Brazilian Portuguese). 
Note that this parameter is deprecated and no longer used by ProcessMaker. 
* _string_ `default_menu`: *Optional.* Default main menu to set for the user: 
    * `""`: Default for role, 
    * `"PM_CASES"`: Home, 
    * `"PM_FACTORY"`: Designer, 
    * `"PM_SETUP"`: Admin,
    * `"PM_DASHBOARD"`: Dashboard,
    * or any custom menu defined by a plugin. 
* _string_ `default_cases_menu`: *Optional.* The default submenu selected in the cases sidebar under **Home**. 
Only used if the `default_menu` is set to `"PM_CASES"`. Available options:
    * `""`: The default which is the **Inbox**,
    * `"CASES_START_CASE"`: New case,
    * `"CASES_INBOX"`: Inbox,
    * `"CASES_DRAFT"`: Draft,
    * `"CASES_PAUSED"`: Paused, 
    * `"CASES_SENT"`: Participated, 
    * `"CASES_SELFSERVICE"`: Unassigned, 
    * `"CASES_SEARCH"`: Advanced search (need PM_ALLCASES in role), 
    * `"CASES_TO_REVISE"`: Review (need PM_SUPERVISOR in role),
    * `"CASES_TO_REASSIGN"`: Reassign (need PM_SUPERVISOR and PM_REASSIGNCASE in role),
    * `"CASES_FOLDERS"`: Documents (need PM_FOLDERS_VIEW in role),
    * or any custom submenu defined by a plugin.

**Response:**  
If successful, the HTTP status code is set to `200` and the response is an object 
holding the updated user configuration.

**PHP example:**   
```php
$userId = '92729925059e5652bebac55017512865';
$defaultMenu = 'PM_CASES';
$defaultCasesMenu = 'CASES_SENT';
$url = "/api/1.0/workflow/extrarest/user/$userId/config";
    
$aVars = array(
  'default_lang'       => 'el',
  'default_menu'       => $defaultMenu,
  'default_cases_menu' => $defaultCasesMenu
);
$oRet = pmRestRequest('PUT', $url, $aVars, $oToken->access_token);
```
---------------------
### Get user's case list: `GET extrarest/cases/user/{user_uid}?{param=option}`

Return the list of cases for a specified user. Extra query string parameters
can be added to get a customized list of cases. If retrieving the case list 
for a user other than the logged-in user, then the logged-in user must have
the PM_ALLCASES permission in her role. 

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/user/{usr_uid}/config`

**URL parameters:**  
  * _string_ `usr_uid`:  Unique ID of user for whom to retrieve the case list. 
  Set to `00000000000000000000000000000000` to retrieve a case list of the logged-in user.
  
**Optional query string parameters:**  
  * `start={integer}`: The number where to begin listing cases.  
  * `limit={integer}`: The maximum number of cases to list.
  * `action={option}`: Specify the type of cases, which can be:  
  `todo`, `draft`, `sent` (participated), `unassigned` (self service), `paused`, 
  `completed`, `cancelled`, `search` (Home > Advanced Search for users with PM_ALLCASES in role),
  `simple_search` (normal search), `to_revise` (Home > Review for process supervisors),
  `to_reassign` (Home > Reassign for process supervisors with PM_REASSIGNCASE permission),
  `all`, `gral` (general), `default` (like `todo`)
  * `filter={option}`: An additional filter for the type of case, which can be:  
  `read`, `unread`, `started`, `completed`
  * `search={string}`: Case-insensitive string to search for in the case number, case title, process title, 
  task title, assigned user's name and any additional fields if using the 
  [Custom Case List Builder](http://wiki.processmaker.com/3.0/Cases_List_Builder) plugin.
  * `pro_uid={uid}`: Retrieve cases from a particular process, specified by its unique ID. 
  * `app_status={status}`: If `action=search`, then this parameter is used to specify the 
  case's status which can be:  
  `TO_DO`, `DRAFT`, `PAUSED`, `CANCELLED`, `COMPLETED`, `ALL`
  * `date_from={YYYY-MM-DD}`: Retrieve cases whose task started or ended after or on the specified date.
  * `date_to={YYYY-MM-DD}`: Retrieve cases whose task started or ended before or on the specified date.
  * `dir={order}`: Set to `ASC` to sort cases in ascending order or `DESC` in descending order.
  * `sort={field}`: Database field in the APP_CACHE_VIEW table by which to sort cases.
  * `cat_uid={uid}`: Retrieve cases whose process is in a category, specified by its unique ID 
  (found in the PROCESS_CATEGORY.CATEGORY_UID field in the database).
  * `configuration={integer}`: Set to `1` to use the configuration or `0` to 
  not use it when searching for cases. 
  * `paged={integer}`: Set to `1` to return a page of cases or `0` to not return the cases in a page. 
  It is recommended to uses pages when dealing with a large numbers of cases.
  * `newer_than={YYYY-MM-DD}`: Retrieve cases whose task started or ended after the specified date.
  This parameter is like `date_from`, but it is > rather than >= the date.
  * `older_than={YYYY-MM-DD}`: Retrieve cases whose task started or ended before the specified date. 
  This parameter is like `date_to`, but it is < rather than <= the date.

**Response:**  
If successful, the HTTP status code is set to `200` and the response is an object with the `totalCount` member which lists
the total number of cases which match the search criteria and the `data` member which is an array of objects holding 
information about each case.

**Example:**    
Get a list of participated cases for the logged-in user since the start of the year 2018.  
*Request:*
```
GET http://pm.example.com/api/1.0/workflow/extrarest/cases/user/00000000000000000000000000000000?action=sent&date_from=2018-01-01
```
*Response:*  
```javascript
200 (OK)
{
  "totalCount": 2,
  "data": [
    {
      "APP_UID":               "5295973255ac51bf10472f6060420954",
      "DEL_INDEX":             "1",
      "DEL_LAST_INDEX":        "1",
      "APP_NUMBER":            "374",
      "APP_STATUS":            "DRAFT",
      "USR_UID":               "00000000000000000000000000000001",
      "PREVIOUS_USR_UID":      "",
      "TAS_UID":               "1029336105ac5121d1624d5021075541",
      "PRO_UID":               "5348454895ac4096b9c3544003146878",
      "DEL_DELEGATE_DATE":     "2018-04-04 14:39:45",
      "DEL_INIT_DATE":         "2018-04-04 14:39:45",
      "DEL_FINISH_DATE":       null,
      "DEL_TASK_DUE_DATE":     "2018-04-05 14:39:45",
      "DEL_RISK_DATE":         "2018-04-05 13:03:45",
      "DEL_THREAD_STATUS":     "OPEN",
      "APP_THREAD_STATUS":     "OPEN",
      "APP_TITLE":             "#374",
      "APP_PRO_TITLE":         "New Product Testing",
      "APP_TAS_TITLE":         "Document New Product",
      "APP_CURRENT_USER":      "admin Administrator",
      "APP_DEL_PREVIOUS_USER": "",
      "DEL_PRIORITY":          "NORMAL",
      "DEL_DURATION":          "0",
      "DEL_QUEUE_DURATION":    "0",
      "DEL_DELAY_DURATION":    "0",
      "DEL_STARTED":           "0",
      "DEL_FINISHED":          "0",
      "DEL_DELAYED":           "0",
      "APP_CREATE_DATE":       "2018-04-04 14:39:45",
      "APP_FINISH_DATE":       null,
      "APP_UPDATE_DATE":       "2018-04-04 16:25:59",
      "APP_OVERDUE_PERCENTAGE":"0",
      "USR_FIRSTNAME":         "Administrator",
      "USR_LASTNAME":          "admin",
      "USR_USERNAME":          "admin",
      "APPDELCR_APP_TAS_TITLE":"Task 1",
      "USRCR_USR_UID":         "00000000000000000000000000000001",
      "USRCR_USR_FIRSTNAME":   "Administrator",
      "USRCR_USR_LASTNAME":    "admin",
      "USRCR_USR_USERNAME":    "admin",
      "APP_STATUS_LABEL":      "Draft"
    },
    {
      "APP_UID":               "3633901045ac51b2bac1c58087855401",
      "DEL_INDEX":             "1",
      "DEL_LAST_INDEX":        "1",
      "APP_NUMBER":            "373",
      "APP_STATUS":            "DRAFT",
      "USR_UID":               "00000000000000000000000000000001",
      "PREVIOUS_USR_UID":      "",
      "TAS_UID":               "1029336105ac5121d1624d5021075541",
      "PRO_UID":               "5348454895ac4096b9c3544003146878",
      "DEL_DELEGATE_DATE":     "2018-04-04 14:36:28",
      "DEL_INIT_DATE":         "2018-04-04 14:36:28",
      "DEL_FINISH_DATE":       null,
      "DEL_TASK_DUE_DATE":     "2018-04-05 14:36:28",
      "DEL_RISK_DATE":         "2018-04-05 13:00:28",
      "DEL_THREAD_STATUS":     "OPEN",
      "APP_THREAD_STATUS":     "OPEN",
      "APP_TITLE":             "#373",
      "APP_PRO_TITLE":         "Customer Complaint Process",
      "APP_TAS_TITLE":         "Review Complaint",
      "APP_CURRENT_USER":      "admin Administrator",
      "APP_DEL_PREVIOUS_USER": "",
      "DEL_PRIORITY":          "NORMAL",
      "DEL_DURATION":          "0",
      "DEL_QUEUE_DURATION":    "0",
      "DEL_DELAY_DURATION":    "0",
      "DEL_STARTED":           "0",
      "DEL_FINISHED":          "0",
      "DEL_DELAYED":           "0",
      "APP_CREATE_DATE":       "2018-04-04 14:36:27",
      "APP_FINISH_DATE":       null,
      "APP_UPDATE_DATE":       "2018-04-04 14:36:27",
      "APP_OVERDUE_PERCENTAGE":"0",
      "USR_FIRSTNAME":         "Administrator",
      "USR_LASTNAME":          "admin",
      "USR_USERNAME":          "admin",
      "APPDELCR_APP_TAS_TITLE":"Task 1",
      "USRCR_USR_UID":         "00000000000000000000000000000001",
      "USRCR_USR_FIRSTNAME":   "Administrator",
      "USRCR_USR_LASTNAME":    "admin",
      "USRCR_USR_USERNAME":    "admin",
      "APP_STATUS_LABEL":      "Draft"
    }
  ]
}
```
---------------------
### Append rows to a PM Table: `PUT extrarest/pmtable/{pmt_uid}/append`
*Available in version 1.2 and later.*

Append records to the end of a PM Table. The logged-in user must have the 
[PM_SETUP](http://wiki.processmaker.com/3.1/Roles#PM_SETUP) and 
[PM_SETUP_PM_TABLES](http://wiki.processmaker.com/3.1/Roles#PM_SETUP_PM_TABLES) 
permissions in his/her in role to use this endpoint. 

`PUT http://{domain-or-ip}/api/1.0/{workspace}/extrarest/pmtable/{pmt_uid}/append`


**URL parameters:**  
  * _string_ `pmt_uid`: The unique ID of the PM Table (which can be found 
  with [GET /pmtable](http://wiki.processmaker.com/3.0/REST_API_Administration#PM_Tables_List:_GET_.2Fpmtable)
  or by querying the `ADDITIONAL_TABLES.ADD_TAB_UID` field in the database).
  
**POST parameters:**  
  * _array_ `rows`: An array of objects, where each object represents
     a row to add to the table and its properties are the field names:  
```
[
  {
    "FIELD1": "VALUE1",
    "FIELD2": "VALUE2",
    ...
  },
  ...
]  
```

**Note:**  
This endpoint does *not* check whether the fields in the PM Table are not
allowed to be NULL (left blank) nor does it check whether they are primary keys, so their
values should be unique.

**Response:**  
If the new rows were added, then the HTTP status code will set to `200` and the
response will be the number of inserted rows. 

If the name of a field is misspelled or not included, the record will be inserted and the 
field will set to NULL (left blank). Fields which are auto-increment do not need to be included.

If *none* of the names of the fields passed to this endpoint match any of 
the names of the PM Table's fields, then the HTTP
status code will be `400` and the following response will be returned:
```javascript
{
   "error": {
      "code":    400,
      "message": "Bad Request: The value of key column is required"
   }
}
```

**Example:**  
*Request:*  
`PUT http://example.com/api/1.0/workflow/extrarest/pmtable/2714474795ad61591723eb7014657886/append`  
```javascript
Content-Type: application/json
{
  "rows": [
    {"CLIENT_NAME": "Jane Doe", "CONTRACT_DATE": "2018-06-20", "AMOUNT": 4500.00 },
    {"CLIENT_NAME": "Bill Row", "CONTRACT_DATE": "2017-12-31", "AMOUNT": 999.99  },
    {"CLIENT_NAME": "Sam Slow", "CONTRACT_DATE": "2018-01-01", "AMOUNT": 27573.50}
  ]
}
```
*Response:*  
```javascript
200 (OK)
3
```
**PHP Example:** 
```php 
$tableId = '2714474795ad61591723eb7014657886'; //ID for PMT_CONTRACTS
$aVars = array(
   'rows' => array(
      array('CLIENT_NAME'=>'Jane May', 'CONTRACT_DATE'=>'2018-06-20', 'AMOUNT'=>4500.00),
      array('CLIENT_NAME'=>'Bill Row', 'CONTRACT_DATE'=>'2017-12-31', 'AMOUNT'=>2699.99),
      array('CLIENT_NAME'=>'Sam Slow', 'CONTRACT_DATE'=>'2018-01-01', 'AMOUNT'=>78.99)
   )
);

$url = "/api/1.0/workflow/extrarest/pmtable/$tableId/append";
$oRet = pmRestRequest("PUT", $url, $aVars, $oToken->access_token);

if ($oRet->status == 200) {
   echo $oRet->response . " records were inserted.";
}   
```

---------------------
### Overwrite a PM Table: `PUT extrarest/pmtable/{pmt_uid}/overwrite`
*Available in version 1.2 and later.*

Remove *all* the existing records in a PM Table and then refill the table with
new records. The logged-in user must have the 
[PM_SETUP](http://wiki.processmaker.com/3.1/Roles#PM_SETUP) and 
[PM_SETUP_PM_TABLES](http://wiki.processmaker.com/3.1/Roles#PM_SETUP_PM_TABLES) 
permissions in his/her in role to use this endpoint. 

`PUT http://{domain-or-ip}/api/1.0/{workspace}/extrarest/pmtable/{pmt_uid}/overwrite`


**URL parameters:**  
  * _string_ `pmt_uid`: The unique ID of the PM Table (which can be found 
  with [GET /pmtable](http://wiki.processmaker.com/3.0/REST_API_Administration#PM_Tables_List:_GET_.2Fpmtable)
  or by querying the `ADDITIONAL_TABLES.ADD_TAB_UID` field in the database).
  
**POST parameters:**  
  * _array_ `rows`: An array of objects, where each object represents
     a row to add to the table and its properties are the field names:  
```
[
  {
    "FIELD1": "VALUE1",
    "FIELD2": "VALUE2",
    ...
  },
  ...
]  
```

**Note:**  
This endpoint does *not* check whether the fields in the PM Table are not
allowed to be NULL (left blank) nor does it check whether they are primary keys, so their
values should be unique.

**Response:**  
If the new rows were added, then the HTTP status code will set to `200` and the
response will be the number of inserted rows. 

If the name of a field is misspelled or not included, the record will be inserted and the 
field will set to NULL (left blank). Fields which are auto-increment do not need to be included.

If *none* of the names of the fields passed to this endpoint match any of 
the names of the PM Table's fields, then the HTTP
status code will be `400` and the following response will be returned:
```javascript
{
   "error": {
      "code":    400,
      "message": "Bad Request: The value of key column is required"
   }
}
```

**Example:**  
*Request:*  
`PUT http://example.com/api/1.0/workflow/extrarest/pmtable/2714474795ad61591723eb7014657886/overwrite`  
```javascript
Content-Type: application/json
{
  "rows": [
    {"CLIENT_NAME": "Jane Doe", "CONTRACT_DATE": "2018-06-20", "AMOUNT": 4500.00 },
    {"CLIENT_NAME": "Bill Row", "CONTRACT_DATE": "2017-12-31", "AMOUNT": 999.99  },
    {"CLIENT_NAME": "Sam Slow", "CONTRACT_DATE": "2018-01-01", "AMOUNT": 27573.50}
  ]
}
```
*Response:*  
```javascript
200 (OK)
3
```
**PHP Example:** 
```php 
$tableId = '2714474795ad61591723eb7014657886'; //ID for PMT_CONTRACTS
$aVars = array(
   'rows' => array(
      array('CLIENT_NAME'=>'Jane May', 'CONTRACT_DATE'=>'2018-06-20', 'AMOUNT'=>4500.00),
      array('CLIENT_NAME'=>'Bill Row', 'CONTRACT_DATE'=>'2017-12-31', 'AMOUNT'=>2699.99),
      array('CLIENT_NAME'=>'Sam Slow', 'CONTRACT_DATE'=>'2018-01-01', 'AMOUNT'=>78.99)
   )
);

$url = "/api/1.0/workflow/extrarest/pmtable/$tableId/overwrite";
$oRet = pmRestRequest("PUT", $url, $aVars, $oToken->access_token);

if ($oRet->status == 200) {
   echo $oRet->response . " records were inserted.";
}   
```

---------------------
### Get document folders: `GET extrarest/documents/{fdr_uid}/folders`
*Available in version 1.3 and later.*

Retrieve a list of subfolders found in a specified parent folder. These are the 
folders displayed by going to **Home > Documents** in the ProcessMaker interface. 
The logged-in user must have the [PM_FOLDERS_VIEW](http://wiki.processmaker.com/3.1/Roles#PM_FOLDERS_VIEW) 
permission in his/her role.

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/documents/{fdr_uid}/folders`

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/documents/{fdr_uid}/folders?limit={int}&start={int}&direction={dir}&sort={field}&search={str}`

**URL parameters:**  
  * _string_ `fdr_uid`: The unique ID of the folder or `root` if the base folder "/".
  
**Optional query string parameters:**  
  * `limit={int}`: The maximum number of folders to return. If set to `0`, 
    which is the default, then an unlimited number of folders will be returned.  
  * `start={int}`: The number where to start listing folders. If set to `0`, 
    which is the default, then will start from the first folder. This parameter
    is often used in conjunction with `limit={int}` when only displaying a
    fixed number of folders at a time.
  * `direction={dir}`: The sort direction which can be `ASC` (ascending which is 
    the default) or `DESC` (descending).
  * `sort={field}`: The field used to sort the folders, which can be `appDocCreateDate`
    (the date the folder was first created), `name` (the folder name) or "``" (no sort order).
  * `search={str}`: A case insensitive string to search for in the folder names. 
    Make sure to use a function such as PHP's url_encode() or JavaScript's 
    encodeURIComponent() so `" "` (space) becomes `%20` and `"é"` becomes `%C3%A9`.
    
**Response:**  
* An object holding the following two properties:
    * `folders`: An array of folder objects. Each folder object has the following properties:
        * `FOLDER_UID`:         The unique ID of the folder.
        * `FOLDER_PARENT_UID`:  The unique ID of the parent folder or `"\/"` if the parent is the root folder.
        * `FOLDER_NAME`:        The folder's name.
        * `FOLDER_CREATE_DATE`: The datetime when the folder was created in `"YYYY-MM-DD HH:MM:SS"` format.
        * `FOLDER_UPDATE_DATE`: The datetime when the folder was last updated in `"YYYY-MM-DD HH:MM:SS"` format.
    * `totalFoldersCount`:      The total number of subfolders.

**Example:**  
*Request:*  
`GET http://example.com/api/1.0/workflow/extrarest/documents/root/folders`

*Response:*
```javascript
{
   "folders": [
      {
         "FOLDER_UID":         "6268552735ade43f898a294019015502",
         "FOLDER_PARENT_UID":  "\/",
         "FOLDER_NAME":        "invoices",
         "FOLDER_CREATE_DATE": "2018-04-23 16:37:12",
         "FOLDER_UPDATE_DATE": "2018-04-23 16:37:12"
      },
      {
         "FOLDER_UID":         "5250317125ade44071e56e2014209371",
         "FOLDER_PARENT_UID":  "\/",
         "FOLDER_NAME":        "reports",
         "FOLDER_CREATE_DATE": "2018-04-23 16:37:27",
         "FOLDER_UPDATE_DATE": "2018-04-23 16:37:27"
      }
   ],
   "totalFoldersCount":        2
}
```

---------------------
### Get documents in folder: `GET extrarest/documents/{fdr_uid}/contents`
*Available in version 1.3 and later.*

Retrieve the documents in a specified folder. 
The logged-in user must have the [PM_FOLDERS_VIEW](http://wiki.processmaker.com/3.1/Roles#PM_FOLDERS_VIEW) 
permission in his/her role.

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/documents/{fdr_uid}/contents`

`GET http://{domain-or-ip}/api/1.0/{workspace}/extrarest/documents/{fdr_uid}/contents?keyword={str}&search_type={type}&limit={int}&start={int}&user={uid}&only_active={boolean}&direction={dir}&sort={field}&search={str}`

**URL parameters:**  
  * _string_ `fdr_uid`: The unique ID of the folder or `root` if the base folder "/".
  
**Optional query string parameters:**  
  * `keyword={str}` Only return files which have the specified keyword in 
    the filename or in the document's tag if `search_type=TAG`.
  * `search_type={type}`: The type of search, which can be `TAG` or `ALL`.
  * `limit={int}`: The maximum number of folders to return. If set to `0`, 
    which is the default, then an unlimited number of folders will be returned.  
  * `start={int}`: The number where to start listing folders. If set to `0`, 
    which is the default, then will start from the first folder. This parameter
    is often used in conjunction with `limit={int}` when only displaying a
    fixed number of folders at a time.
  * `user={uid}`: Only return files that can be accessed by the specified user, 
    indicated by his/her unique ID. This parameter can only be specified 
    if the logged-in user has the PM_ALLCASES permission in his/her role. 
  * `only_active={boolean}`: Set to `true` if only active files (not deleted and 
    not overwritten with new versions) will be returned. Default is `false` so all files 
    will be returned.
  * `direction={dir}`: The sort direction which can be `ASC` (ascending which is 
    the default) or `DESC` (descending).
  * `sort={field}`: The field used to sort the folders, which can be `appDocCreateDate`
    (the date the folder was first created), `name` (the folder name) or "``" (no sort order).
  * `search={str}`: A case insensitive string to search for in the folder names. 
    Make sure to use a function such as PHP's url_encode() or JavaScript's 
    encodeURIComponent() so `" "` (space) becomes `%20` and `"é"` becomes `%C3%A9`.
    
**Response:**  
* An object holding the following two properties:
    * `documents`: An array of document objects. 
    * `totalDocumentsCount`: The total number of documents in the directory.

**Example:**  
*Request:*  
`GET http://example.com/api/1.0/workflow/extrarest/documents/root/contents`

*Response:*
```javascript
{
   "documents": [
      {
         "APP_DOC_UID": "4674916455a9611cfa0e8e6010566295",
         "APP_DOC_FILENAME": "invoice_Acme_Inc",
         "APP_DOC_COMMENT": null,
         "DOC_VERSION": 2,
         "APP_UID": "4731930415a9611cb69fca4029962239",
         "DEL_INDEX": 1,
         "DOC_UID": "6146400035a8f9fd86551f3042633729",
         "USR_UID": "00000000000000000000000000000001",
         "APP_DOC_TYPE": "OUTPUT",
         "APP_DOC_CREATE_DATE": "2018-02-27 21:21:15",
         "APP_DOC_INDEX": 2,
         "FOLDER_UID": "",
         "APP_DOC_PLUGIN": "",
         "APP_DOC_TAGS": "",
         "APP_DOC_STATUS": "ACTIVE",
         "APP_DOC_STATUS_DATE": null,
         "APP_DOC_FIELDNAME": null,
         "APP_DOC_DRIVE_DOWNLOAD": null,
         "SYNC_WITH_DRIVE": "UNSYNCHRONIZED",
         "SYNC_PERMISSIONS": null,
         "APP_TITLE": "#279",
         "APP_DESCRIPTION": "",
         "APP_NUMBER": 279,
         "APP_PARENT": "",
         "APP_STATUS": "DRAFT",
         "PRO_UID": "4942606665a8f9f9f2860f0087218330",
         "APP_PROC_CODE": "",
         "STATUS": "Draft",
         "CREATOR": "Administrator admin",
         "CREATE_DATE": "2018-02-27 21:19:55",
         "UPDATE_DATE": "2018-02-27 21:21:19",
         "PRO_TITLE": "Purchase Request",
         "OUT_DOC_UID": "6146400035a8f9fd86551f3042633729",
         "OUT_DOC_TITLE": "Invoice for client",
         "OUT_DOC_DESCRIPTION": "",
         "OUT_DOC_FILENAME": "invoice_@#client",
         "OUT_DOC_REPORT_GENERATOR": "HTML2PDF",
         "OUT_DOC_LANDSCAPE": 0,
         "OUT_DOC_MEDIA": "Letter",
         "OUT_DOC_GENERATE": "BOTH",
         "OUT_DOC_TYPE": "HTML",
         "OUT_DOC_CURRENT_REVISION": null,
         "OUT_DOC_FIELD_MAPPING": null,
         "OUT_DOC_VERSIONING": 1,
         "OUT_DOC_DESTINATION_PATH": "",
         "OUT_DOC_TAGS": "",
         "OUT_DOC_PDF_SECURITY_ENABLED": 0,
         "OUT_DOC_PDF_SECURITY_OPEN_PASSWORD": "",
         "OUT_DOC_PDF_SECURITY_OWNER_PASSWORD": "",
         "OUT_DOC_PDF_SECURITY_PERMISSIONS": "",
         "OUT_DOC_OPEN_TYPE": 1,
         "USR_USERNAME": "johndoe",
         "USR_FIRSTNAME": "John",
         "USR_LASTNAME": "Doe",
         "DELETE_LABEL": "Delete",
         "DOWNLOAD_LABEL": ".pdf",
         "DOWNLOAD_LINK": "..\/cases\/cases_ShowOutputDocument?a=4674916455a9611cfa0e8e6010566295&v=2&ext=pdf&random=98480022",
         "DOWNLOAD_LABEL1": ".doc",
         "DOWNLOAD_LINK1": "..\/cases\/cases_ShowOutputDocument?a=4674916455a9611cfa0e8e6010566295&v=2&ext=doc&random=2117881970",
         "APP_DOC_UID_VERSION": "4674916455a9611cfa0e8e6010566295_2"
      },
      {
         "APP_DOC_UID": "6339878345a8664d1553481052427126",
         "APP_DOC_FILENAME": "labor_contract_mary_williams.doc",
         "APP_DOC_COMMENT": "final draft of contract",
         "DOC_VERSION": 1,
         "APP_UID": "6450207755a8664b146d762079795117",
         "DEL_INDEX": 1,
         "DOC_UID": "3361325995a866456c2e8c3050631873",
         "USR_UID": "00000000000000000000000000000001",
         "APP_DOC_TYPE": "INPUT",
         "APP_DOC_CREATE_DATE": "2018-02-15 23:57:53",
         "APP_DOC_INDEX": 1,
         "FOLDER_UID": "",
         "APP_DOC_PLUGIN": "",
         "APP_DOC_TAGS": "INPUT",
         "APP_DOC_STATUS": "ACTIVE",
         "APP_DOC_STATUS_DATE": null,
         "APP_DOC_FIELDNAME": "contractFile",
         "APP_DOC_DRIVE_DOWNLOAD": null,
         "SYNC_WITH_DRIVE": "UNSYNCHRONIZED",
         "SYNC_PERMISSIONS": null,
         "APP_TITLE": "#269",
         "APP_DESCRIPTION": "",
         "APP_NUMBER": 269,
         "APP_PARENT": "",
         "APP_STATUS": "DRAFT",
         "PRO_UID": "1953128925a864f18e70955009266756",
         "APP_PROC_CODE": "",
         "STATUS": "Draft",
         "CREATOR": "Baker Sally",
         "CREATE_DATE": "2018-02-15 23:57:21",
         "UPDATE_DATE": "2018-02-16 00:13:49",
         "PRO_TITLE": "Onboard New Employee",
         "INP_DOC_UID": "3361325995a866456c2e8c3050631873",
         "INP_DOC_TITLE": "Contract File",
         "INP_DOC_DESCRIPTION": "",
         "INP_DOC_FORM_NEEDED": "VIRTUAL",
         "INP_DOC_ORIGINAL": "ORIGINAL",
         "INP_DOC_PUBLISHED": "PRIVATE",
         "INP_DOC_VERSIONING": 0,
         "INP_DOC_DESTINATION_PATH": "",
         "INP_DOC_TAGS": "INPUT",
         "INP_DOC_TYPE_FILE": ".*",
         "INP_DOC_MAX_FILESIZE": 0,
         "INP_DOC_MAX_FILESIZE_UNIT": "KB",
         "USR_USERNAME": "sallybaker",
         "USR_FIRSTNAME": "Sally",
         "USR_LASTNAME": "Baker",
         "DELETE_LABEL": "Delete",
         "DOWNLOAD_LABEL": "Download",
         "DOWNLOAD_LINK": "..\/cases\/cases_ShowDocument?a=6339878345a8664d1553481052427126&v=1",
         "DOWNLOAD_LABEL1": "",
         "DOWNLOAD_LINK1": "",
         "APP_DOC_UID_VERSION": "6339878345a8664d1553481052427126_1"
      },
   ],
   "totalDocumentsCount": 38
}
```

-----------------------
## Version Control

### Version 1.3 (2018-04-23)
Added endpoints:  
* [Get document folders: `GET extrarest/documents/{fdr_uid}/folders`](#get-document-folders-get-extrarestdocumentsfdr_uidfolders)
* [Get documents in folder: `GET extrarest/documents/{fdr_uid}/contents`](#get-documents-in-folder-get-extrarestdocumentsfdr_uidcontents)

### Version 1.2 (2018-04-17)
Added endpoints:  
* [Append rows to a PM Table: `PUT extrarest/pmtable/{pmt_uid}/append`](#append-rows-to-a-pm-table-put-extrarestpmtablepmt_uidappend)
* [Overwrite a PM Table: `PUT extrarest/pmtable/{pmt_uid}/overwrite`](#overwrite-a-pm-table-put-extrarestpmtablepmt_uidoverwrite)

Commented out the code of [Execute database query: `POST extrarest/sql`](#execute-database-query-post-extrarestsql) 
for security reasons, but left it in the source code in case anyone wants to enable it for testing or adapting.

Commented out the code to use `extraRest/setup.xml` because people reported that it caused problems installing the plugin.

### Version 1.1 (2018-04-04)
First version posted on Github. There are several versions 1 which
were posted on the forum, so numbered 1.1 to distinguish this version. 


   
