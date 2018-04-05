# Extra Functions Plugin (`extraFunctions-0.*X*.tar`)

The extraFunctions plugin provides useful functions that can be be used in 
ProcessMaker triggers and plugins.

Author:    Amos Batto (amos@processmaker.com)  
Version:   0.1 (2018-04-04)  
Tested in: ProcessMaker 3.2.1 Community  
License:   Public Domain  

## Functions
The following functions are available in the extraFunctions plugin.

-----------
### PMFGetUidFromUsername()

`PMFGetUidFromUsername()` gets the unique ID of a user from the username.

  _variant_ PMFGetUidFromUsername($username)

**Parameters:**  
  * _string_ `$username`: The username of a user.
 
**Return Value:**  
Returns the unique ID of user; otherwise returns `FALSE`.

**Example 1:**  
The process designer needs to assign the next task in the process based on the 
order amount. If the amount is over 500, then the order needs to be approved 
by the district manager John Doe. If the order is less than 500, then the order 
needs to be approved by the local manager Jane Row:
```php
if (!empty(@#orderAmount) and @#orderAmount >= 500) { 
   @@nextAssignedUser = PMFGetUidFromUsername("johndoe");
} else {
   @@nextAssignedUser = PMFGetUidFromUsername("janerow");
}
```

**Example 2:**  
This function is also used when creating new cases that are assigned to a particular user.
In this example, a new case needs to be created which is assigned to the district manager 
if there are observations in the case:
```php
if (isset(@@observations) and trim(@@observations) != '') {
   PMFNewCase($processId, PMFGetUidFromUsername("johndoe"), $taskId, 
      array("observations"=>@@observations));
}
```
------------------
### PMFCalculateDate()

`PMFCalculateDate()` calculates the date for a specified time duration based 
on the configured calendar for a specified user, task and process. In
ProcessMaker, the calendar of the user has priority. If the user doesn't have
a calendar, then the calendar of the task is used. If the task doesn't have a 
calendar, then the calendar of the process is used. 

  *string* PMFCalculateDate($startTime, $duration, $timeUnits = 'DAYS',
     $userUid = null, $taskUid = null, $processUid = null)

**Parameters:**  
  * _string_ `$startTime`: Datetime in "YYYY-MM-DD HH:MM:SS" format from which to start.
  * _float_  `$duration`: Number of time units to add to the $startTime.
  * _string_ `$timeUnits`: The unit of time for the $duration, which can be: 'DAYS' (default), 'HOURS' or 'MINUTES'
  * _string_ `$userUid`: Unique ID of user assigned to the task. Default is the current logged-in user.
  * _string_ `$taskUid`: Unique ID of the task whose duration will be calculated. Default is the current task.
  * _string_ `$processUid`: Unique ID of the process. Default is the current process.  
  
**Return Value:**  
A string representing the calculated datetime in `"YYYY-MM-DD HH:MM:SS"` format.

**Example:**  
Get the date in 10 working days using configured calendar for the current user 
and task in the current case:
```php
@@date = PMFCalculateDate("2017-04-04 12:30:20", 10, "DAYS"); 
```
------------------------
### PMFTaskDuration()
`PMFTaskDuration()` calculates the amount of time a task has taken, depending
on the calendar which is configured for the task or the assigned user.

  _variant_ PMFTaskDuration($startTime, $endTime = null, $timeUnits = 'seconds',
       $userUid = null, $taskUid = null, $processUid = null)

**Parameters:**  
  * _string_ `$startTime`:  Time when task started in YYYY-MM-DD HH:MM:SS format.
  * _string_ `$endTime`:    Time when the task ended in YYYY-MM-DD HH:MM:SS format. If left blank or not included, then set to current time
  * _string_ `$timeUnits`:  The unit of time which is returned by the function, which can be: "seconds" (default), "minutes", "hours", "days", "string" or "array"
  * _string_ `$userUid`:    Unique ID of user assigned to the task. Default is the current logged-in user.
  * _string_ `$taskUid`:    Unique ID of the task whose duration will be calculated. Default is the current task.
  * _string_ `$processUid`: Unique ID of the process. Default is the current process.  

**Return Value:**  
Floating point number (or array if `$timeUnits` is set to `"array"`)

