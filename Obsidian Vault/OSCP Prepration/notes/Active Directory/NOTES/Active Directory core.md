# Windows Domains
### Why Active Directory

Picture yourself administering a small business network with only five computers and five employees. In such a tiny network, you will probably be able to configure each computer separately without a problem. You will manually log in to each computer, create users for those who will use them, and make specific configurations for each employee's account. If a user's computer stops working, you will probably go to their location and it on-site.

While this sounds like a very relaxed lifestyle, let's suppose your business suddenly grows and now has 157 computers and 320 different users located across four different offices. Would you still be able to manage each computer as a separate entity, manually configure policies for each of the users across the network and provide on-site support for everyone? The answer is most likely no.

To overcome these limitations, we can use a Windows domain. Simply put, a **Windows domain** is a group of users and computers under the administration of a given business. The main idea behind a domain is to centralise the administration of common components of a Windows computer network in a single repository called **Active Directory (AD)**. The server that runs the Active Directory services is known as a **Domain Controller (DC)**.

The main advantages of having a configured Windows domain are:

- **Centralised identity management:** All users across the network can be configured from Active Directory with minimum effort.
- **Managing security policies:** You can configure security policies directly in Active Directory and apply them to users and computers across the network as needed.

### A Real-World Example

If this sounds a bit confusing, chances are you have already interacted with a Windows domain at some point in your school, university, or workplace.

In school/university networks, you will often be provided with a username and password that you can use on any of the computers available on campus. Your credentials are valid for all machines because whenever you enter them on a machine, the authentication process is forwarded back to Active Directory, where your credentials are checked. Thanks to Active Directory, your credentials don't need to be stored on each machine and are available across the network.

Active Directory is also the component that allows your school/university to restrict your access to the control panel on its machines. Policies will usually be deployed across the network so you don't have administrative privileges on those computers.

# Active Directory
### Parent Domain

We will now be working in the parent (or root) domain of our Active Directory forest. This is the TryHackMe (`THM.LOC`) domain. Connect to the Domain Controller (`ROOTDC.THM.LOC`) through RDP using the following credentials:

```
Credentials

Username: THM\Administrator

Password: learningadisfun1!

IP address: 192.168.10.100

Connection via RDP

xfreerdp /u:Administrator /p:'learningadisfun1!' /d:THM.LOC /v:192.168.10.100 /dynamic-resolution

```

### Active Directory Domain Service

The core of any Windows Domain is the **Active Directory Domain Service (AD DS)**. This service acts as a catalogue that stores information of all of the "objects" that exist on your network. Amongst the many objects supported by AD, we have users, groups, machines, printers, shares and many others. Let's look at some of them:

**Users**

Users are one of the most common object types in Active Directory. Users are one of the objects known as **security principals**, meaning they can be authenticated by the domain and assigned privileges over **resources** like files or printers. You could say that a security principal is an object that can act on network resources.

Users can be used to represent two types of entities:

- **People:** Users will generally represent individuals in your organisation who need to access the network, such as employees.
- **Services:** You can also define users for services such as IIS or MSSQL. Every service requires a user to run, but service users differ from regular users, as they only have the privileges needed to run their specific service.

**Machines**

Machines are another type of object in Active Directory; for every computer that joins the Active Directory domain, a machine object is created. Machines are also considered "security principals" and are assigned an account just as any regular user. This account has somewhat limited rights within the domain itself.

The machine accounts themselves are local administrators on the assigned computer; they are generally not supposed to be accessed by anyone except the computer itself, but as with any other account, if you have the password, you can use it to log in.

  
![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861) **Note:** Machine account passwords are automatically rotated and typically consist of 120 random characters.

Identifying machine accounts is relatively easy. They follow a specific naming scheme. The machine account name is the computer's name followed by a dollar sign. For example, a machine named `ROOTDC` will have a machine account called `ROOTDC$`.

**Security Groups**

If you are familiar with Windows, you probably know that you can define user groups to assign access rights to files or other resources to entire groups rather than individual users. This allows for better manageability as you can add users to an existing group, and they will automatically inherit all of the group's privileges. Security groups are also considered security principals and can therefore have privileges over network resources.

Groups can contain both users and machines as members. If needed, groups can include other groups as well.

Several groups are created by default in a domain that can be used to grant specific privileges to users. As an example, here are some of the most important groups in a domain:

|   |   |
|---|---|
|**Security Group**|**Description**|
|Domain Admins|Users of this group have administrative privileges over the entire domain. By default, they can administer any computer on the domain, including the DCs.|
|Server Operators|Users in this group can administer Domain Controllers. They cannot change any administrative group memberships.|
|Backup Operators|Users in this group are allowed to access any file, ignoring their permissions. They are used to perform backups of data on computers.|
|Account Operators|Users in this group can create or modify other accounts in the domain.|
|Domain Users|Includes all existing user accounts in the domain.|
|Domain Computers|Includes all existing computers in the domain.|
|Domain Controllers|Includes all existing DCs on the domain.|

You can obtain the complete list of default security groups from the [Microsoft documentation(opens in new tab)](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups).

### Active Directory Users and Computers

To configure users, groups or machines in Active Directory, we need to log in to the Domain Controller and run "Active Directory Users and Computers" from the Start menu:

