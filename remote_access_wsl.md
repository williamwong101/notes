# Access WSL From Remote Machine in LAN

## Intro

- This simple tutorial documents the steps to access WSL from a remote machine, say a Macbook, under same LAN.
- You can host a Windows machine as server, and access its resources(CPU, GPU, Memory, Network bandwidth) in WSL from a remote machine in LAN. This allows you to develop on Mac with a more powerful Linux environment, without the need to rent a AWS machine.

## Steps

1. Install WSL2 on Windows machine
   1. There are plenty of tutorial on this, assume it is done.
2. Check Windows machine local IP, and assign static IP for it.
   1. Open PowerShell in windows, use `ipconfig` to inspect IP info. Under LAN section, check `IPv4 Address`, this will be your current IP address.
      1. `Subnet Mask` is used to identify network prefix and Host identifier in your IP. Bitwise AND of the `Subnet Mask` and your IP is network prefix, while the other part is Host identifier.
      2. `Default Gateway` will be the IP of your router.
   2. Use `Default Gateway` to access your Router Manager, and assign a static IP to your Windows Host
      1. This is important, as Router uses DHCP to assign a new IP to your Windows upon each connection.
      2. We want a fixed IP to be used in SSH
3. Ping your IP address to confirm connectivity
   1. If ping fails, enter `Windows Defender Firewall with Advanced Security` > `Inbound Rules` > `File and Printer Sharing(Echo Request - ICMPv4-In)` enable this rule. This will allow `ping` traffic, which uses ICMP protocal.
4. Install SSH Server in Windows, and enable auto startup
   1. `Settings` > `Optional features` > `Add an optional feature` > `OpenSSH Server`
   2. `Services` > `OpenSSH SSH Server` > `Startup type = Automatic`
5. SSH connect to Windows and configure SSH keys
   1. First try connect to your Windows with the same password as in Windows user: `ssh <windows_username>@<windows_ip>`
   2. Then send your SSH public-key to Windows:

        ```bash
        authorizedKey=$(echo .ssh/id_ed25519.pub);

        remotePowershell="powershell New-Item -Force -ItemType Directory -Path \$env:USERPROFILE\.ssh; Add-Content -Force -Path \$env:USERPROFILE\.ssh\authorized_keys -Value '$authorizedKey'";

        ssh <windows_username>@<windows_ip> $remotePowershell
        ```

   3. Save this login info in .ssh/config

        ```shell
        Host host_name
            HostName windows_ip
            User windows_username
            IdentityFile ~/.ssh/id_ed25519

        ```

   4. Try ssh connection: `ssh host_name`
6. Connect Your VSCode on Mac Remotely to Windows
   1. Set connection env to be Linux and it will automatically invoke the WSL env

## Refs

- <https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui#install-openssh-for-windows>
- <https://superuser.com/questions/627208/unable-to-ping-a-windows-machine-from-linux>
- <https://www.linksys.com/support-article?articleNum=137818>
- <https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement#deploying-the-public-key>
- <https://linuxize.com/post/using-the-ssh-config-file/>
- <https://gist.github.com/adamelliotfields/16dfac1bacf6d2eeada0582fdfbbb7b6>
