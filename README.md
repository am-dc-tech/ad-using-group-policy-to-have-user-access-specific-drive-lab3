# Group Policy Lab: Restricted Finance Share

## Summary

Created a network share that only one user (`kearl`) can access, then used a Group Policy Object (GPO) to automatically map it as drive `F:` at sign-in. Access is controlled by share and NTFS permissions. The GPO only delivers the drive.

## Environment

| Item | Value |
|------|-------|
| Domain | AMTech.local |
| Domain controller / file server | AM-DC-01 |
| Folder | `C:\CompanyData\Finance` (shared as `Finance`) |
| User | Kenna Earl (`kearl`) |
| Organizational Unit | Financial |
| Group Policy Object | Financial Object |

## Steps

### 1. Create the OU
In **Active Directory Users and Computers**, create an OU named `Financial` under `AMTech.local`.

<img width="195" height="403" alt="Create Financial OU" src="https://github.com/user-attachments/assets/6f6d7832-c2dd-4bbb-b9fe-cfd66e331d40" />

### 2. Move the user
Move Kenna Earl (`kearl`) into the `Financial` OU.

<img width="500" height="342" alt="Move Kenna into OU" src="https://github.com/user-attachments/assets/3b40ee90-d86b-4a4a-8709-7ea185b96481" />

### 3. Share the folder
Share `C:\CompanyData\Finance` as `Finance`. Share permissions: `kearl` and Administrators only.

<img width="362" height="479" alt="Finance folder share permissions" src="https://github.com/user-attachments/assets/a4655eb0-2d49-4ff7-b80e-d1b679c0814b" />

### 4. Set NTFS permissions
On the **Security** tab, disable inheritance and keep only:

- `kearl`: Modify
- Administrators: Full control
- SYSTEM: Full control

<img width="400" height="484" alt="NTFS permissions for kearl" src="https://github.com/user-attachments/assets/d8f8af21-4e81-4d2f-958b-fd93289a7189" />

### 5. Open Group Policy Management
As `Administrator`, open **Server Manager** > **Tools** > **Group Policy Management**.

<img width="500" height="339" alt="Open Group Policy Management" src="https://github.com/user-attachments/assets/40c92b3e-67cd-46f5-afa2-3b102281c23f" />

### 6. Create and link the GPO
Right-click the `Financial` OU > **Create a GPO in this domain, and Link it here**. Name it `Financial Object`.

<img width="752" height="607" alt="Financial Object GPO" src="https://github.com/user-attachments/assets/5b953a62-b63f-4bc5-ac61-5858be5f73b3" />

### 7. Add the drive map
Edit the GPO: **User Configuration** > **Preferences** > **Windows Settings** > **Drive Maps** > **New** > **Mapped Drive**.

- Action: `Create`
- Location: `\\AM-DC-01\Finance`
- Drive letter: `F:`

<img width="600" height="447" alt="Mapped drive settings" src="https://github.com/user-attachments/assets/c5cc251d-1983-4baa-939a-48372e834f57" />

### 8. Sign in as the user
Sign in to a domain-joined machine as `kearl`.

<img width="287" height="54" alt="kearl sign in" src="https://github.com/user-attachments/assets/90f90d13-20ec-4539-9502-7412be205ff5" />

### 9. Apply policy and verify
Run `gpupdate /force` and confirm `F:` appears.

<img width="600" height="458" alt="F drive mapped" src="https://github.com/user-attachments/assets/28de5194-f77a-4ddf-9f15-0be7fb433af4" />

### 10. Test as Administrator
Sign in as `Administrator` and check for the drive. It does not appear, because the GPO only applies to users in the `Financial` OU.

<img width="600" height="449" alt="Administrator does not see F drive" src="https://github.com/user-attachments/assets/9af41f73-0a4a-403a-a1bb-67f8c75510f9" />

## Result

- `kearl` sees `Finance (\\AM-DC-01) (F:)` under **Network locations**.
- The `Administrator` account does not get the drive map, since it is not in the `Financial` OU.

## Best Practices

- Grant access to a **security group** (e.g. `Finance-Users`) instead of an individual user.
- Use clear, distinct names for OUs, shares, and GPOs to avoid confusion (e.g. `Finance-Users` for the group, `Finance` for the share).
- Before deleting an OU, move its users out first. Deleting an OU deletes everything inside it.

## What I Learned

- An OU holds Active Directory objects. It cannot hold a folder.
- Folder access comes from **share and NTFS permissions**, not from the OU or GPO.
- A user GPO only applies to users inside the OU it is linked to.
- Drive Maps in Group Policy Preferences deliver the shortcut. They do not grant access.
- A GPO's name is just a label. Linking is what makes it apply.

## Troubleshooting

**"Failed to enumerate objects in container"**: This can happen if you remove permissions before adding Administrators back. Fix it by taking ownership of the folder and adding Administrators first.
