# **PowerTools API Emulator**
#### **Christian Contreras | Dell PowerTools Intern**
---



## Introduction
##### **Use PowerTools API Emulator (PT-APIE) to emulate instances of PowerTools Agent (PTAgent) on virtual hosts.**
---
#
#
#
## Getting Started | Use PT-APIE Now ***(Tested with centOS 7.0)***
### External Applications
##### **PT-APIE requires two external programs installed and working to run:**
##### -- Python 2.7.5
###### >> Download Python 2.7.5: https://www.python.org/downloads/release/python-275/
##### -- MIMIC Web Simulator Suite Version 19.00 | Gambit Communications
###### >> MIMIC Web Simulator Suite Version 19.00 Evaluation Download: https://www.gambitcomm.com/site/evaluation.php
###### >>> Setup instructions: MIMIC.md
##### ---
### Setup
Follow these instructions to get PT-APIE running:
1. Extract PT-APIE.tar.gz

    `tar -xzvf PT-APIE.tar.gz`
#####
2. Specify MIMIC external library

    `Open PT-APIE/lib/MIMIC.cfg and define "MIMICRoot" with the root directory of your MIMIC installation`
    `> e.g. "usr/local/mimic19.10"`
#####
3. Execute PT-APIE

    `python src/PT-APIE.py`
#####
---
#
#
#
## Emulating PTAgent Cluster Environments
### Creating a PTAgent Server
##### Configuration Files
PT-APIE asks for a `config` upon session startup. Configuration files are stored in `cfg` in `.json` format.
The key-value data in `config.json` files configures the environment PT-APIE will emulate:
* > `"logLevel"` : The logging level for this session | `TRACE`, `DEBUG`, `INFO` (default), `WARNING`, `ERROR`, or `CRITICAL`
* > `"logDir"` : The logging directory for this session | Default value is `/var/log/PT-APIE`
* > `"hosts"` : The virtual hosts we want to emulate and their corresponding PTAgent servers:
    > * > `"ip"` : The IP address to host the PTAgent simulation on | Default value increments from `1.99.0.1`
    > *  > `"port"` : The port to host the PTAgent simulation on | Default value is `80`
    > *  > `"responsePkg"` : Set of static responses (see below) to override the ones specified by the `rule file` | Default value is `null`
    > *  > `"customHandler"` : `.mtcl` or "`MIMIC` `.tcl`" script handler (see below) to override the one specified by the `rule file`| Default value is `null`
    > *  > `"ruleFile"` : Advanced only | See below
    > *  > `"restore"` : Set `"rulefile"`, `"customHandler"`, and `"responsePkg"` to that of an untouched, fully-functional PTAgent server | `True` or `False` (default)

To configure a PTAgent server, simply define its host entry.
##### Running the Server
1. Define a `.json config file` with a host entry and save it in `cfg` as described above
2. Execute PT-APIE

    `python src/PT-APIE.py`
#####
3. Type in the name of your `.json config file` when prompted (excluding the `.json` extension) | Leave blank for default config `cfg.json`
4. PT-APIE will begin setting up the server and inform you of progress:
```python
    PT-APIE: now attempt to set up host "10.147.1.65aaa:9090aaa"...
    PT-APIE: configure ip "10.147.1.65aaa" fails... using default ip list: 1.99.0.1
    PT-APIE: port "9090aaa" invalid... using default port 80 instead...
    PT-APIE: * * SUCCESS-- host is running! * *
```
* An invalid `ip` defaults to `1.99.0.1` incrementing | An invalid port defaults to `80`
* Any invalid server data defaults to that of a working PTAgent simulation

*As long as you define a host entry with its `port` and `ip` fields, you should have a PTAgent server running in your emulation.* ***Congratulations!***
```python
    PT-APIE: ----- HOSTS ----------
    PT-APIE: ----- '1.99.0.1:80': 1,
```
* The `value` following the `ip:port` `key` is the `agent` number of that host in MIMIC *(see below)*

