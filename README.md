# IBM Tririga Assistant

An assistant for IBM TRIRIGA Workplace Services applications that enables users to book rooms, make service requests, locate people/places and ask questions with an AI assistant. 

![image](images/assistant-1.png)

---

## Introduction

TRIRIGA Assistant is an AI chatbot powered by the IBM Watson Assistant Service. It offers a brand new experience for employees to engage with their workplace services. It allows employees to make room reservations, report issues or lookup people using natural language.  

This is the free basic version of the assistant that is only accessible within TRIRIGA Workplace Experience Apps. The premium version has not yet been released, but it will allow more customizations, additional capabilities and it will open up the platform for creating your own custom skills.

These instructions will guide you through 1)  provisioning a TRIRIGA Assistant skill on the IBM Cloud, 2) installing and testing the OSLC APIs and 3) installing the chatbot interface into your workplace services apps. It is highly advisable that these steps are performed by a developer that is trained and experienced with IBM TRIRIGA.

## Prerequisites

- TRIRIGA 10.6.1/3.6.1 
- TRIRIGA Workplace Services apps deployed
- TRIRIGA Request Central and (optionally) TRIRIGA Reserve 
- TRIRIGA instance that is accessible securely from the internet. TRIRIGA Assistant is a SaaS service hosted securely on IBM cloud. The service will need to communicate via secured OSLC APIs to your TRIRIGA instance. This is usually not an issue for TRIRIGA SaaS customers.

## Estimated time

This installation should take 5-8 hours to complete, not including the time for IBM to provision the TRIRIGA Assistant skill.

## Steps

### Part 1 - Gather data

#### A) GATHER YOUR BUILDING AND ROOM NAMES.

You will need to run a few TRIRIGA Reports to gather a list of buildings, floors, and reservable spaces. These are the reports to run:

- All Buildings with Reservable Spaces: `triBuilding -  Find - All Buildings with Reservable Spaces` 
- Reservable Spaces: `triSpace - Reservable Space`


1. Log in to TRIRIGA account (your account should have access to run all of these - if not, you may need to ask the sys-admin for access)

