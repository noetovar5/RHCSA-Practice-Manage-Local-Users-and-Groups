# RHCSA-Practice-Manage-Local-Users-and-Groups
RHCSA Practice — Manage Local Users and Groups
Today’s objective: Create and modify local users and groups, assign supplementary group membership, configure password aging, and verify access.
Estimated time: 45 minutes
Lab system: RHEL 9 or RHEL 10
Privileges required: Root or sudo
This builds on your work with permissions and shell commands. Local account and group administration is an explicit RHCSA objective.
Important safety warning
The commands in this lesson change local authentication records in /etc/passwd, /etc/shadow, and /etc/group. Mistakes could affect account access.
Use only the temporary account and group names shown below. Do not modify root, your regular administrative account, or production accounts. Keep your current root or sudo session open until verification and cleanup are complete.
1. The concept in plain language — 8 minutes
Linux uses three important identity values:
- UID: Numeric user identifier
- Primary GID: User’s default group identifier
- Supplementary groups: Additional groups granting access to shared resources
The primary account databases are:
File	Purpose
/etc/passwd	Usernames, UIDs, home directories, shells
/etc/shadow	Password hashes and aging information
/etc/group	Group definitions and membership
/etc/gshadow	Protected group information


Do not edit these files manually during this lesson. Use commands such as useradd, usermod, groupadd, and chage.
Important commands
useradd   Create a user
usermod   Modify a user
userdel   Delete a user
groupadd  Create a group
groupdel  Delete a group
id        Display UID and group membership
getent    Query identity databases
chage     Configure and display password aging

2. Preflight verification — 4 minutes
Become root if necessary:
sudo -i

Confirm your identity:
whoami
id

Expected username:
root

Check whether our temporary identities already exist:
getent passwd labops1
getent passwd labops2
getent group labadmins

Expected result:
No output

No output means the temporary identities are available.
If any command returns an existing record, do not recreate or delete that identity until you confirm it came from a previous copy of this specific lab.
3. Guided practice: create a group and two users — 9 minutes
This section is explicitly guided practice.
Authentication-change warning
The next commands add local identities. They will not change your current account, but they will modify the system’s account databases.
Create a shared administrative group:
groupadd labadmins

Verify it:
getent group labadmins

Expected pattern:
labadmins:x:GID:

Your numeric GID will vary.
Create two users with home directories:
useradd -m -c "RHCSA Lab Operator One" labops1
useradd -m -c "RHCSA Lab Operator Two" labops2

Options:
- -m creates the home directory.
- -c adds a descriptive comment.
- The final value is the username.
Verify the accounts:
getent passwd labops1
getent passwd labops2

Expected patterns:
labops1:x:UID:GID:RHCSA Lab Operator One:/home/labops1:/bin/bash
labops2:x:UID:GID:RHCSA Lab Operator Two:/home/labops2:/bin/bash

Inspect identity and home-directory information:
id labops1
id labops2
ls -ld /home/labops1 /home/labops2

Each user initially has a private primary group with the same name.
4. Add supplementary group membership — 6 minutes
Authentication-change warning
The following commands change authorization through group membership. The -a option is critical: without it, usermod -G can replace existing supplementary memberships.
Add both users to labadmins:
usermod -aG labadmins labops1
usermod -aG labadmins labops2

Understand the options:
- -G labadmins specifies supplementary groups.
- -a appends membership without removing other supplementary groups.
Verify:
id labops1
id labops2
getent group labadmins

Expected group record:
labadmins:x:GID:labops1,labops2

A user already signed in might need a new login session before newly assigned group membership becomes active. The id username command queries the account database directly.
5. Build and test a shared workspace — 8 minutes
Create a temporary shared directory:
mkdir -p /tmp/rhcsa-user-lab/shared
chown root:labadmins /tmp/rhcsa-user-lab/shared
chmod 2770 /tmp/rhcsa-user-lab/shared

What these commands do:
- chown root:labadmins makes root the owner and labadmins the group.
- 2770 grants full access to owner and group.
- The leading 2 enables the setgid bit.
- Setgid on a directory causes new files to inherit the directory’s group.
Verify:
stat -c '%A %a %U %G %n' /tmp/rhcsa-user-lab/shared

Expected:
drwxrws--- 2770 root labadmins /tmp/rhcsa-user-lab/shared

Create a file as labops1:
runuser -u labops1 -- \
  touch /tmp/rhcsa-user-lab/shared/operator1.txt

Create content as labops2:
runuser -u labops2 -- \
  bash -c 'echo "Validated by labops2" > /tmp/rhcsa-user-lab/shared/operator2.txt'

Inspect ownership:
ls -l /tmp/rhcsa-user-lab/shared

Expected group ownership:
labops1 labadmins ... operator1.txt
labops2 labadmins ... operator2.txt

