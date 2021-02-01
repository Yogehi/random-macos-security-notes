## Scheduled Tasks (see `cronjobs.md` and `launchd.md`)

## Profiles (see `profiles.md`)

Sometimes profiles can contain sensitive information about the specific users. Use `dscl . profileexport` to extract specific profiles from the machine.

Enumerate profiles for a local admin user:
```
yaymacyay:~ lowPrivUser$ dscl . profilelist /Users/adminUser
Profile 1:
                    Kind: Configuration
          Attribute UUID: 8029CF1F-EFB4-49F8-8CC5-B40951C744B1
        Import file name: exampleProfile.mobileconfig
            Profile name: Untitled
            Profile UUID: FC085B28-0849-456F-8CA2-42AB05F77CD6
      Profile Identifier: Yays-Mac.D46888F6-98AC-438F-B1A1-B4278DB8696A
            Profile size: 4162
    Attribute value size: 3416
```

Export a specific profile:
```
yaymacyay:~ lowPrivUser$ dscl . profileexport /Users/adminUser Yays-Mac.D46888F6-98AC-438F-B1A1-B4278DB8696A ~/Desktop
Exporting to: /Users/lowPrivUser/Desktop
  Exported 8029CF1F-EFB4-49F8-8CC5-B40951C744B1 as: exampleProfile.mobileconfig
```

Under the correct domain setting conditions, you could also tell `dscl` to enumerate/extract profiles from other Mac machines:

Use the username `destinationUsername` and the password `destinationPassword` to list profiles on the host `192.168.210.50` that belong to the same user:
```
yaymacyay:~ lowPrivUser$ dscl -u destinationUsername -p destinationPassword 192.168.210.50 profilelist /LDAPv3/127.0.0.1/Users/destinationUsername
```

Use the local MacBook to connect to another MacBook, with the same creds as the local MacBook user, and list profiles on another MacBook:
```
yaymacyay:~ lowPrivUser$ dscl /LDAPv3/targetHostName profilelist /Users/targetUser
```