##### Communicating with the Server
Use `Postman` and `PT Agent 1.8.4 - REST api Collection` to send PTAgent API requests to your server @ `ip:port`:
* Send requests to `http://{{agent_uri}}...`, **not** `https://{{agent_uri}}...`
* Set `{{agent_uri}}` to `ip:port`
* Set `Authorization` to `Basic auth`
    * Username: `root`
    * Password: `Passw0rd!`
* Specify `Content-Length` header **(case-sensitive!)** accurately

cURL representation of `host/SMF/inventory` request:
```javascript
    curl -X POST \
    http://1.99.0.1:80/api/PT/v1/host/SMF/inventory \
    -H 'Accept: */*' \
    -H 'Authorization: Basic cm9vdDpQYXNzdzByZCE=' \
    -H 'Connection: keep-alive'
    -H 'Content-Length: 2' \
    -H 'Content-Type: application/json' \
    -H 'Host: 100.80.92.100:8086' \
    -H 'User-Agent: PostmanRuntime/7.15.0' \
    -H 'accept-encoding: gzip, deflate' \
    -d '{}'
```
```javascript
    {
        "MessageId": "PTA_SMF_3009",
        "Message": "Attempted to run with no payload installed.",
        "error": "Attempted to run with no payload installed.",
        "agentstatus": "NO PAYLOAD INSTALLED"
    }
```
##### ---
### Running Multiple Servers
**Running multiple servers is extremely easy.**
1. Define multiple hosts through multiple host entries.
    * That's it. You're done! You have as many servers as you define host entries.
```python
    PT-APIE: ----- HOSTS ----------
    PT-APIE: ----- '1.99.0.1:80': 1,
    PT-APIE: ----- HOSTS ----------
    PT-APIE: ----- '1.99.0.2:80': 2,
    PT-APIE: ----- HOSTS ----------
    PT-APIE: ----- '1.99.0.3:80': 3,
```
So just like that, you have set up a cluster environment of PTAgent server emulations. Test it out with some API requests to the different servers!
##### ---
### Session Profiles
###### **Every PT-APIE session is automatically saved within `data/<ip>,<port>/api` and `data/<ip>,<port>/handler.mtcl`.**
###### You can restore individual hosts within a session to the default PTAgent simulation of a fully-functional server, in the `config file` host entry:
###### `"restore": true` | Default server data is held in `data/restore`
---
#
#
#
## Customizing PTAgent Servers
* ##### Understanding how the `MIMIC Web Simulator` works could be a good idea first *(see below)*
### Degraded Responses
##### Inidividually Degrading Responses
**Static Responses:**
* You can find the default degraded static responses in `data/restore/api`.
* The degraded response you need will be under that API request's URI, marked `error.html`
* Copy this response over to `data/<ip>,<port>/load` **without loss of format**
    * Include all parent directories up to `api/...`
    * Rename as the response file you want to replace

Follow this procedure for all degraded static responses. During setup, PT-APIE will load the specified `responsePkg` (or default one). Afterwards, it will merge `data/<ip>,<port>/api` with the contents of `data/<ip>,<port>/load` (priority).

**Dynamic Responses:**
* You can find the default degraded dynamic responses within `data/restore/handler.mtcl`.
* The degraded response you need will be under the `proc` named after that API request's URI, labeled `## Error case:` and encapsulated in quotes `""`.
* ***In a newly-duplicated handler.mtcl*** under this `proc`, just `set` `retval` to the degraded response and `return` `$retval`

```tcl
    proc POST_API_PT_V1_HOST_SMF_INVENTORY_HTTP_1_1_1 {} {
        set retval "
        "\{    \"Error\": \"Invalid\"\}"
        "
        return $retval
    }
```
* Copy this new `.mtcl` handler over to `data/<ip>,<port>/prof`
* Specify your handler name (minus the `.mtcl` extension) in your `config file` under the host entry's `customHandler`

