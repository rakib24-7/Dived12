
name: Free RDP

on: [push]

jobs:
  build:
    runs-on: windows-latest
    steps:
    - name: Create RDP User
      run: |
        net user RDPUser Maren123 /add
        net localgroup administrators RDPUser /add
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

    - name: Enable Firewall for RDP
      run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

    - name: Download Ngrok
      run: |
        Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
        Expand-Archive ngrok.zip
        move ngrok.exe C:\ngrok.exe

    - name: Start Ngrok Tunnel
      run: |
        C:\ngrok.exe authtoken 2zlDzvMAIkGqKZt0Zb5oMazYmYC_5vByizeJ3eQQXkmXdMuca
        Start-Process "C:\ngrok.exe" 'tcp 3389' -WindowStyle Hidden

    - name: Show RDP Address
      run: |
        Start-Sleep -Seconds 20
        Invoke-RestMethod http://127.0.0.1:4040/api/tunnels > tunnels.json
        $url = (Get-Content tunnels.json | ConvertFrom-Json).tunnels[0].public_url
        Write-Output "n📡 Your RDP address:n$url"
