# SA-rsearch
Sample custom search commands for Splunk to allow limited and controlled access to restricted indexes and lookups.

## Warnings
The code in this app provides a mechanism for bypassing access controls in Splunk. 

* If you are inexperienced with Splunk or if you cannot understand exactly what every line of the included code is doing, you should not install this app. 
* You should carefully analyze this code and perform a risk assessment before installing this app on a production system. 

**Beware that this app stores Splunk usernames and passwords in plain text in a configuration file in the filesystem of the Splunk search head.** This is generally considered a bad practice. If this violates your policies, standards, or compliance mandates, you should not install this app.


*No guarantees are made about the efficacy of this app. Use at your own risk.*


## Installation
```
$ cd $SPLUNK_HOME/etc/apps
$ git clone https://github.com/daveherrald/SA-rsearch.git
$ $SPLUNK_HOME/bin/splunk restart
```

## Configuration
Edit $SPLUNK_HOME/etc/apps/SA-rsearch/bin/rsearch.config to configure a strong password.
```
[rsearch]
USER = privileged_splunk_user
PASS = <put a strong password here, and ditch the angle brackets>
HOST = 127.0.0.1
PORT = 8089
```

Next create the Splunk user. Make sure the username and password match what is specified in the config file above. The user should be granted only the privileged_reader role. 

NOTE: the privileged_reader role is defined by this app when it is installed.

## Try it out
For these sceanrios, create a simple Splunk user called 'luser' with the role of 'user'. This user will mimic a non-privileged user who you want to be able to access a restricted resource. NOTE: This is a different user than the one specified above.

### Scenario 1 - Read a restricted lookup table
In this scenario, we will allow the unprivileged user access to **only certain rows** of a restricted lookup table called employeedata.csv (included with this app). 

employeedata.csv contains the following sensitive records:

```
user,salary,employee_rating
mallory,100000,4
bob,75000,3
alice,200000,5
luser,20000,2
```

Log in as the unprivileged user(luser) and attempt to read the lookup. Splunk will report 'No results found...' as expected.

![](https://github.com/daveherrald/SA-rsearch/raw/master/images/failed-inputlookup.png "Logo Title Text 1")

Now, as the same unprivileged user(luser), run the custom search command called rinputlookup, and observe the restricted results.

![](https://github.com/daveherrald/SA-rsearch/raw/master/images/rinputlookup.png "Logo Title Text 1")

## Customize

Edit the search in the file $SPLUNK_HOME/etc/apps/SA-rsearch/bin/rinputlookup.py. The search appears as follows:

```
    user = self._metadata.searchinfo.username
    searchquery = """
    | inputlookup employeeinfo.csv | search user={}
    """.format(user)
```

## Yep it's kludgey!
We know! Again, use at your own risk.