The file owners differ, but both files inherit the labadmins group because the directory has setgid enabled.
Confirm each user can read the other user’s file:
runuser -u labops1 -- \
  cat /tmp/rhcsa-user-lab/shared/operator2.txt

runuser -u labops2 -- \
  ls -l /tmp/rhcsa-user-lab/shared

6. Configure password aging — 5 minutes
Authentication-change warning
Password-aging settings can prevent a user from signing in if configured incorrectly. The next commands affect only temporary labops1.
Display the current settings:
chage -l labops1

Configure:
- Minimum password age: 1 day
- Maximum password age: 90 days
- Expiration warning: 7 days
chage -m 1 -M 90 -W 7 labops1

Options:
- -m 1 prevents another password change for one day.
- -M 90 requires a password change after 90 days.
- -W 7 begins warnings seven days before expiration.
Verify:
chage -l labops1

Expected values include:
Minimum number of days between password change : 1
Maximum number of days between password change : 90
Number of days of warning before password expires : 7

We are deliberately not assigning a password in this lab. That avoids exposing a password in terminal history or training notes.
7. Examine the account databases safely — 2 minutes
Query only the relevant entries:
getent passwd labops1
getent group labadmins

Inspect the password status without revealing a hash:
passwd -S labops1

Because no password was assigned, the status will normally indicate that the password is locked.
Do not display /etc/shadow contents. Password hashes are sensitive even in a lab.
8. Independent challenge — 7 minutes
Complete this without copying the earlier creation commands.
Assignment
Create and configure:
User:  labanalyst
Group: labreports

Requirements:
1. Create labreports.
2. Create labanalyst with:
   - Home directory /home/labanalyst
   - Comment RHCSA Reporting Analyst
   - Bash login shell
3. Make labreports a supplementary group for labanalyst.
4. Configure password aging:
   - Minimum: 2 days
   - Maximum: 60 days
   - Warning: 10 days
5. Create:
/tmp/rhcsa-user-lab/reports

6. Configure the directory so:
   - Owner is root
   - Group is labreports
   - Owner and group have full access
   - Others have no access
   - New files inherit labreports
7. As labanalyst, create:
/tmp/rhcsa-user-lab/reports/status.txt

Containing:
Application status: Healthy

8. Verify the account, group membership, aging configuration, directory mode, file ownership, and file contents.
Do not assign a password.
9. Independent verification
These commands verify the result without giving away every creation command:
getent passwd labanalyst
getent group labreports
id labanalyst
chage -l labanalyst

Verify the home directory:
test -d /home/labanalyst \
  && echo "PASS: home directory exists" \
  || echo "FAIL: home directory missing"

Verify supplementary membership:
id -nG labanalyst | tr ' ' '\n' | grep -qx labreports \
  && echo "PASS: supplementary group is correct" \
  || echo "FAIL: labreports membership missing"

Verify the directory:
stat -c '%A %a %U %G %n' /tmp/rhcsa-user-lab/reports

Expected:
drwxrws--- 2770 root labreports /tmp/rhcsa-user-lab/reports

Verify the file:
stat -c '%A %U %G %n' /tmp/rhcsa-user-lab/reports/status.txt
cat /tmp/rhcsa-user-lab/reports/status.txt

Expected owner and group:
labanalyst labreports

Expected contents:
Application status: Healthy

10. Knowledge check
Answer these without reviewing the lesson:
1. What is the difference between a primary group and a supplementary group?
2. What does useradd -m do?
3. What does usermod -aG do?
4. Why is omitting -a from usermod -G potentially dangerous?
5. Which command verifies a user’s UID and group memberships?
6. Which command queries /etc/passwd through the system account database?
7. What does chage -M 90 configure?
8. What does the leading 2 in directory mode 2770 mean?
9. Why did files created by different users inherit the same group?
10. Which database contains password hashes?
11. Why should /etc/shadow not be displayed or copied into training notes?
12. Why should an administrator keep the current privileged session open while modifying authentication settings?
I would consider this objective mastered when you can independently create a user and group, assign supplementary membership, configure aging, build a shared setgid directory, and verify every result.
Safe cleanup
Authentication-change warning
The following commands delete only the temporary users, their home directories, and temporary groups created in this lesson. Verify every name carefully before running them.
Remove the temporary users:
userdel -r labops1
userdel -r labops2
userdel -r labanalyst

Remove the temporary groups:
groupdel labadmins
groupdel labreports

Filesystem cleanup warning
Confirm that the path is exactly /tmp/rhcsa-user-lab:
rm -rf /tmp/rhcsa-user-lab

Verify complete cleanup:
getent passwd labops1
getent passwd labops2
getent passwd labanalyst
getent group labadmins
getent group labreports
test ! -e /tmp/rhcsa-user-lab \
  && echo "PASS: user-management lab removed safely"

Expected result:
- No account or group records
- PASS: user-management lab removed safely
Finally, leave the root shell if you entered it with sudo -i:
exit
