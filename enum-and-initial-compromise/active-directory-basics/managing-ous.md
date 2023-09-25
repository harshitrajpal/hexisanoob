# Managing OUs

### Deletion

1. Delete users. Go to Active Directory Users and Computers
2. View-> Advanced Features
3. Choose OU->Properties->Objects->Uncheck Accidental deletion
4. Then right click on OU and delete

### Delegation

One of the nice things you can do in AD is to give specific users some control over some OUs. This process is known as **delegation** and allows you to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in.

One of the most common use cases for this is granting `IT support` the privileges to reset other low-privilege users' passwords. According to our organisational chart, Phillip is in charge of IT support, so we'd probably want to delegate the control of resetting passwords over the Sales, Marketing and Management OUs to him.

1. Choose OU->Delegate Control
2. Add users and groups to whom delegation is supposed to happen
3. Choose features (like reset password on next logon

### Powershell command to change password

Where sophie is the desired user whose password will be changed.

```shell-session
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```

### Powershell command to make a user change password on next logon

```shell-session
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```



### Creating New OUs

When we go to computers in AD DS, we see laptops, PCs and servers. We can organize them neatly by creating separate OUs.

Choose domain->new->OU

We can now shift computers to respective OUs.

Choose multiple computers->right click->move->choose OUs.

<figure><img src="../../.gitbook/assets/image (5) (1) (2).png" alt=""><figcaption></figcaption></figure>