**Example 1:**  
Check how much time has elapsed in the current task:
```php
$c = new Cases();
$aCase = $c->LoadCase(@@APPLICATION, @%INDEX);
@%TaskSeconds = PMFTaskDuration($aCase['DEL_DELEGATE_DATE']);
```

**Example 2:**  
Display a grid of tasks, sorted by task and user.
```php
$sql = "SELECT PRO_UID, TAS_UID, USR_UID, DEL_DELEGATE_DATE, DEL_FINISH_DATE
   FROM APP_DELEGATION WHERE DEL_THREAD_STATUS='CLOSED' GROUP BY TAS_UID, USR_UID";
$aTasks = executeQuery($sql);

//Calculate task times:
for ($i=1; $i <= count($aTasks); $i++) {
   $aTasks[$i]['TASK_TIME'] = PMFTaskDuration(
      $aTasks[$i]['USR_UID'],
      $aTasks[$i]['PRO_UID'],
      $aTasks[$i]['TAS_UID'],
      $aTasks[$i]['DEL_DELEGATE_DATE'],
      $aTasks[$i]['DEL_FINISH_DATE'],
      'string'
   );         
}
@=TasksGrid = $aTasks;
```
-------------------------
### PMFActivityInfo()
`PMFActivityInfo()` returns an array of information about an activity 
which can be a task or subprocess. 

  _array_ PMFActivityInfo($activityUid)

**Parameters:**  
  * _string_ `$uid`: The unique ID of an activity (task or subprocess).
 
**Return Value:**  
A associative array of information about the specified activity.

**Example:**  
Get the expected time to complete the current task in the current case:
```php
$taskId = @@TASK;
$aTaskInfo = PMFActivityInfo($taskId);
@@taskDuration = $aTaskInfo['TAS_DURATION'] .' '. $aTaskInfo['TAS_TIMEUNIT'];
```
In this example `$aTaskInfo` contains:
```php
array(
   "PRO_UID"                           => "525089953576c664761a208010895181"
   "TAS_UID"                           => "173419697576c697007da98073154447"
   "TAS_TYPE"                          => "NORMAL"
   "TAS_DURATION"                      => 1.0
   "TAS_DELAY_TYPE"                    => ""
   "TAS_TEMPORIZER"                    => 0.0
   "TAS_TYPE_DAY"                      => ""
   "TAS_TIMEUNIT"                      => "DAYS"
   "TAS_ALERT"                         => "FALSE"
   "TAS_PRIORITY_VARIABLE"             => ""
   "TAS_ASSIGN_TYPE"                   => "BALANCED"
   "TAS_ASSIGN_VARIABLE"               => "@@SYS_NEXT_USER_TO_BE_ASSIGNED"
   "TAS_GROUP_VARIABLE"                => NULL
   "TAS_MI_INSTANCE_VARIABLE"          => "@@SYS_VAR_TOTAL_INSTANCE"
   "TAS_MI_COMPLETE_VARIABLE"          => "@@SYS_VAR_TOTAL_INSTANCES_COMPLETE"
   "TAS_ASSIGN_LOCATION"               => "FALSE"
   "TAS_ASSIGN_LOCATION_ADHOC"         => "FALSE"
   "TAS_TRANSFER_FLY"                  => "FALSE"
   "TAS_LAST_ASSIGNED"                 => "0"
   "TAS_USER"                          => "0"
   "TAS_CAN_UPLOAD"                    => "FALSE"
   "TAS_VIEW_UPLOAD"                   => "FALSE"
   "TAS_VIEW_ADDITIONAL_DOCUMENTATION" => "FALSE"
   "TAS_CAN_CANCEL"                    => "FALSE"
   "TAS_OWNER_APP"                     => "FALSE"
   "STG_UID"                           => ""
   "TAS_CAN_PAUSE"                     => "FALSE"
   "TAS_CAN_SEND_MESSAGE"              => "TRUE"
   "TAS_CAN_DELETE_DOCS"               => "FALSE"
   "TAS_SELF_SERVICE"                  => "FALSE"
   "TAS_START"                         => "FALSE"
   "TAS_TO_LAST_USER"                  => "FALSE"
   "TAS_SEND_LAST_EMAIL"               => "FALSE"
   "TAS_DERIVATION"                    => "NORMAL"
   "TAS_POSX"                          => 444
   "TAS_POSY"                          => 65
   "TAS_WIDTH"                         => 110
   "TAS_HEIGHT"                        => 60
   "TAS_COLOR"                         => ""
   "TAS_EVN_UID"                       => ""
   "TAS_BOUNDARY"                      => ""
   "TAS_DERIVATION_SCREEN_TPL"         => ""
   "TAS_SELFSERVICE_TIMEOUT"           => 0
   "TAS_SELFSERVICE_TIME"              => ""
   "TAS_SELFSERVICE_TIME_UNIT"         => ""
   "TAS_SELFSERVICE_TRIGGER_UID"       => ""
   "TAS_SELFSERVICE_EXECUTION"         => "EVERY_TIME"
   "TAS_TITLE"                         => "Level2-1"
   "TAS_DESCRIPTION"                   => ""
   "TAS_DEF_TITLE"                     => ""
   "TAS_DEF_DESCRIPTION"               => ""
   "TAS_DEF_PROC_CODE"                 => ""
   "TAS_DEF_MESSAGE"                   => ""
   "TAS_DEF_SUBJECT_MESSAGE"           => ""
   "ASSIGNED_USERS"                    => array()
)  
```

