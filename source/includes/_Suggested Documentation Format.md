# Suggested Documentation Format

Each point below represents a section in documentation:

1. Model - list all object types in separate sections, similar to [System Objects](#system-objects) section here.
* Search - this section can contain a table with all supported ES indexes. The column can have columns:
   * Index Name
   * Fields
   * Index Type
   * Description
* Jobs - a table with all supported jobs, columns: 
   * Task Code
   * Input Parameters
   * Output Parameters
   * Description
* Logic Rules - a table with a list of all rules implemented in **logic** service. Suggested columns:
   * Rule Number
   * Trigger
   * Action
* Notifications - this section can contain info on all notifications and notification templates supported in the system. It will contain a list of templates and a table with all notifications supported in the system. Suggested columns:
   * Notification Number
   * Notification Type
   * Trigger
   * Target
   * Description

In general, documentation will contain much more sections than suggested here, depending on the system specifics. At the same time, we believe that virtually any system built with EGF2 will benefit from having sections, suggested above.

