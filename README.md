# Bash script for Check MK using REST API

It can use different Check MK sites and users when working with one server.
Script was tested using user "automation" with secret key (authentication set to "Automation secret for machine accounts").

Supported:
* Get information about Check MK id (hostname entry)
* Get information about Check MK services of host
* Get information about problematic Check MK services of host
* Create new id in specific folder and set DCHP or static IPv4
* Perform discovery on id using one of the many discovery types: 'new', 'remove', 'fix_all', 'refresh' or 'only_host_labels'
* Activate changes on site with the possibility of forcing foreign user changes
* Delete id
* List folders and ids in specific folder
* List only folder or only ids
* Set or change tags

For better view of json, pipe the output of this command to jq (command line JSON processor). \
You can pipe secret to stdin of this process to avoid entering secret key every time. \
Don't end paths with extra /. For example "/folder1" is ok, while "/folder1/" is not. Internally ~ is used instead of /.

## Examples:
Check MK site "test" used in this example, has the following hosts and folders: \
/host1 on DHCP \
/host2 on IPv4 "192.168.1.1" \
/folder1/subfolder1/ \
/folder1/subfolder2/ \
/folder1/host3 on IPv4 "192.168.1.2" \
/folder2/ 


## Example: Get info about id (hostname entry)
Pipe secret key to checkmk.sh and get information about id "host1" from site "test" using user "automation".
```
$ cat ~/.sec/secret | ./checkmk.sh get test automation host1
...
json output
...
```

We can use jq for json parsing:
```
$ cat ~/.sec/secret | ./checkmk.sh get test automation host1 | jq .
```

Get id, title, folder (location) and ipv4 for "host1":
```
$ cat ~/.sec/secret | ./checkmk.sh get test automation host1 | jq .ok.id,.ok.title,.ok.extensions.folder,.ok.extensions.attributes.ipaddress
"host1"
"host1"
""
null
```

Get id, title, folder (location) and ipv4 for "host3":
```
$ cat ~/.sec/secret | ./checkmk.sh get test automation host3 | jq .ok.id,.ok.title,.ok.extensions.folder,.ok.extensions.attributes.ipaddress
"host3"
"host3"
"folder1"
"192.168.1.2"
```

Don't forget to secure secret file with chmod 600 ~/.sec/secret.

## Example: List folders and ids in specific folder
List displays "null" for IPv4 if dhcp is used.
```
[user@alma checkmk]$ cat ~/.sec/secret | ./checkmk.sh ls test automation /
d /folder1                                 folder1
d /folder2                                 folder2
f host1                                    host1 (ip: null)
f host2                                    host2 (ip: 192.168.1.1)
[user@alma checkmk]$ 
```

```
[user@alma checkmk]$ cat ~/.sec/secret | ./checkmk.sh ls test automation /folder1
d /folder1/subfolder1                      subfolder1
d /folder1/subfolder2                      subfolder2
f host3                                    host3 (ip: 192.168.1.2)
[user@alma checkmk]$ 
```

```
[user@alma checkmk]$ cat ~/.sec/secret | ./checkmk.sh ls test automation /folder1/subfolder1
[user@alma checkmk]$
```

You can also use commands lsf or lsd to only output ids or folders.

## Example: Create new id in specific folder and set DCHP or static IPv4

Create id "newhost1" in "/folder1/subfolder1" on site "test" using user "automation". It uses DHCP as IP is not specified.

```
$ cat ~/.sec/secret | ./checkmk.sh create test automation newhost1 /folder1/subfolder1
...

$ cat ~/.sec/secret | ./checkmk.sh ls test automation /folder1/subfolder1
f newhost1                                 newhost1 (ip: null)
```

Create id "newhost2" using static ip "192.168.1.3" in "/folder1/subfolder2" on site "test" using user "automation":
```
$ cat ~/.sec/secret | ./checkmk.sh create test automation newhost2 /folder1/subfolder2 192.168.1.3
...

$ cat ~/.sec/secret | ./checkmk.sh ls test automation /folder1/subfolder2
f newhost2                                 newhost2 (ip: 192.168.1.3)
```

Remember to not add extra / at the end of path.

## Example: Delete id

Delete id "host1":
```
$ cat ~/.sec/secret | ./checkmk.sh delete test automation host1
{"ok" : "No Content: Operation done successfully. No further output." }
```
Notice how ls does not show "host1" in "/" anymore:
```
$ cat ~/.sec/secret | ./checkmk.sh ls test automation /
d /folder1                                 folder1
d /folder2                                 folder2
f host2                                    host2 (ip: 192.168.1.1)
```


## Example: Activate changes on site

Activate changes on site "test" even if there are changes made by other users.
```
$ cat ~/.sec/secret | ./checkmk.sh activate test automation true
```
If you use false or skip the force parameter, the command will activate user changes only if there are no changes made by other users (waiting to be activated).



## Example: Perform discovery on id
Discovery can be performed on id using one of the many discovery types: 'new', 'remove', 'fix_all', 'refresh' or 'only_host_labels'

Execute "fix_all" discovery on id "host2":
```
$ cat ~/.sec/secret | ./checkmk.sh discover test automation host2 fix_all
```

## Example: Set tag
Set or change tags.

First find the etag of host2:
```
$ cat ~/.sec/secret | ./checkmk.sh get test automation host2 | jq .etag
"552cfcd33a80e98d516a03efaebfaeb03fc3830d50a8b1fb27ec473c29f32b1a"
```

Then set tag on host2: set tag value "https" from tag group "tag_tcp_test"
!! Tag group names start with "tag_" prefix!
!! If we create tag group in Check MK WEB GUI with "Tag group ID" set to "servis",
!! it will be named "tag_servis" here!
```
$ cat ~/.sec/secret | ./checkmk.sh settag test automation host2 "552cfcd33a80e98d516a03efaebfaeb03fc3830d50a8b1fb27ec473c29f32b1a" tag_tcp_test https
```

## Example: Get information about services of host
```
$ cat ~/.sec/secret | ./checkmk.sh services test automation host1 | jq .
```

## Example: Get information about problematic services of host
```
$ cat ~/.sec/secret | ./checkmk.sh service_problems test automation host1 | jq .
```
