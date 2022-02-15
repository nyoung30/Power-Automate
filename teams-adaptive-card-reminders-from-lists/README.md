# Teams Adaptive Card Reminders From Lists

## Summary

This sample will send a *Microsoft Teams* adaptive card reminder from *Microsoft Lists* to the item owner in advance of the due date using *Power Automate*.

[![Flow overview](/teams-adaptive-card-reminders-from-lists/assets/flow-overview.png "Flow overview")](/teams-adaptive-card-reminders-from-lists/assets/flow-overview.png "Flow overview")

## Requirements
This Flow requires the following Microsoft List columns and settings:
* **Title**
	* Settings: "Require that this column contains information" set to **Yes**
* **DueDate**
	* Type: Date time with "Include time" set to **No**
	* Settings: "Require that this column contains information" set to **Yes**
* **Owner**
	* Type: Person with "Allow multiple selections" set to **No**
	* Settings: "Require that this column contains information" set to **Yes**

## Import Solution

 1. Download the solution file [TeamsAdaptiveCardRemindersFromLists.zip](/teams-adaptive-card-reminders-from-lists/solution/TeamsAdaptiveCardRemindersFromLists.zip)

 2. Import the solution using Power Automate. Click **My flows** and then click **Import** 
 	[![Flow import](/teams-adaptive-card-reminders-from-lists/assets/flow-import.png "Flow import")](/teams-adaptive-card-reminders-from-lists/assets/flow-import.png "Flow import")