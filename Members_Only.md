# Members Only

[[TOC]]

---
# About
\
Members Only is a security feature to prevent others from viewing records assigned to specific teams. This is helpful when your organization has a team that handles sensitive information that does not need to be seen by others.

This guide is to help ServiceNow admins setup Members Only for user groups insuring records can only be seen by specific assignment groups. The goal for Members Only is to work in conjunction with the default out-of-the-box (OOTB) ServiceNow role base security model to insure future upgradeability across ServiceNow releases. 

&nbsp;

---
## How Members Only works
\
Members Only is a custom true/false field on the *Group [sys_user_group]* table. If the Members Only field is set to true, then only the following people can read the record:
- requested for (or) caller
- opened by
- manager of the assignment group
- members of the assignment group
- accounts with the members_only_override role

If you follow the steps below, you can add, remove, or change criteria as warranted to fit your business requirements.

&nbsp;

---
## Follow along
\
ServiceNow offers many ways to solve a problem or configure an operational business model. These instructions are _*a*_ way and does not represent _*the*_ way on creating a Members Only feature. The best way to use these instructions is to read through them and see how the feature can be adopted and modified to fit your organization requirements. 

!!! note Update sets 
    There is an update set that implement this guide; _Members Only_. The update set can be found at [https://github.com/ChristopherCarver/MembersOnly](https://github.com/ChristopherCarver/MembersOnly).   

!!! danger Modifying OOTB Tables
    This guide focuses on creating a robust feature tied closely to the out of the box (OOTB) tables already existing in ServiceNow. There are alternative implementation solutions within ServiceNow to accomplish the same result. Your mileage may vary depending on scope defined by and practices set by your organization and/or development team. This feature is meant to be as open and flexible to meet varying modifications to suit present and future requirements.

&nbsp;

---
# Members Only Foundation
\
The foundation for the Members Only feature is based on the use of _before query_ business rules. This allows for a tighter security control around tables extended from the base *Task [task]* table. Using before query business rules allows for a cleaner experience for accounts with record access roles, when compared to access control lists (ACLs), without sacrificing any security. 

Instead of creating a single before query business rule on base tables like the *Task [task]* table, it is better practice to create a new before query business rule for each table that extends another as warranted; e.g., incidents, request items, sc tasks, etc. etc.. 

!!! note Coverage 
    This guide will setup a before query business rule on the *Incident [incident]* table. The same can easily be applied to other tables by following the example provided.

&nbsp;

---
## Members Only Field
\
The OOTB *Group [sys_user_group]* table contains groups used for assignment. The purpose of the Members Only feature is that when a record (ticket) is assigned to specific group, then that ticket becomes hidden from others. Therefore  a custom field needs to be added to the *Group [sys_user_group]* table to denote which groups are Members Only.

_Instructions:_

1. Navigate to **System Definition > Tables**.
1. Filter the listing where **Name** is **sys_user_group**.
1. Open the **Group** record.
1. Under the **Columns** tab, click **New**.
1. Under the **Dictionary Entry New record** section, fill in the following fields:
    - _Type:_ True/False 
    - _Column label:_ Members only
    - _Column name:_ u_members_only
1. Under the **Default Value** tab, fill in the following fields:
    - _Default value:_ false
1. Click **Submit**.

&nbsp;

---
## Members Only Override Role
\
The *members_only_override* role allows accounts to view records assigned to groups where the *Members only* field is true and they are _not_ a member or manager of the group. 

_Instructions:_

1. Navigate to **User Administration > Roles**.
1. Click **New**.
1. Under the **Role New record** section, fill in the following fields:
    - _Name:_ members_only_override 
    - _Description:_ Accounts can view records assigned to Members Only groups.
1. Click **Submit**.

&nbsp;

---
## Members Only Before Query Business Rule
\
The Members Only before query business rule should allow only the following accounts access to a record (ticket):
- requested for (or) caller
- opened by
- manager of the assignment group
- members of the assignment group
- accounts with the members_only_override role
- accounts with the admin role

!!! note Other Tables 
    The following before query business rule applies the Members Only feature to incident records. The same before query business rule can be applied to other tables to extend Members Only functionality; adjust accordingly.

_Instructions:_

1. Navigate to **System Definition > Business Rules**.
1. Click **New**.
1. Under the **Business Rule New record** section, fill in the following fields:
    - _Name:_ Members Only - Incident
    - _Table:_ Incident [incident]
    - _Advanced:_ true
1. Under the **When to run** tab, fill in the following fields:
    - _When:_ before
    - _Insert:_ false
    - _Update:_ false
    - _Delete:_ false
    - _Query:_ true
1. Under the **Advanced** tab, fill in the following fields:
    - _Script:_ 

```
(function executeRule(current, previous /*null when async*/ ) {
   
    if (gs.getUser().hasRole('admin') || gs.getUser().hasRole('members_only_override')) {
        return;
    }

    var excludeGroups = [];
    var membersOnlyGroups = new GlideRecord('sys_user_group');
    membersOnlyGroups.addQuery('u_members_only', true);
    membersOnlyGroups.query();
    while (membersOnlyGroups.next()) {
        if (!gs.getUser().isMemberOf(membersOnlyGroups.getDisplayValue()) &&
            gs.getUser().getRecord().getValue('sys_id') != membersOnlyGroups.manager) {
            excludeGroups.push(membersOnlyGroups.sys_id);
        }
    }
    var userReference = gs.getUser().getRecord().getValue('sys_id');
    if (excludeGroups.length > 0) {
        current.addQuery('assignment_group', 'NOT IN', excludeGroups.toString()).addOrCondition             ('assignment_group', 'NULL').addOrCondition('caller_id', userReference).addOrCondition('watch_list',      'CONTAINS', userReference);
    }
})(current, previous);
```
1. Click **Submit**.

&nbsp;

---
# Next Steps

With Members Only in place, the next steps will be to turn on Mwmbers Only flag on the groups. 

&nbsp;

---
# Outro

You have now successfully setup the Members Only feature. Congratulations.
