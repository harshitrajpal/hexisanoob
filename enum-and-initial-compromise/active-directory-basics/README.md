# Active Directory - Basics

Active Directory allows network administrators to create and manage domains, users, and objects within a network. For example, an admin can create a group of users and give them specific access privileges to certain directories on the server. As a network grows, Active Directory provides a way to organize a large number of users into logical groups and subgroups, while providing access control at each level.

The Active Directory structure includes three main tiers: 1) domains, 2) trees, and 3) forests. Several objects (users or devices) that all use the same database may be grouped in to a single domain. Multiple domains can be combined into a single group called a tree. Multiple trees may be grouped into a collection called a forest. Each one of these levels can be assigned specific access rights and communication privileges.

Main concepts of an Active Directory:

1. **Directory** – Contains all the information about the objects of the Active directory
2. **Object** – An object references almost anything inside the directory (a user, group, shared folder...)
3. **Domain** – The objects of the directory are contained inside the domain. Inside a "forest" more than one domain can exist and each of them will have their own objects collection.&#x20;
4. **Tree** – Group of domains with the same root. Example: _dom.local, email.dom.local, www.dom.local_
5. **Forest** – The forest is the highest level of the organization hierarchy and is composed by a group of trees. The trees are connected by trust relationships.

Active Directory provides several different services, which fall under the umbrella of "Active Directory Domain Services," or AD DS. These services include:

1. **Domain Services** – stores centralized data and manages communication between users and domains; includes login authentication and search functionality
2. **Certificate Services** – creates, distributes, and manages secure certificates
3. **Lightweight Directory Services** – supports directory-enabled applications using the open (LDAP) protocol
4. **Directory Federation Services** – provides single-sign-on (SSO) to authenticate a user in multiple web applications in a single session
5. **Rights Management** – protects copyrighted information by preventing unauthorized use and distribution of digital content
6. **DNS Service** – Used to resolve domain names.

AD DS is included with Windows Server (including Windows Server 10) and is designed to manage client systems. While systems running the regular version of Windows do not have the administrative features of AD DS, they do support Active Directory. This means any Windows computer can connect to a Windows workgroup, provided the user has the correct login credentials.

![](<../../.gitbook/assets/image (29).png>)
