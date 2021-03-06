AAA on a Cisco IOS includes the ability to authorize and log every CLI command executed on the router. 
Unfortunately, the AAA command accounting only supports TACACS+ 
as the AAA transport protocol, making it unusable in environments using RADIUS.

You can use Embedded Event Manager as a workaround. The following configuration commands will log every command 
executed on the router. 

```
event manager applet CLIaccounting
event cli pattern ".*" sync no skip no
action 1.0 syslog priority informational msg "$_cli_msg"
set 2.0 _exit_status 1
```
The log messages generated by this EEM applet have the following format:
```
%HA_EM-6-LOG: CLIaccounting: command
```
As the EEM uses standard IOS logging mechanisms, you can use the show logging command to examine the command 
execution history or store the messages on a syslog server.
