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