Follow this procedure for all degraded dynamic responses. During setup, PT-APIE will load the specified `responsePkg`.
##### Degrade All Responses
This shortcut utilizes `data/restore/degraded` to automatically create and assume a fully-degraded profile for the desired host entry:

`"ruleFile": "data/restore/degraded/degraded.rul"`

Along with a degraded `ruleFile`, a corresponding degraded `responsePkg` and `customHandler` will be copied over and loaded upon server startup.
##### ---
### Custom Responses
* ##### Having a minimal grasp of `tcl/tk` could be a good idea first *(see below)*
* ##### Use of `regexp` in `tcl/tk` offers much more complete solutions *(see below)*
##### Static and Dynamic Responses
* We already know how to edit both static and dynamic responses within default data response behavior
    * You can inject in whatever data you want during those edits!
* To define your own custom `.py` python script as a response, use it as you `set` `retval`

    `set retval [exec python $::gPrivateDir/Python/responseScript.py]`
    * You must set the environment variable `MIMIC_IGNORE_SIGCHLD` to `1` for a `.py` to function 
##### MIMIC Web Simulator Suite
`MIMIC Web Simulator` is a powerful simulation tool which utilizes proprietary `rule files` and  `tcl/tk` programming to identify API requests and script responses.

*Terminology:*
* `.rul` `rule file`
API responses are foremost defined by `rules` in `MIMIC`
* `.mtcl` script handler
Rules can redirect API requests to a `.mtcl`, or "`MIMIC` `.tcl`", script handler

*The roadmap for a REST API call (i.e. `host/SMF/inventory`) is as follows:*
* Request is read by the specified `.rul` `rule file`
    * `GET` requests tend to contain an entire `request_header`
    	* In `MIMIC` `rules`, a `request_header` can be defined to immediately return a `response_file`
    	* This implementation is what we see for PTAgent static requests
    * `POST` and `PUT` requests will not contain a `request_header` and are instead identified by their `request_type` (`POST`/`PUT`) and directed towards the `.mtcl` script handler
* Request is read by the `rule`-specified `.mtcl::proc` in the `response_header_script` field (i.e. `*.mtcl::common_handler`)
    * The `proc` `common_handler` by default handles all `POST`/`PUT` requests
        * Individual responses are embedded within the `.mtcl` script handler, each as a `proc`
    * After checking against WSMan (`[check_WSMan]`), `identify_proc` is incurred
        * `::gCurrentHeader` contains the request URI and `content-type` header
        * `.json` requests are trimmed down i.e. `POST/api/PT/v1/host/SMF/inventory.json HTTP/1.1` -> `POST_API_PT_V1_HOST_SMF_INVENTORY_HTTP_1_1` via `regexp`. This is the same naming style as the embedded responses.
        * The resulting request, `message`, is compared against `proclist`, which contains each embedded response as a `proc`
        * The matching `proc` is  saved as an element of `identify_proc`, `proc_name`
    * The matching `proc` executes under `proc_name` and returns to `common_handler` as `ret0`
    (`set ret0 [string trim [$proc_name]]`)
    * `.json` requests are met with the generic passthrough response
    * `ret0`, specifying the correct `proc` for the request, is appended to the response and the response is returned
    * This implementation is what we see for PTAgent dynamic requests

*You should now have quite a bit of control over the customization and even scratch creation of your emulations just by editing a `.rul` or `.mtcl`.* ***Here is further help for implementing within `MIMIC` on your own:***

A `rule` in the `rule file` has different request/response pair parameters
* `request` identifies a request body
* `request_header` identifies a request header
* `request_type` identifies a request type (`GET`, `POST`, etc.)
* `response` hardcodes a response to return
* `response_file` identifies a file's contents to return as a response
* `response_script` specifies a `.mtcl` script to return a response
* `response_header_script` specifies a `.mtcl` script to return a response
> `request_header` is for non-SOAP HTTP commands and will only pair with `response_file` and `response_header_script`

