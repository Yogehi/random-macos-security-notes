## Summary

Requirements: A 'device profile' with the setting 'limit profile installation' cannot be installed on the machine.

Profiles can be used to slightly modify how a MacOS computer will behave. There are two types of profiles; User and Device.

User profiles will only affect the user that the profile was assigned to. The assigned user also has to be logged in before the User profile's settings will take affect.

System profiles will affect all users of the MacOS device, whether or not the user is logged in. This includes the root user.

All profiles are stored in the `/private/var/db/ConfigurationProfiles/Store` directory, but this directory cannot be accessed while the MacOS device is running, even by the root user. Therefore, there are two commands that you should familiarize yourself with: `profiles` and `dscl`.

## "profiles" command

The `profiles` binary is located at `/usr/bin`. This binary can be used to enumerate, import and remove installed profiles, but most of the fun stuff can only be done via `dscl`.

`profiles` help command below:

```
yaymacyay:~ lowPrivUser$ profiles help
profiles allows you to add or remove configuration or application provisioning profiles on macOS.
	Use 'profiles help' for this help section, or use the man page for expanded instructions.
	Basic usage is in the form:  'profiles <command verb> [<options and parameters>]'
 
    Command Verbs:
                    status - indicates if profiles are installed
                    list - list profile information
                    show - show expanded profile information
                    install - install profile
                    remove - remove profile
                    sync - synchronize installed configuration profiles with known users
                    renew - renew configuration profile installed certificate
                    version - display tool version number
 
    Options:    (not all options are meaningful for a command)
                    -type=<string> - type of profile; either 'configuration', 'provisioning', 'enrollment', or 'startup'
                    -user=<string> - short user name
                    -identifier=<string> - profile identifier
                    -path=<string> - file path
                    -uuid=<string> - profile UUID
                    -enrolledUser=<string> - enrolled user name
                    -verbose - enable verbose mode
                    -forced - when removing profiles, automatically confirms requests
                    -all - select all profiles
                    -quiet - enable quiet mode
```

Some example commands are below that can be ran as an unprivileged user:

Enumerate device installed profiles: `profiles -C`

```
yaymacyay:~ lowPrivUser$ profiles -C
_computerlevel[1] attribute: profileIdentifier: Yays-Mac.BBBBBBBB-BBBB-438F-BBBB-B4278DB8BBBB
There are 1 system configuration profiles installed
```

Enumerate user installed profiles: `profiles list`

```
yaymacyay:~ lowPrivUser$ profiles list
lowPrivUser[1] attribute: profileIdentifier: Yays-Mac.AAAAAAAA-AAAA-438F-AAAA-B4278DB8AAAA
There are 1 user configuration profiles installed for 'lowPrivUser'
```

Note: it is not possible to enumerate other users' installed profiles with the `profiles` command unless you are running the command with `sudo` privileges

Enumerate all installed profiles: `sudo profiles list` or `sudo profiles -P`

Install a profile and do not require a computer restart to have the profile take affect: `profiles install -path <string>` or `profiles -I -F <string>`

Install a profile and have it require a computer restart to have the profile take affect: `profiles -s -F <string>`

Install a profile as a device level profile: `sudo profiles install -path <string>` or `sudo profiles -I -F <string>`

Install a profile on a remote Mac machine: `profiles -I -p <Mac location>`

## "dscl" command

The other tool when working with installed profiles on a MacOS device. The cool part about this command is that you can enumerate profiles that have been installed and assigned to other users. So for example, what if "UserB" had their identity cert installed via a User profile.

`dscl` help command, in relation to profile management

```
yaymacyay:~ lowPrivUser$ dscl . help
<snipped>
MCX Profile Extensions:
    -profileimport    <record path> <profile file path>
    -profiledelete    <record path> <profile specifier>
    -profilelist      <record path> [optArgs]
    -profileexport    <record path> <profile specifier> <output folder path>
    -profilehelp
```
Example commands below. Any command that requires elevated privileges will have a `sudo` at the beginning of the command.

List profiles installed on a user, including other users: `dscl . profilelist /Users/<username>`

Example output:
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

Export a specific profile on a specific user, including other users: `dscl . profileexport /Users/<username> <Profile Identifier> <Destination Folder>`

Example output:
```
yaymacyay:~ lowPrivUser$ dscl . profileexport /Users/adminUser Yays-Mac.D46888F6-98AC-438F-B1A1-B4278DB8696A ~/Desktop
Exporting to: /Users/lowPrivUser/Desktop
  Exported 8029CF1F-EFB4-49F8-8CC5-B40951C744B1 as: exampleProfile.mobileconfig
```

Export all profiles on a specific user, including other users: `dscl . profileexport /Users/<username> all <Destination Folder>