# Teams Adaptive Card Reminders From Lists

## Summary

This sample will send a *Microsoft Teams* adaptive card reminder from *Microsoft Lists* to item owner in advance of the due date using *Power Automate*.

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

 1. Download the solution from `solution` folder: TeamsAdaptiveCardRemindersFromLists.zip
 2. Import the solution from [Power Automate](https://flow.microsoft.com/), click **My flows** and then click **Import**