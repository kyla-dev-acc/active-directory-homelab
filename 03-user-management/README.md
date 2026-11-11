# 03 - User and Group Management

This folder documents how I organized the `abc.local` domain into Organizational Units (OUs) and created the company's employee accounts. It builds on the domain created in [02-active-directory-setup](../02-active-directory-setup/).

## OU Structure

```
abc.local
│
└── OU=ABC-Company
    │
    ├── OU=Departments
    │   ├── OU=IT
    │   ├── OU=HR
    │   └── OU=Finance
    │
    ├── OU=Workstations
    ├── OU=Groups
    └── OU=Service-Accounts
```

## Employees Created

| Employee | Username | Department |
|----------|----------|------------|
| John Santos | jsantos | IT |
| Maria Reyes | mreyes | HR |
| Carlo Dela Cruz | cdelacruz | Finance |
| Ana Garcia | agarcia | IT |
| Mark Villanueva | mvillanueva | HR |
| Lisa Mendoza | lmendoza | Finance |

## Contents

| File | Description |
|------|-------------|
| [organizational-units.md](organizational-units.md) | Building the OU hierarchy, and why users and computers go in separate OUs |
| [user-creation.md](user-creation.md) | Creating the six employee accounts with temporary passwords, and the wrong-OU troubleshooting scenario |

More guides will be added here as I complete the next tasks: security groups and permissions and access.

## What I Accomplished

- Built a department-based OU hierarchy under a single `ABC-Company` OU, keeping the domain root clean.
- Created six user accounts, each in the correct department OU.
- Configured temporary passwords with a forced change at next logon, plus job descriptions.
- Explained how OU placement affects Group Policy and delegation.

## Skills Demonstrated

- Active Directory Users and Computers (ADUC)
- OU design and AD structure planning
- User account creation and lifecycle basics
- PowerShell for AD (`New-ADOrganizationalUnit`, `New-ADUser`, `Get-ADUser`)
- Troubleshooting OU placement mistakes

## Next

Security groups and permissions, followed by Group Policy management in `04-group-policy-management`.
