# ConnectWise Control

Setting up this integration allows you to
1. Import customers from Control
2. Import computers from Control
3. Manage all computers in Control without deploying the ImmyBot Agent
4. Remote into computers from the ImmyBot interface using Control (Note: We default to requiring customer consent, you can disable this under Settings->Preferences)
5. Fix the Automate agent using the Control agent (by creating a cross-tenant deployment for the Automate Agent and creating a schedule for your customers)

## Install ImmyBot Extension for Control

![](./.vuepress/images/2021-03-23-19-12-34.png)

![](./.vuepress/images/2021-03-23-19-13-56.png)

![](./.vuepress/images/2021-03-23-19-17-38.png)

## Create RMMLink for Control

![](./.vuepress/images/2022-01-26-00-11-38.png)
![](./.vuepress/images/2022-01-26-00-09-33.png)

FAQ
What custom property do I use?

By default most ConnectWise Control instances you would select 1 for the ClientName CustomProperty field, this is the "Company" property in Control.
Secondary group is any number between 1-8 that you would like to group from based on Control groups.
You can find more infomation about Control custom propertys here;
https://docs.connectwise.com/ConnectWise_Control_Documentation/Get_started/Administration_page/Appearance_page/Add_custom_properties_to_sessions

## Import your customers

![](./.vuepress/images/2021-03-23-18-57-19.png)

![](./.vuepress/images/2021-03-23-19-01-36.png)

Alternatively, you can create/map only certain customers.

When you map a customer from an RMM, the computers will undergo Identification

![](./.vuepress/images/2021-03-23-19-21-03.png)

![](./.vuepress/images/2021-03-23-19-24-06.png)

![](./.vuepress/images/2021-03-23-19-08-30.png)

## Troubleshooting

ConnectWise Control uses Sqlite under the hood, making it vulnerable to performance issues unless aggressive Database Maintenance Tasks are enabled.

While ImmyBot only uses Control (and other integrations) to spawn an out-of-band connection, over the course of time the database can grow as ImmyBot establishes this connection on each machine once every 24 hours to collect inventory.

Commands go into the SessionEvent table with EventType = 44
Responses go into SessionConnectionEvent with EventType = 70

### Automatic Cleanup
Ensure you have the following maintenance actions:
Access Sessions: Purge records of session activity older than 7 days for all events EXCEPT AddedNote

![image](https://user-images.githubusercontent.com/1424395/160918827-b5de85c0-b528-46cc-bb21-5815fcb07fd0.png)

Access Sessions: Purge records of session connections older than 7 days for Host and Guest connections

![image](https://user-images.githubusercontent.com/1424395/160918906-6d6c0485-fe57-4ff8-b52f-99fa03b4cd74.png)

### Manual Cleanup 

If you host Control yourself, you can restore performance by doing the following

Download and Install [DB Browser for SQLite](https://sqlitebrowser.org/) on your server
Open `C:\Program Files (x86)\ScreenConnect\App_Data\session.db`
Navigate to SQL Editor
Paste in the following SQL

#### Soft delete all commands and responses (This usually solves the problem without kicking everyone out of Control)
```sql
update SessionEvents set EventAttributes = 1 where EventType = 44;
update SessionConnectionEvents set EventAttributes = 1 where EventType = 70;
```
You will need to commit the changes in DB Browser. It will prompt you to commit the changes, click yes.

The reason this works is because the UI doesn't fetch soft-deleted items, so things become much snappier. 

However, if it doesn't work, do you following

#### Delete commands and responses older than 7 days
```sql
-- Delete queue commands in db older than 7 days
DELETE
FROM SessionEvent
WHERE (EventType = '44') AND (Time < DATETIME('now', '-7 day')));

-- Delete responses older than 7 days
DELETE
FROM SessionConnectionEvent
WHERE (EventType = '70') AND (Time < DATETIME('now', '-7 day'))
```