Dynamic state is maintained in `.mtcl` scripts through `mimic agent store get/set/exists` of `store` variables
* Observe this example of the blinking LED server dynamic, encapsulated by `POST host/led/status` and `PUT host/led` (with a `blink` request field of `0` or `1`):
```tcl
proc POST_API_PT_V1_HOST_LED_STATUS_HTTP_1_1_1 {} {
    set retval "
    \{    \"LED State\": \"Off\"\}
    "
    if { [mimic agent store exists led_status] == 1 && [mimic agent store get led_status] == 1 } {
    set retval "
    \{    \"LED State\": \"Blinking\"\}
    "
    }
    return $retval
}
 
proc PUT_API_PT_V1_HOST_LED_HTTP_1_1_2 {} {
    set retval "
    \{    \"Result\": \"success\"\}
    "
   
    if { [regexp "^\\s*{\\s*\"request\":\\s*{\\s*\"blink\": \"1\"\\s*}\\s*}\\s*$" $::gCurrentRequest] == 1 } {
        mimic agent store set led_status 1
    } elseif { [regexp "^\\s*{\\s*\"request\":\\s*{\\s*\"blink\": \"0\"\\s*}\\s*}\\s*$" $::gCurrentRequest] == 1 } {
        mimic agent store set led_status 0
    } else {
        set retval "\{    \"Error\": \"Invalid\"\}"
    }
   
    return $retval
}
```
`proc POST_API_PT_V1_HOST_LED_STATUS_HTTP_1_1_1 {}`
```tcl
    set retval "
    \{    \"LED State\": \"Off\"\}
    "
```
* The default value for the server LED is `Off`
```tcl
    if { [mimic agent store exists led_status] == 1 && [mimic agent store get led_status] == 1 } {
    set retval "
    \{    \"LED State\": \"Blinking\"\}
    "
    }
```
* Determine the state of `store` variable `led_status`
	* If `led_status` exists (has been set before) *and* is true, the server LED is `Blinking`
    * Therefore `led_status` is akin to the `blink` request field

`proc PUT_API_PT_V1_HOST_LED_HTTP_1_1_2 {}`
```tcl
    set retval "
    \{    \"Result\": \"success\"\}
    "
```
* By default the `PUT` request to `blink` or `unblink` a server LED is successful
```tcl
    if { [regexp "^\\s*{\\s*\"request\":\\s*{\\s*\"blink\": \"1\"\\s*}\\s*}\\s*$" $::gCurrentRequest] == 1 } {
        mimic agent store set led_status 1
    } elseif { [regexp "^\\s*{\\s*\"request\":\\s*{\\s*\"blink\": \"0\"\\s*}\\s*}\\s*$" $::gCurrentRequest] == 1 } {
        mimic agent store set led_status 0
    }
```
* `$::gCurrentRequest` is the `global` for the current request's contents
* `regexp` carefully takes care of variable spacing and formatting within requests as well as accurate exclusion, though this is not needed
	* `[regexp <pattern> $::gCurrentRequest]` returns `1` upon a successful `regexp` of `<pattern>` within `$::gCurrentRequest`
* Check for the correctly-defined value of `blink` alone
* `"blink": "1"` and `"blink": "0"` are valid requests
	* Set `store` variable `led_status` to `blink` if valid
    * Otherwise, `led_status` stays unset (does not exist)
```tcl
else {
        set retval "\{    \"Error\": \"Invalid\"\}"
    }
```
* If `blink` was invalid or nonexistent then the request was unsuccessful

