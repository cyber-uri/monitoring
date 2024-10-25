# monitoring

#Instalar servicio SNMP Windows
#Comprueba si es un Windows Server o un Windows Client
#$os = (Get-CimInstance -ClassName Win32_OperatingSystem).Caption
$comprobar_so = (Get-WmiObject Win32_OperatingSystem | Select-Object Caption, Version).Caption


if ($comprobar_so -like '*Windows Server 2008*') {
  Import-Module ServerManager
  Get-WindowsFeature *SNMP* | Add-WindowsFeature
} elseif ($comprobar_so -like '*Windows Server*') {
  Get-WindowsFeature -Name "*SNMP*" | Install-WindowsFeature
}
else {
  Add-WindowsCapability -Online -Name "SNMP.Client~~~~0.0.1.0"
}


$comprobar_si_snmp_esta_instalado = (Get-Service -Name SNMP).Status
if ($comprobar_si_snmp_esta_instalado -eq "Running") {
   #Borramos configuración anterior del regedit


   Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\PermittedManagers" -Name *
   Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities" -Name *

  #Configurar comunidad y IP
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\PermittedManagers /v 1 /t REG_SZ /d 10.110.70.47 /f
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities /v public /t REG_DWORD /d 0x00000004 /f

  #Configurar comunidad y IP
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\PermittedManagers /v 1 /t REG_SZ /d 10.0.3.148 /f
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities /v public /t REG_DWORD /d 0x00000004 /f


  #Configurar comunidad y IP
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\PermittedManagers /v 1 /t REG_SZ /d 10.130.49.132 /f
  reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities /v public /t REG_DWORD /d 0x00000004 /f


   #Abrir puertos para ping y SNMP
   netsh advfirewall firewall add rule name="ICMP Allow In" protocol=icmpv4:8,any dir=in action=allow
   netsh advfirewall firewall add rule name="SNMP" dir=in action=allow protocol=UDP localport=161


   #Inicio de SNMP automático al arrancar, y reiniciamos servicio
   Set-Service -Name SNMP -StartupType Automatic
   Start-Service -Name SNMP
   Restart-Service -Name SNMP
   exit 0
} else {
   exit 1
}
