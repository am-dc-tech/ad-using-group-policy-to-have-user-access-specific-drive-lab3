# Group Policy Lab: Restricted Finance Share

## Summary
Built a network share that only one user (`kearl`) can access, and used a Group Policy Object (GPO) to automatically map it as drive `F:` at sign-in. Folder permissions control access. The OU and GPO control who receives the drive map.

## Environment
- **Domain:** AMTech.local
- **Domain controller / file server:** AM-DC-01
- **Client:** Windows 11, domain-joined
- **Folder:** `C:\CompanyData\Finance` (shared as `Finance`)
- **User:** Kenna Earl (`kearl`)
- **OU:** Financial
- **GPO:** Financial Object

## Lab Goal
Create a shared folder that only `kearl` can open, and deliver it to her as a mapped drive using Group Policy.

## Steps
1. Open up **Sever Manager** and click **Tool** -> **Group Policy Management**  
<img width="500" height="339" alt="ad-GPM" src="https://github.com/user-attachments/assets/40c92b3e-67cd-46f5-afa2-3b102281c23f" />


2. **Create the OU.** In Active Directory Users and Computers, created an OU named `Financial` under AMTech.local

<img width="195" height="403" alt="Screenshot 2026-10-23 135102" src="https://github.com/user-attachments/assets/6f6d7832-c2dd-4bbb-b9fe-cfd66e331d40" />

3. **Move the user.** Moved Kenna Earl (`kearl`) into the Financial OU.

<img width="500" height="342" alt="Move-Kenna-OU" src="https://github.com/user-attachments/assets/3b40ee90-d86b-4a4a-8709-7ea185b96481" />


4. **Share the folder.** Shared `C:\CompanyData\Finance` as `Finance`. Share permissions: only `kearl` (and Administrators).

<img width="362" height="479" alt="Finance-folder-share" src="https://github.com/user-attachments/assets/a4655eb0-2d49-4ff7-b80e-d1b679c0814b" />


5. **Set NTFS permissions.** On the Security tab, disabled inheritance and kept only `kearl` (Modify), Administrators (Full control), and SYSTEM (Full control).

<img width="400" height="484" alt="Security-tab-kearl-permis" src="https://github.com/user-attachments/assets/d8f8af21-4e81-4d2f-958b-fd93289a7189" />


6. **Create the GPO.** In Group Policy Management, right-clicked the Financial OU > *Create a GPO in this domain, and Link it here*, named it `Financial Object`.

<img width="500" height="375" alt="Link-to-FInancial-OU" src="https://github.com/user-attachments/assets/0b1c35c8-6c51-4e37-9ea1-05f0e03a135a" />


7. **Add the drive map.** Edited the GPO: *User Configuration > Preferences > Windows Settings > Drive Maps > New > Mapped Drive*. Action: Create, Location: `\\AM-DC-01\Finance`, Drive letter: `F:`.

<img width="600" height="447" alt="map-drive" src="https://github.com/user-attachments/assets/c5cc251d-1983-4baa-939a-48372e834f57" />


8. **Test.** Signed in as `kearl` on the client, ran `gpupdate /force`, and confirmed `F:` appeared.

<img width="600" height="458" alt="f-drive" src="https://github.com/user-attachments/assets/5b90e092-bc1d-40d0-924e-0c9d67b8aaaa" />


9. **Test** to see if the **Administrator** has access to the same folder when you sign in.

<img width="600" height="449" alt="can-admin-see-f-drive" src="https://github.com/user-attachments/assets/9af41f73-0a4a-403a-a1bb-67f8c75510f9" />



## Result
- `kearl` sees `Finance (\\AM-DC-01) (F:)` under Network locations.
- The admin account does not get the drive map, since it is not in the Financial OU.

## Security

- When deleting an OU make sure to move users out or else everything in the OU will be deleted
- In a real environment, I would grant access to a security group (e.g. `Finance-Users`) instead of an individual user.
- Naming conventions for OU, Share Name, GPO object should be unique or give modifiers example Finance (Keep), Financial (Users)

## What I Learned

- AN Organizational Unit holds Active Directory objects but it cannot hold a folder. 
- Folder access comes from **NTFS and share permissions**, not from the OU or GPO.
- A user GPO only applies to users inside the OU it is linked to.
- Drive Maps under Group Policy Preferences deliver the shortcut. They do not grant access.
- Removing permissions before adding Administrators back can cause the "failed to enumerate objects in container" error. Fix it by taking ownership and adding Administrators first.
- A GPO's name is just a label. Linking is what makes it apply.


