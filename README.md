## Setting Up a Windows Client for Ansible Management

To set up a Windows client ready to be used via Ansible, follow these steps:

### 1. **Installing OpenSSH Server and Starting the Service**

   - **Using Chocolatey:**
     ```powershell
     choco install --package-parameters=/SSHServerFeature openssh
     ```

   - **Using Windows Features:**
     Navigate to Control Panel -> Programs -> Turn Windows Features on or off -> Select "OpenSSH Server" -> Click OK.

   - **Starting the service:**
     ```powershell
     Start-Service sshd
     ```

   - **Setting the service to run on startup:**
     ```powershell
     Set-Service -Name sshd -StartupType 'Automatic'
     ```

### 2. **Allowing SSH Access through Firewall**

   - **PowerShell Command:**
     ```powershell
     New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
     ```

### 3. **Creating SSH Key Pair and Copying the Public Key**

   - **Creating SSH Key Pair:**
     ```bash
     ssh-keygen -t rsa -b 4096
     ```

   - **Copying the Public Key:**
     ```bash
     ssh-copy-id <username>@<host_ip_address>
     ```

### 4. **Running the Playbook**

   - **Example Playbook:**
     ```yaml
     ---
     - name: Check Connectivity
       hosts: windows
       tasks:
         - name: Ping Test
           win_ping:
     ```


## Ansible Windows Performance Optimization

This summary provides insights on optimizing Windows performance for Ansible management based on the [official documentation](https://docs.ansible.com/ansible/latest/os_guide/windows_performance.html).

### 1. **Optimize PowerShell Performance**
   Reduce Ansible task overhead by optimizing PowerShell startup time using the provided PowerShell snippet. This optimization employs the native image generator, `ngen`, to create native images for the assemblies PowerShell relies on, improving the startup time by approximately 10x.

```powershell
function Optimize-PowershellAssemblies {
  # NGEN powershell assembly, improves startup time of powershell by 10x
  $old_path = $env:path
  try {
    $env:path = [Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory()
    [AppDomain]::CurrentDomain.GetAssemblies() | % {
      if (! $_.location) {continue}
      $Name = Split-Path $_.location -leaf
      if ($Name.startswith("Microsoft.PowerShell.")) {
        Write-Progress -Activity "Native Image Installation" -Status "$name"
        ngen install $_.location | % {"`t$_"}
      }
    }
  } finally {
    $env:path = $old_path
  }
}
Optimize-PowershellAssemblies