`MIMIC` and any `agent` all have several additional useful programmabilities through MIMICShell
```tcl
mimic agent assign <agent number>: Defines this agent as the universal agent in the MIMICShell
mimic agent start/stop/pause/halt/resume
mimic agent ipalias add/delete/start/stop/list
mimic agent store: Store variables which are held throughout the session, individual to each agent
mimic store: Universal session store variables which are held for the duration of the session
mimic value info/list
mimic timer script add/delete/list
mimic trap config add/delete/list
```
* You can make any of this part of the scripted behavior
	* Cause hosts to behave strangely or dump valuable information!
    * Commands also available within the `MIMIC` Python API, referenceable by any `.py` script

Record and convert your own conversations into responses. If you can pick out the response you can inject it!

Worst case -- resort to a `.py` scripted response. Even file I/O should be seamless through both `tcl/tk` and `Python`.

***For further assistance in configuring `MIMIC` refer to the attached MIMIC Web Simulator Suite Documentation: MIMIC.md***

---
###
###
###
## Debugging
### Logging
There are five log levels: `TRACE, DEBUG, INFO, WARNING, ERROR, CRITICAL`

You can find log outputs for PT-APIE here:

`/var/log/PT-APIE/visor.log`
#####
##### MIMIC Web Simulator
Large portions of output are funnelled through the logs of `MIMIC`.

You can find log outputs for your `MIMIC` session in background here:

`/var/log/PT-APIE/hosts.log`
#####
Navigate to `<mimic>/config/debug.cfg` and insert `postcallback.cc` for extensive debug logging within `MIMIC`.
* *You won't be able to functionally convert conversations with this line unless it is commented out*

You can always go into `MIMICView` or `MIMICShell` itself in the background and configure yourself in real-time! (Remember: MIMIC.md)
***
###
###
###
## Reference
### Program Structure
* > `cfg`: Holds `.json`-formatted `config files`
* > `data`: Contains default PTAgent server data and workspaces for virtual hosts
* > `doc`: Contains the PT-APIE documentation, including this document
* > `lib`: Points to library for external dependencies (`MIMIC Web Simulator`)
* > `src`: Executables for launching PT-APIE
##### ---
### `MIMIC Simulator`

`MIMIC` runs the virtual hosts you provide on the specified `ip` and `port` given per host entry. Each host is known within `MIMIC` as an `agent` -- the primary interface object for deploying, customizing, and controlling web simulations.

***Refer to: MIMIC.md***

***Official MIMIC 16.20 Documentation by Gambit Communications: https://www.gambitcomm.com/your1620/mimic1620.pdf***

##### ---
### `tcl/tk`
*`tcl/tk` is a lightweight and simplified scripting language*
* Quick syntax/conventions guide for `tcl/tk`: https://www.tcl.tk/man/tcl8.4/TclCmd/Tcl.htm

Common:
```tcl
.tcl: A script consisting of a string of commands
command: Phrases of multiple words formatted to be evaluated
word: The different parts of a command, separated by whitespace
set <parameter> <value>: Defines the key "parameter" with value "value"
append <parameter> <value>: Adds "value" on top of the existing contents of "parameter"
return <value>
if <condition> <value>...elseif <condition> <value>...else <value>
regexp/regsub
global
proc: "Procedure"; similar to a function in object-oriented programming, but wired to serve one defined purpose to fit tcl/tk form
* Enclosing a value within square brackets [] identifies it as a proc execution ("command substitution") *
* The result of the execution can be captured with variable substitution by calling the word using a leading $ *
```
Global variables in `tcl/tk`-derived `.mtcl` ("`MIMIC` `.tcl`") scripts:
```tcl
gCurrentAgent
gCurrentRequest
gSharedDir
gOwner
gPrivateDir
```
`regexp` allows for concise and precise search matching of specific patterns
* `tcl/tk` simple guide to `regexp`: https://www.tutorialspoint.com/tcl-tk/tcl_regular_expressions.htm

`File I/O`
* `tcl/tk` simple guide to file access: https://www.tcl.tk/man/tcl8.5/tutorial/Tcl24.html
##### ---
### FAQ
Future
Plans
Errors
Clarifications
Credit

**Thank you!**
