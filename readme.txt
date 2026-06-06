Two files. Here's what to do:
Step 1 — Supabase SQL Editor
Run the contents of action-tracker-setup.sql — creates the three tables and all policies. One time only.
Step 2 — GitHub
Upload actions.html to your repo alongside index.html. It will be live at:
https://liamofaolain.github.io/meeting-poll/actions.html
How project passwords work
When you add a project via the Manage Projects screen, the app will show you a message like:
PROJECT_PASSWORDS['cancer-screening'] = 'mypassword123';
Copy that line, open actions.html on GitHub, find the PROJECT_PASSWORDS section near the top, paste it in, and commit. That's the only time you need to edit the file for a new project.
Your organiser password: Wh3el#Kp9mX2$vN