`PMFActivityInfo()` will return an associative array like this for a subprocess:
```php
array(
   "PRO_UID"                     => "525089953576c664761a208010895181"
   "TAS_UID"                     => "789438304576d6eda1d4ea7008448515"
   "TAS_TYPE"                    => "SUBPROCESS"
   "TAS_DURATION"                => 1.0
   "TAS_DELAY_TYPE"              => ""
   "TAS_TEMPORIZER"              => 0.0
   "TAS_TYPE_DAY"                => ""
   "TAS_TIMEUNIT"                => "DAYS"
   "TAS_ALERT"                   => "FALSE"
   "TAS_PRIORITY_VARIABLE"       => ""
   "TAS_ASSIGN_TYPE"             => "BALANCED"
   "TAS_ASSIGN_VARIABLE"         => "@@SYS_NEXT_USER_TO_BE_ASSIGNED"
   "TAS_GROUP_VARIABLE"          => NULL
   "TAS_MI_INSTANCE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCE"
   "TAS_MI_COMPLETE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCES_COMPLETE"
   "TAS_ASSIGN_LOCATION"         => "FALSE"
   "TAS_ASSIGN_LOCATION_ADHOC"   => "FALSE"
   "TAS_TRANSFER_FLY"            => "FALSE"
   "TAS_LAST_ASSIGNED"           => "0"
   "TAS_USER"                    => "0"
   "TAS_CAN_UPLOAD"              => "FALSE"
   "TAS_VIEW_UPLOAD"             => "FALSE"
   "TAS_VIEW_ADDITIONAL_DOCUMENTATION" => "FALSE"
   "TAS_CAN_CANCEL"              => "FALSE"
   "TAS_OWNER_APP"               => "FALSE"
   "STG_UID"                     => ""
   "TAS_CAN_PAUSE"               => "FALSE"
   "TAS_CAN_SEND_MESSAGE"        => "TRUE"
   "TAS_CAN_DELETE_DOCS"         => "FALSE"
   "TAS_SELF_SERVICE"            => "FALSE"
   "TAS_START"                   => "FALSE"
   "TAS_TO_LAST_USER"            => "FALSE"
   "TAS_SEND_LAST_EMAIL"         => "FALSE"
   "TAS_DERIVATION"              => "NORMAL"
   "TAS_POSX"                    => 380
   "TAS_POSY"                    => 475
   "TAS_WIDTH"                   => 110
   "TAS_HEIGHT"                  => 60
   "TAS_COLOR"                   => ""
   "TAS_EVN_UID"                 => ""
   "TAS_BOUNDARY"                => ""
   "TAS_DERIVATION_SCREEN_TPL"   => ""
   "TAS_SELFSERVICE_TIMEOUT"     => 0
   "TAS_SELFSERVICE_TIME"        => ""
   "TAS_SELFSERVICE_TIME_UNIT"   => ""
   "TAS_SELFSERVICE_TRIGGER_UID" => ""
   "TAS_SELFSERVICE_EXECUTION"   => "EVERY_TIME"
   "TAS_TITLE"                   => "Select cases to reopen"
   "TAS_DESCRIPTION"             => ""
   "TAS_DEF_TITLE"               => ""
   "TAS_DEF_DESCRIPTION"         => ""
   "TAS_DEF_PROC_CODE"           => ""
   "TAS_DEF_MESSAGE"             => ""
   "TAS_DEF_SUBJECT_MESSAGE"     => ""
   "ASSIGNED_USERS"              => array()
)  
```
-----------------------------------
### PMFNextActivities()
`PMFNextActivities()` returns a list of the next activities (tasks and subprocesses)
in the process which come after a specified task or subprocess. It works with 
both BPMN and normal processes.

  _array_ PMFNextActivities($activityUid)

