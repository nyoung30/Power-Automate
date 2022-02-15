# Teams Adaptive Card Reminders From Lists

## Summary

This Flow will sends Microsoft Teams adaptive card reminders from Microsoft Lists in advance of a due date.

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

 1. Download the solution from `solution` folder: TeamsAdaptiveCardRemindersFromLists.zip
 2. Import the solution from [Power Automate](https://flow.microsoft.com/), click **My flows** and then click **Import**