This will open up window that shows the hierarchy of users, computers, and groups in the domain. These objects are organised in **Organizational Units (OUs),** which are container objects that allow you to classify users and machines. OUs are mainly used to define sets of users with similar policing requirements. The people in the Sales department of your organisation are likely to have a different set of policies applied to them than those in IT, for example. Keep in mind that a user can belong to only one OU at a time.

Checking our machine, we can see that there is already an OU called `THM` with five child OUs for the IT, Management, Marketing, R&D, and Sales departments. It is very typical to see the OUs mimic the business's structure, as it allows for efficiently deploying baseline policies that apply to entire departments. Remember that while this would be the expected model most of the time, you can define OUs arbitrarily. Feel free to create a new OU called `Students` just for fun by right-clicking the `THM` OU and selecting **New**, then **Organization Unit**.

If you open any OUs, you can see the users they contain and perform simple tasks like creating, deleting, or modifying them as needed. You can also reset passwords if needed (pretty useful for the helpdesk):

You probably noticed already that there are other default containers apart from the `THM` OU. These containers are created by Windows automatically and contain the following:

- **Builtin:** Contains default groups available to any Windows host.
- **Computers:** Any machine joining the network will be put here by default. You can move them if needed.
- **Domain Controllers:** Default OU that contains the DCs in your network.
- **Users:** Default users and groups that apply to a domain-wide context.
- **Managed Service Accounts:** Holds accounts used by services in your Windows domain.

### Security Groups vs OUs

You are probably wondering why we have both groups and OUs. While both are used to classify users and computers, their purposes are entirely different:

- **OUs** are handy for **applying policies** to users and computers, including specific configurations that apply to sets of users based on their roles in the enterprise. Remember, a user can only be a member of a single OU at a time, as it wouldn't make sense to apply two different sets of policies to a single user.
- **Security Groups**, on the other hand, are used to **grant permissions over resources**. For example, you can use groups to allow some users to access a shared folder or a network printer. A user can be a part of many groups, which is needed to grant access to multiple resources.

# Authentication Methods

When using Windows domains, all credentials are stored in the Domain Controllers. Whenever a user tries to authenticate to a service using domain credentials, the service must ask the Domain Controller to verify that the credentials are correct. Two protocols can be used for network authentication in Windows domains:

- **Kerberos:** Used by any recent version of Windows. This is the default protocol in any recent domain.
- **NetNTLM:** Legacy authentication protocol kept for compatibility purposes.

While NetNTLM should be considered obsolete, most networks will have both protocols enabled. Let's take a deeper look at how each of these protocols works.

## Kerberos Authentication

Kerberos authentication is the default authentication protocol in recent versions of Windows. Users who log into a service using Kerberos will be assigned tickets. Think of tickets as proof of a previous authentication. Users with tickets can present them to a service to demonstrate that they have already authenticated into the network and are therefore authorised to use it.

When Kerberos is used for authentication, the following process happens:

1. The user sends their username and a timestamp encrypted with a key derived from their password to the **Key Distribution Center (KDC)**, a service usually installed on the Domain Controller that issues Kerberos tickets on the network.  
      
    The KDC will create and send back a **Ticket Granting Ticket (TGT)**, which will allow the user to request additional tickets to access specific services. The need for a ticket to get more tickets may sound weird, but it allows users to request service tickets without having to enter their credentials every time they want to connect to a service. Along with the TGT, a **Session Key** is provided to the user, which they will need to generate for subsequent requests.  
      
    Notice that the TGT is encrypted using the **krbtgt** account's password hash, so the user can't access its contents. It is essential to know that the encrypted TGT includes a copy of the Session Key as part of its contents, and the KDC does not need to store the Session Key, as it can recover a copy by decrypting the TGT if needed.

2. When a user wants to connect to a service on the network, like a share, website or database, they will use their TGT to request a **Ticket Granting Service (TGS)** ticket from the KDC. TGS are tickets that allow connection only to the specific service they were created for. To request a TGS, the user will send their username and a timestamp, encrypted with the Session Key, along with the TGT and a **Service Principal Name (SPN)** indicating the service and server name we intend to access.  
  
	As a result, the KDC will send us a TGS along with a **Service Session Key**, which we will need to authenticate to the service we want to access. The TGS is encrypted using a key derived from the **Service Owner Hash**. The Service Owner is the user or machine account that the service runs under. The TGS contains a copy of the Service Session Key in its encrypted contents, so the Service Owner can access it by decrypting the TGS.

3. The TGS can then be sent to the desired service to authenticate and establish a connection. The service will use its configured account's password hash to decrypt the TGS and validate the Service Session Key.
## NetNTLM Authentication

NetNTLM works using a challenge-response mechanism. The entire process is as follows:
1. The client sends an authentication request to the server they want to access.
2. The server generates a random number and sends it as a challenge to the client.
3. The client combines their NTLM password hash with the challenge (and other known data) to generate a response to the challenge and sends it back to the server for verification.
4. The server forwards the challenge and the response to the Domain Controller for verification.
5. The domain controller uses the challenge to recalculate the response and compares it to the original response sent by the client. If they both match, the client is authenticated; otherwise, access is denied. The authentication result is sent back to the server.
6. The server forwards the authentication result to the client.

Note that the user's password (or hash) is never transmitted through the network for security.

  
![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)  **Note:** The described process applies when using a domain account. If a local account is used, the server can verify the response to the challenge itself without requiring interaction with the domain controller, since it already has the password hash stored locally in its SAM.