2. In the top left of the menu click 'My Reports' (directly on 'My Reports', don't select the 'Report Scheduler' sub-menu)

3. Select 'System Reports' from the row of tab headers

4. In the Name filter enter 'triSpace - Reservable Space' and press Enter (or click the 'apply filters')

5. This should list the 'triSpace - Reservable Space' report

6. In the 2nd column click the Paper+Run icon

7. The report will open in a separate window

8. You can use the 'Export' action at the top/bottom to export the results to a `.xlsx` file.

#### B) GATHER THE TIMEZONES FOR EACH BUILDING BY DOING THE FOLLOWING: 

You will need to also run a TRIRIGA Report to gather the time zones for each building.

1.	Find the query named `Location - Find - Find All Property And Buildings` 
2.	Edit the query and add the `TimeZones` business object from the `Classification Module`. 
3.	In the Columns tab for that same query, click the Classification Module and select `Timezone` (triTimeZoneTX). 
4.	Click Save 
5.	Click Run Report 
6.	Click Export from the report window that appears 


### Part 2 - Import OSLC resources, create assistant user and test

#### C) LOAD THE OSLC RESOURCES

1.	Create new Object Migration import package selecting the wa-tri-assistant-*.zip file provided in this folder.
2.	Validate and Import the OM package.
3.	If you can reserve rooms using the Workplace Services apps, then go to next step, otherwise you will need to define buildings, spaces, reservation space groups, reservation coordinators and other things following [this documentation](https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html) ([https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html](https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html))


#### D) SET UP THE ASSISTANT USER.

In order to allow the assistant user to create location reservations and service requests on behalf of other users, we have to create the user with the proper groups and licenses. 

1.  Create a user with user with the following:

    Groups: 
    - TRIRIGA Request Central 
    - TRIRIGA Request Central - Fundamentals 
    - TRIRIGA Request Central - Reserve
    - TRIRIGA Request Central - Reserve - Fundamentals
    - WAS Reserve OSLC
    
    License:
    - IBM Facilities and Real Estate Management on Cloud Self Service, or
    - IBM TRIRIGA Request Central, or
    - IBM TRIRIGA Workplace Reservation Manager

2.	Make note of the user name and password because it will need to be provided to IBM.
3.	Important: Do NOT give the assistant user a primary location.  This user will be used to book rooms and submit service requests on behalf of all users in the TRIRIGA instance.

#### E) TEST THE OSLC ENDPOINTS

Test the OSLC calls using the POSTMAN collection provided in the postman directory.  You will need to change the payload to have 'location/building/space' you have defined in your TRIRIGA instance. A successful test of the OSLC APIs when there are no OSLC errors.
    

### Part 3 - Request the Assistant Integration ID

#### D) REQUEST YOUR ASSISTANT INTEGRATION ID

Your integration ID will be used to connect your TRIRIGA with the Assistant Skill on the cloud. You will use this Integration ID in the next step.

To request your Integration ID simply send the building xls files and the username/password of the assistant account to 
**[jtmoore@us.ibm.com](jtmoore@us.ibm.com)** the `.xls` files generated from the reports for the creation of your assistant.


### Part 4 - Add Assistant UI to Workplace Services apps

#### F) EDIT THE WORKPLACE SERVICES APP TO ADD CHAT UI ACCESS

1.	Once you have received your Integration ID, open the Workplace Service view by going to Tools > Web View Designer > triWorkplaceServices.
2.	Copy the value in `Development Filename` and paste it as the value in the "Production Filename". (Note: This is done for testing purposes and can be reversed after testing passes. A link to instructions have been add below.)
3.	In the View Files section, click on `/trilazy-imports.html`.
4.	Click on the Download View File icon.
5.  Edit the `trilazy-importshtml` file and add the following below the last `<link>` tag line:

    ```html
    <link rel="import" href="../ibmTriAssistant/ibmTriAssistant.html">
    ```

6.  Upload the changes by clicking on the Upload view file icon.
7.  Click `Save & Close` button in upper right corner.

8.	In the View Files section, click on /triview-workplace-services-dev.html
9.	Click on the Download View File icon.
10.	Edit the `triview-workplace-services-devhtml` file and at the bottom of the template section, paste the following lines of code above the `</template>` so the following HTML is between the `<template>` and `</template>` lines:
**Make sure to replace `PASTE_THE_INTEGRATION_ID_HERE` with the Integration ID provided.**
    
    ```html
    <ibm-TriAssistant integration-id="PASTE_THE_INTEGRATION_ID_HERE" region="us-south" model-and-view="ibmTriAssistant" instance-id="-1" online="[[online]]"> </ibm-TriAssistant>
    ```
11. Upload the changes by clicking on the Upload view file icon.
12. Click `Save & Close` button in upper right corner.

#### G) EDIT THE ROOM RESERVATION AND SERVICE REQUEST VIEWS

From the Web View Designer, repeat the same steps directly above for the other views: 
    - triRoomReservation View (edit files `trilazy-imports.html` and `triview-room-reservation-dev.html`).
    - triServiceRequest View (edit files `trilazy-imports.html` and `triview-service-request-dev.html`).

#### H) TEST THE WORKPLACE SERVICES APPS

If all edits were done correctly, you should see a chat icon appear at the bottom right of the Workplace Services apps.  Log into the Workplace Services apps as a user that isn't the system or assistant user and test the assistant.

#### I) (OPTIONAL) VULCANIZE THE VIEWS

If you feel that your workplace service apps are loading much slower after the edits, then you can "vulcanize" the apps [following these instructions](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/IBM TRIRIGA1/page/How to vulcanize your UX application).  If you do this, make sure you undo the change like F2 that sets the `Production Filename` to the `Development Filename`.

### Part 5 - Configure users to use the assistant for reservations

#### J) Allow Assistant user to create reservations on behalf of other users

1.	If assistant will be used to make room reservations, the `TRIRIGAWEB.properties` should have the `SHOW_PREFERENCES_LINK` env var set to `Y` 
    
    - (note you will need to restart your WebSphere server if you have to change this value).

2.	For each user that plans to use the assistant to create location reservations, they must do the following:
    - Click the Welcome, {name} in the main UI home page
    - Click the Preferences tab
    - Click the Reservation Delegates tab
    - In the Reservation Delegates section, click the Find button
    - Click the checkbox for the `triassistant` user and click OK

## Summary

State any closing remarks about the task or goal you described and its importance. Reiterate specific benefits the reader can expect from completing your tutorial. Recommend a next step (with link if possible) where they can continue to expand their skills after completing your tutorial.

## Related links

Include links to other resources that may be of interest to someone who is reading your tutorial.