**Parameters:**  
  * _string_ `$activityUid`: The unique ID of an activity (task or subprocess). 

**Return Value:**  
A array of associative arrays with information about the activities.

**Example:**  
Get the names of the tasks following the current task in the case to display 
in a textarea:
```php
$taskId = @@TASK;
$aTasks = PMFNextActivities($taskId);
@@nextTasks = '';

foreach ($aTasks as $aTask) {
   @@nextTasks .= (empty(@@nextTasks) ? '' : "\n") . $aTask['TAS_TITLE'];
} 
```

When `PMFNextActivities()` is executed, it will return an array of associative arrays like this:
```php
array(
  array(
    "PRO_UID"                     => "525089953576c664761a208010895181"
    "TAS_UID"                     => "789438304576d6eda1d4ea7008448515"
    "TAS_TYPE"                    => "SUBPROCESS"
    "TAS_DURATION"                => 1.0,
    "TAS_DELAY_TYPE"              => ""
    "TAS_TEMPORIZER"              => 0.0
    "TAS_TYPE_DAY"                => ""
    "TAS_TIMEUNIT"                => "DAYS"
    "TAS_ALERT"                   => "FALSE"
    "TAS_PRIORITY_VARIABLE"       => ""
    "TAS_ASSIGN_TYPE"             => "BALANCED"
    "TAS_ASSIGN_VARIABLE"         => "@@SYS_NEXT_USER_TO_BE_ASSIGNED"
    "TAS_GROUP_VARIABLE"          => NULL
    "TAS_MI_INSTANCE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCE"
    "TAS_MI_COMPLETE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCES_COMPLETE"
    "TAS_ASSIGN_LOCATION"         => "FALSE"
    "TAS_ASSIGN_LOCATION_ADHOC"   => "FALSE"
    "TAS_TRANSFER_FLY"            => "FALSE"
    "TAS_LAST_ASSIGNED"           => "0"
    "TAS_USER"                    => "0"
    "TAS_CAN_UPLOAD"              => "FALSE"
    "TAS_VIEW_UPLOAD"             => "FALSE"
    "TAS_VIEW_ADDITIONAL_DOCUMENTATION" => "FALSE"
    "TAS_CAN_CANCEL"              => "FALSE"
    "TAS_OWNER_APP"               => "FALSE"
    "STG_UID"                     => ""
    "TAS_CAN_PAUSE"               => "FALSE"
    "TAS_CAN_SEND_MESSAGE"        => "TRUE"
    "TAS_CAN_DELETE_DOCS"         => "FALSE"
    "TAS_SELF_SERVICE"            => "FALSE"
    "TAS_START"                   => "FALSE"
    "TAS_TO_LAST_USER"            => "FALSE"
    "TAS_SEND_LAST_EMAIL"         => "FALSE"
    "TAS_DERIVATION"              => "NORMAL"
    "TAS_POSX"                    => 380
    "TAS_POSY"                    => 475
    "TAS_WIDTH"                   => 110
    "TAS_HEIGHT"                  => 60
    "TAS_COLOR"                   => ""
    "TAS_EVN_UID"                 => ""
    "TAS_BOUNDARY"                => ""
    "TAS_DERIVATION_SCREEN_TPL"   => ""
    "TAS_SELFSERVICE_TIMEOUT"     =>     int(0)
    "TAS_SELFSERVICE_TIME"        => ""
    "TAS_SELFSERVICE_TIME_UNIT"   => ""
    "TAS_SELFSERVICE_TRIGGER_UID" => ""
    "TAS_SELFSERVICE_EXECUTION"   => "EVERY_TIME"
    "TAS_TITLE"                   => "Select cases to reopen"
    "TAS_DESCRIPTION"             => ""
    "TAS_DEF_TITLE"               => ""
    "TAS_DEF_DESCRIPTION"         => ""
    "TAS_DEF_PROC_CODE"           => ""
    "TAS_DEF_MESSAGE"             => ""
    "TAS_DEF_SUBJECT_MESSAGE"     => ""
    "ASSIGNED_USERS"              => array()
  )
  array(
    "PRO_UID"                     => "525089953576c664761a208010895181"
    "TAS_UID"                     => "173419697576c697007da98073154447"
    "TAS_TYPE"                    => "NORMAL"
    "TAS_DURATION"                => 1.0
    "TAS_DELAY_TYPE"              => ""
    "TAS_TEMPORIZER"              => 0.0
    "TAS_TYPE_DAY"                => ""
    "TAS_TIMEUNIT"                => "DAYS"
    "TAS_ALERT"                   => "FALSE"
    "TAS_PRIORITY_VARIABLE"       => ""
    "TAS_ASSIGN_TYPE"             => "BALANCED"
    "TAS_ASSIGN_VARIABLE"         => "@@SYS_NEXT_USER_TO_BE_ASSIGNED"
    "TAS_GROUP_VARIABLE"          => NULL
    "TAS_MI_INSTANCE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCE"
    "TAS_MI_COMPLETE_VARIABLE"    => "@@SYS_VAR_TOTAL_INSTANCES_COMPLETE"
    "TAS_ASSIGN_LOCATION"         => "FALSE"
    "TAS_ASSIGN_LOCATION_ADHOC"   => "FALSE"
    "TAS_TRANSFER_FLY"            => "FALSE"
    "TAS_LAST_ASSIGNED"           => "0"
    "TAS_USER"                    => "0"
    "TAS_CAN_UPLOAD"              => "FALSE"
    "TAS_VIEW_UPLOAD"             => "FALSE"
    "TAS_VIEW_ADDITIONAL_DOCUMENTATION" => "FALSE"
    "TAS_CAN_CANCEL"              => "FALSE"
    "TAS_OWNER_APP"               => "FALSE"
    "STG_UID"                     => ""
    "TAS_CAN_PAUSE"               => "FALSE"
    "TAS_CAN_SEND_MESSAGE"        => "TRUE"
    "TAS_CAN_DELETE_DOCS"         => "FALSE"
    "TAS_SELF_SERVICE"            => "FALSE"
    "TAS_START"                   => "FALSE"
    "TAS_TO_LAST_USER"            => "FALSE"
    "TAS_SEND_LAST_EMAIL"         => "FALSE"
    "TAS_DERIVATION"              => "NORMAL"
    "TAS_POSX"                    => 444
    "TAS_POSY"                    => 65
    "TAS_WIDTH"                   => 110
    "TAS_HEIGHT"                  => 60
    "TAS_COLOR"                   => ""
    "TAS_EVN_UID"                 => ""
    "TAS_BOUNDARY"                => ""
    "TAS_DERIVATION_SCREEN_TPL"   => ""
    "TAS_SELFSERVICE_TIMEOUT"     => 0
    "TAS_SELFSERVICE_TIME"        => ""
    "TAS_SELFSERVICE_TIME_UNIT"   => ""
    "TAS_SELFSERVICE_TRIGGER_UID" => ""
    "TAS_SELFSERVICE_EXECUTION"   => "EVERY_TIME"
    "TAS_TITLE"                   => "Level2-1"
    "TAS_DESCRIPTION"             => ""
    "TAS_DEF_TITLE"               => ""
    "TAS_DEF_DESCRIPTION"         => ""
    "TAS_DEF_PROC_CODE"           => ""
    "TAS_DEF_MESSAGE"             => ""
    "TAS_DEF_SUBJECT_MESSAGE"     => ""
    "ASSIGNED_USERS"              => array()
  )
)
```




