# IBM Tririga Assistant

An assistant for IBM Tririga Workplace Services applications that enables users to book rooms, make service requests, locate people/places and ask questions with an AI assistant. 

![image](images/assistant-1.png)

---

## Introduction

TRIRIGA Assistant is an AI chatbot powered by the IBM Watson Assistant Service. It offers a brand new experience for employees to engage with their workplace services. It allows employees to make room reservations, report issues or lookup people using natural language.  

This is the free basic version of the assistant that is only accessible within TRIRIGA Workplace Experience Apps. The premium version has not yet been released, but it will allow more customizations, additional capabilities and it will open up the platform for creating your own custom skills.

These instructions will guide you through 1)  provisioning a TRIRIGA Assistant skill on the IBM Cloud, 2) installing and testing the OSLC APIs and 3) installing the chatbot interface into your workplace services apps. It is highly advisable that these steps are performed by a developer that is trained and experienced with IBM TRIRIGA.

## Prerequisites

- TRIRIGA 10.6.1/3.6.1 
- TRIRIGA Workplace Services apps delpoyed
- TRIRIGA Request Central and (optionally) TRIRIGA Reserve 
- TRIRIGA instance that is accessible securely from the internet. TRIRIGA Assistnat is a SaaS service hosted securely on IBM cloud. The service will need to communicate via secured OSLC APIs to your TRIRIGA instance. This is usually not an issue for TRIRIGA SaaS customers.

## Estimated time

This installation should take 5-8 hours to complete, not including the time for IBM to provision the TRIRIGA Assistant skill.

## Steps

### Part 1 - Configure TRIRIGA and Provision your TRIRIGA Assistant Skill

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

#### C) SET UP THE ASSISTANT USER.

In order to allow the assistant user to create location reservations and service requests on behalf of other users, we have to create the user with the proper groups and licenses as well as have users add the assistant account as a Reservation Delegate in their personal preferences.  
1.	Create a user with user with the following groups and licenses:
    
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
    
    
2.	Make note of the user name and password because it will need to be entered into the ai assistant.
3.	Important: Do NOT give the assistant user a primary location.  This user will be used to book rooms and submit service requests on behalf of all users in the TRIRIGA instance.
4.	If assistant will be used to make room reservations, the `TRIRIGAWEB.properties` should have the `SHOW_PREFERENCES_LINK` env var set to `Y` 
    
    - (note you will need to restart your WebSphere server if you have to change this value).

5.	For each user that plans to use the assistant to create location reservations, they must do the following:
    - Click the Welcome, {name} in the main UI home page
    - Click the Preferences tab
    - Click the Reservation Delegates tab
    - In the Reservation Delegates section, click the Find button
    - Click the checkbox for the `triassistant` user and click OK


#### D) REQUEST YOUR ASSISTANT INTEGRATION ID
Your integration ID will be used to connect your TRIRIGA with the Assistant Skill on the cloud. You will use this Integration ID in the last step.

To request your Integration ID simply send the building xls files and the username/password of the assistant account to 
**[jtmoore@us.ibm.com](jtmoore@us.ibm.com)** the `.xls` files generated from the reports for the creation of your assistant.

### Part 2 - Install and Test the OSLC APIs into TRIRIGA

#### E) LOAD THE OSLC RESOURCES

