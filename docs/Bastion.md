



5. **Configure Entra ID sign-in for VMs**
   
   As metioned above, both Linux and Windows VMs were initially deployed using traditional authentication methods. The Linux VMs used SSH key authentication, while the Windows VM used a local administrator account and password.
   
   To improve security , user experience and simplify identity management, we change the authentication method to Entra ID

#### Linux VM Entra ID Sign-in Requirements

```
To use Entra ID authentication for Linux VM sign-in, the following requirements must be met.
```

| Area             | Requirement                                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| VM Identity      | **System assigned managed identity** must be enabled on the Linux VM.                                                                       |
| VM Extension     | The `AADSSHLoginForLinux` extension must be installed.                                                                                      |
| Extension Status | The extension status must show **succeeded**. If the status is `Installing`, `Transitioning` or `Failed`, Entra ID SSH login will not work. |
| RBAC Role        | The Entra user must be assigned role of either `Virtual Machine Administrator Login` or `Virtual Machine User Login`.                       |

- Install Login Extension for Linux VM
  
  vm-hr-linux1→settings→ Extensions + applications→ Add→ Azure AD Based SSH Login

![](../screenshots/8linux.jpg)

Owner or Contributor permission does not automatically grant operating system login access to the Linux VM. The VM login role must be assigned separately.

In this lab, `Virtual Machine Administrator Login` is assigned to the lab administrator account for testing.
