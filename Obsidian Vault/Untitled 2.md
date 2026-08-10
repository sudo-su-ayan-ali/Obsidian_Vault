`Get-NetUser | select cn` is a PowerShell command used in Active Directory enumeration. It queries the domain for all user objects using the PowerView tool and filters the output to display only the Common Name (`cn`) property for each user. [[1](https://www.youtube.com/watch?v=TLTolJXdny0&t=439), [2](https://quizlet.com/study-guides/powerview-commands-for-active-directory-enumeration-3bb8393b-8ac6-427a-af56-dc310fc9fe83), [3](https://www.geeksforgeeks.org/ethical-hacking/powerview-domain-enumeration/)]

Overview of the Command

- **`Get-NetUser`**: A PowerView function that pulls detailed user data and attributes from Active Directory.

- **`| select cn`**: A pipeline filter that selects and shows only the common names of those user accounts, keeping the screen clean of extra details. [[1](https://powersploit.readthedocs.io/en/stable/Recon/README/), [2](https://quizlet.com/study-guides/powerview-commands-for-active-directory-enumeration-3bb8393b-8ac6-427a-af56-dc310fc9fe83), [3](https://www.geeksforgeeks.org/ethical-hacking/powerview-domain-enumeration/)]

Common Context and Usage

- **Active Directory Discovery**: Used by administrators and security professionals during domain audits. [[1](https://quizlet.com/study-guides/powerview-commands-for-active-directory-enumeration-3bb8393b-8ac6-427a-af56-dc310fc9fe83)]

- **Penetration Testing**: Used during the reconnaissance phase to quickly map out user accounts in a network. [[1](https://www.hackingarticles.in/active-directory-enumeration-powerview/), [2](https://quizlet.com/study-guides/powerview-commands-for-active-directory-enumeration-3bb8393b-8ac6-427a-af56-dc310fc9fe83)]

- For related built-in Windows options, you can review Microsoft Learn's [net user](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-user) documentation or official [Get-ADUser (ActiveDirectory)](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2025-ps) guidelines.

- For step-by-step methods on running these queries, check out the [Active Directory Enumeration PowerView](https://www.hackingarticles.in/active-directory-enumeration-powerview/) guide on Hacking Articles, the Enumerating Active Directory with Powerview/enumerating-active-directory-with-powerview).html walkthrough on Mossé Cyber Security Institute/enumerating-active-directory-with-powerview).html, or troubleshooting notes on [GitHub](https://github.com/PowerShellEmpire/PowerTools/issues/83). [[1](https://github.com/PowerShellEmpire/PowerTools/issues/83)]