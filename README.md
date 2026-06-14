Question 2
Workflow is an app that allows teams to manage and track their work efficiently. The team management featur lets admins create teams, view team details, edit team information, and delete teams when needed. However, t edit and delete team features are not functioning correctly in the backend, and your task is to fix them.
Issue Summary:
Admins can edit team information or delete a team and see a success message, but the updated data does not appear on the screen.
Steps to Reproduce:
• Log in using Admin credentials:
Email: alex@workfLow.dev
Password: Password@123
• Hover over any team in the left sidebar.

Click the • menu.
®
0.
Workflow
Engineering


• Todo 5
ENG-10
• Create admin dashboard ahLow
ENG-9
• Add Redie caching layer
M Medium
© in Progress s
• In Review +
ASSURE
Design •
ENG-22
© Sample Title
ENG-16
Le High
Marketing • Product •
ENG-13
© Implement data export feature
ENG-AS
Low
Medium

50-8
Implement APi rate limiting
Al Medium
ENG -12
© Add QAuth integration
ENG-24
Mi High
At Migh
• Implement webhook system
Create APt documentation
• Add two-factor authentication

over any team in the left sidebar.
Click the . menu.
Workflow
Your teams v
Engineering
ssues
Design >
Marketing>
Product »
Engineering
Todo 5
ENG-18
• Create admin dashboard al Low
ENG-9
• Add Redis caching layer la Medium
ENG 8
O Implement AP rate limiting
L Medium
ENG-7
O Setup CUCD pipeline
Lt High
ENG-6
• Optimize database queries for performance
le Low
© In Progress 5
ENG-22
© Sample Title
High
ENG-L3
Implement data export feature
ah Low
ENG-12
Add Auth integration
High
ENG-11
© Implement GraphQL API
LE Medium
ENG-4
• Build real-time notifications system &i Medium
in Review 4
ENG-16
© Implement webho
Lt Medium
ENG-15
Create API documer
L Medium
ENG-14
Add two-factor auther
& High
ENG-S
Add search functionality
• Choose Edit, update the team information, then click Save changes, or choose Delete to remove the tean
< Back
Edit team
Undate team settings

Notice the team in the sidebar does not update to reflect your changes (or does not disappear after deletion).
Expected Behavior:
• When an admin edits a team's information (name, key, icon, iconColor), the updated team details should be saved and reflected immediately in the sidebar.
• When an admin deletes a team, it should be removed from the sidebar immediately and should no longer appear anywhere in the app.
Refer to the README.md file for more details.
Note: The acceptance criteria for this task require that your solution pass all predefined unit tests. Use failing test cases to guide debugging.

Workflow(React + Django): Team Management
Overview
Workflow is an app that allows teams to manage and track their work efficiently. The team management feature lets admins create teams, view team details, edit team information (name, key, icon, iconColor), and delete teams when needed.
However, the edit and delete team features are not functioning correctly in the backend, and your task is to fix them.
Expected API Behavior
1. PUT /api/teams/:id
Purpose: Updates an existing team
Auth: Required (Bearer token + Admin role)
Request Body:
"name": "Engineering Updated",
"icon": "rocket"
"¿conColor"; "bLue",
"key": "ENG"

Success Response (200):
"message": "Team updated successfully",
"team”：｛
"id": "team_id",
"name": "Engineering Updated",
"key": "ENG",
"icon": "rocket",
"iconColor"; "bLue"
.
"description"; "*
"members": 11,
"createdAt®;2024-01-01T00:00:00.000Z",
"updatedAt"; "2024-01-01T00: 30:00.000Z"
Error Responses:
Non-admin user (403):

{ "message": "Admin access required" }
Team not found (404):
{ "message": "Team not found" }
Duplicate key (400):
"message": "Team key already exists" }
2. DELETE /api/teams/:id
Purpose: Deletes a team permanentiy
Auth: Required (Bearer token + Admin role)
Success Response (200):
"message": "Team deleted successfully" }
Error Responses:
Non-admin user (403):
{ "message"; "Admin access required" }
Team not found (404):
"message": "Team not found" }
I


Additional Information
• Only users with the admin role can update or delete teams.
• Team keys are stored in uppercase. If the key is updated, it must be checked for uniqueness.
• To manually reset the database, stop the running server and restart it.
• To reset the database without restarting the server, run pm run reset-db -
• If you're using Run and Debug mode in the IDE, the frontend server may start before the backend (including database seeding) is ready.
Please reload the preview once the backend setup is complete.



