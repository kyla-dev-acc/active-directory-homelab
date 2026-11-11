# Organizational Units (OU) Structure

## Objective

Organize the `abc.local` domain by department by building an OU hierarchy under a single top-level OU, `ABC-Company`, instead of placing objects directly in the domain root.

## Prerequisites

- AD DS installed and DC01 promoted to a domain controller for `abc.local`
- Logged in to DC01 as a Domain Admin
- Tool: **Active Directory Users and Computers** (ADUC)

## Target Structure

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
    │
    ├── OU=Groups
    │
    └── OU=Service-Accounts
```

| OU | Parent | Purpose |
|----|--------|---------|
| ABC-Company | abc.local (domain root) | Top-level container for everything the company owns |
| Departments | ABC-Company | Holds one OU per department |
| IT, HR, Finance | Departments | Department user accounts |
| Workstations | ABC-Company | Computer accounts |
| Groups | ABC-Company | Security and distribution groups |
| Service-Accounts | ABC-Company | Accounts used by services and applications |

## Steps

### 1. Open Active Directory Users and Computers

On DC01: **Server Manager > Tools > Active Directory Users and Computers** (or run `dsa.msc`).

### 2. Create the top-level OU

1. Right-click the domain **abc.local** > **New** > **Organizational Unit**.
2. Name: `ABC-Company`.
3. Leave **Protect container from accidental deletion** checked and click **OK**.

### 3. Create the Departments OU

1. Right-click **ABC-Company** > **New** > **Organizational Unit**.
2. Name: `Departments` and click **OK**.

### 4. Create the department OUs

Right-click **Departments** > **New** > **Organizational Unit** and create each of these:

- `IT`
- `HR`
- `Finance`

### 5. Create the remaining OUs

Right-click **ABC-Company** (not Departments) > **New** > **Organizational Unit** and create each of these:

- `Workstations`
- `Groups`
- `Service-Accounts`

### 6. Verify the hierarchy

Check in ADUC that `Departments`, `Workstations`, `Groups`, and `Service-Accounts` all sit directly under `ABC-Company`, and that `IT`, `HR`, and `Finance` sit under `Departments`. Nothing except `ABC-Company` should be created in the domain root.


## Challenge: Why Separate Users and Computers into Different OUs?

Placing users and computers in separate OUs is a standard practice in real companies for several reasons:

1. **Different Group Policies.** User policies (desktop restrictions, password rules, folder redirection) and computer policies (firewall, software installation, security baselines) apply to different kinds of objects. Separate OUs let each GPO be linked to exactly the right objects.
2. **Delegated administration.** The helpdesk can be given rights to reset passwords in the user OUs without being able to modify computer accounts, and a desktop support team can manage the Workstations OU without touching user accounts.
3. **Easier management and cleanup.** Stale computer accounts, new hires, and departing employees follow different lifecycles. Keeping them apart makes it simple to find and disable old objects.
4. **Better security.** Servers, workstations, and service accounts can have stricter or different policies than regular users, which limits how far a compromised account can reach.
5. **Cleaner structure and troubleshooting.** When a policy misbehaves, it is much easier to find which objects it applies to if the OU layout reflects their role.
6. **Scalability.** As the company grows, new departments or locations slot into the existing structure without redesigning it.

The default `Users` and `Computers` containers are not OUs, so Group Policy cannot be linked to them directly. That is another reason to build a custom OU structure.

## Key Takeaways

- OUs organize objects and are the unit for linking Group Policy and delegating administration.
- Everything is built under one top-level OU (`ABC-Company`), and the domain root stays clean.
- Protecting OUs from accidental deletion helps prevent costly mistakes.

## Next Steps

Create users and place them in the department OUs, then create security groups in the `Groups` OU.