1.	Create new Object Migration import package selecting the wa-tri-assistant-*.zip file provided in this folder.
2.	Validate and Import the OM package.
3.	If you can reserve rooms using the Workplace Services apps, then go to next step, otherwise you will need to define buildings, spaces, reservation space groups, reservation coordinators and other things following [this documentation](https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html) ([https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html](https://www.ibm.com/support/knowledgecenter/en/SSFCZ3_10.5.2/com.ibm.tri.doc/res_overview/t_ctr_manage_reservations.html))

4.	Test the OSLC calls using the POSTMAN collection provided in the postman directory.  You will need to change the payload to have 'location/building/space' you have defined in your TRIRIGA instance. A successful test of the OSLC APIs when there are no OSLC errors, and when the responses from the POSTMAN test take 4-7 seconds or less.

### Part 3 - Install the Chatbot UI and configure it for your assistant

#### F) EDIT THE WORKPLACE SERVICES APPS TO ADD CHAT UI ACCESS

1.	Open the Workplace Service view by going to Tools > Web View Designer > triWorkplaceServices.
2.	Copy the value in `Development Filename` and paste it as the value in the "Production Filename".
3.	In the View Files section, click on /triview-workplace-services-dev.html
4.	Click on the Download View File icon.
5.	Edit the `triview-workplace-services-dev.html` file as follows:
    - at the bottom of the template section, paste the following lines of code above the `</template>` so the following HTML is between the `<template>` and `</template>` lines:
    
        ```html
        <triplat-ds id="currentUser" name="currentUser" data="{{currentUser}}"></triplat-ds>
        <triplat-ds id="primaryLocation" name="primaryLocation" data="{{primaryLocation}}" on-ds-get-complete="_setupWA">                        
        <triplat-ds-context name="currentUser" context-id="[[currentUser._id]]"></triplat-ds-context>
        </triplat-ds>
        ```
    
    - after the `</template>` line add, on a new line, the following line of code:
    
        ```<script src="https://assistant-web.watsonplatform.net/loadWatsonAssistantChat.js"></script>```
        
    - Then at the bottom of the script section, paste the code below BEFORE the last function in the section (before `_handleBackButtonTap` in workplaces services file)
    
        ```
        _setupWA: function() {
            window.loadWatsonAssistantChat({
                integrationID: "PASTE_INTEGRATION_ID_HERE",
                region: "us-south"
            }).then((instance) => {
                function setContext(event) {
                    event.data.context = event.data.context || {};
                    event.data.context.skills = event.data.context.skills || {};
                    event.data.context.skills['main skill'] = event.data.context.skills['main_skill'] || {};
                    event.data.context.skills['main skill'].user_defined = event.data.context.skills['main skill'].user_defined || {};
                    event.data.context.skills['main skill'].user_defined.userContext = event.data.context.skills['main skill'].user_defined.userContext || {};
                    event.data.context.skills['main skill'].user_defined.userContext.location = event.data.context.skills['main skill'].user_defined.userContext.location || {};
                    event.data.context.skills['main skill'].user_defined.userContext.location.building = event.data.context.skills['main skill'].user_defined.userContext.location.building || {};
                    event.data.context.skills['main skill'].user_defined.userContext.name = event.data.context.skills['main skill'].user_defined.userContext.name || {};
                    
                    if (this.primaryLocation.building) { // workplace services and reservation use building
                        event.data.context.skills['main skill'].user_defined.userContext.location.building = this.primaryLocation.building.value;
                    } else if (this.primaryLocation.parentBuilding) { // service request uses parentBuilding 
                        event.data.context.skills['main skill'].user_defined.userContext.location.building = this.primaryLocation.parentBuilding.value;
                    }
                    
                    event.data.context.skills['main skill'].user_defined.userContext.name.first = this.currentUser.firstName;
                    event.data.context.skills['main skill'].user_defined.userContext.name.last = this.currentUser.lastName
                }
                instance.on({type: "pre:send", handler: setContext.bind(this)});
                instance.render();
            });
        },
        ```
    - Upload the changes by clicking on the Upload view file icon.
    - Click `Save & Close` button in upper right corner.
    - From the Web View Designer, repeat the same steps for: 
        - triRoomReservation view (file `triview-room-reservation-dev.html`)
        - triServiceRequest (file `triview-service-request-dev.html`).

## Summary

State any closing remarks about the task or goal you described and its importance. Reiterate specific benefits the reader can expect from completing your tutorial. Recommend a next step (with link if possible) where they can continue to expand their skills after completing your tutorial.

## Related links

Include links to other resources that may be of interest to someone who is reading your tutorial.

