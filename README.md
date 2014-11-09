Falco: security intelligence for software maintainers

Falco is a project to find known latent bugs in 3rd party software packages by
refering to CVE CPE entries from the National Vulnerability Database(NVD). We wrote the tool because there were no accessible tools for developers and project maintainers to easily find known security vulnerabilities in software they use as part of a project.

Falco does not test the software, it simply looks to see that the package and version you tell it are in a vulnerability database.  If they are in the database, you have a vulnerability in software you depend upon which you need to respond to by updating your software.

Mark Menkhus, RedCup working group
menkhus@icloud.com

July 11, 2014

All rights reserved.

Copyright 2014 Mark Menkhus, Glynn Mitchell

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

usage: falco [-h] [-b] [-d [VFEED_DATABASE]] [--debug] [-f [PACKAGELISTFILE]]
             [-i [ITEMS_REPORTED]] [-n [PACKAGE_NAME]] [-v [PACKAGE_VERSION]]
             [-V]

Checks command line or, a file list of software programs for known security
defects documented in the National Vulnerability Database. Matches a project
name and version name to CPE URIs in the NVD database.

optional arguments:
  -h, --help            show this help message and exit  
  -b, --build_environment
                        for use in build environments, return fail if items
                        found  
  -d [VFEED_DATABASE], --vfeed_database [VFEED_DATABASE]
                        location of vfeed.db sqlite database from vfeed
                        project  
  --debug               turn on debug output  
  -f [PACKAGELISTFILE], --packagelistfile [PACKAGELISTFILE]  
                        file where the list of packages to evaluate is stored  
  -i [ITEMS_REPORTED], --items_reported [ITEMS_REPORTED]   
                        number of items reported for NVD/CVE matches  
  -n [PACKAGE_NAME], --package_name [PACKAGE_NAME]    
                        package name to search for  
  -v [PACKAGE_VERSION], --package_version [PACKAGE_VERSION]  
                        package version to look for  
  -V, --Version         report the version of falco and exit  

Examples:  
Assumes vfeed.db is in the same directory as falco  

Example 1, check a package named 'python' version '2.7.3' for vulnerabilities in the NVD database:  
$ ./falco.py -n python -v 2.7.3  
One Item being checked  
        *** Potential security defect found in python:2.7.3  
CVE: CVE-2013-7040  
CVSS Score: 4.3  
CPE id: cpe:/a:python:python:2.7.3  
Published on:             2014-05-19T10:55:09.987-04:00  
Summary Description: Python 2.7 before 3.4 only uses the last eight bits of the prefix to randomize hash values, which causes it to compute hash values without restricting the ability to trigger hash collisions predictably and makes it easier for context-dependent attackers to cause a denial of service (CPU consumption) via crafted input to an application that maintains a hash table.  NOTE: this vulnerability exists because of an incomplete fix for CVE-2012-1150.  
  
Example 2, using falco in build situations:  
check a package named 'python' version '2.7.3' for vulnerabilities in the NVD database and if any are found, return a non zero return value.  Placing this in a makefile will cause make to exit:  
$ ./falco.py -b -n python -v 2.7.3 >> falcolog  
$ echo $?  
1  
$  
  
Example 3, falco in a makefile, using the -b (break the build) flag:  
Makefile:  
# this will FAIL make because this version 1.14.7 bash is found in the NVD 
# database.
# Solution is twofold: to update bash to most recent code, and 
# to change the # make entry to reflect that new version number:
bash.build.out:  
    ./falco.py -b -n bash  -v 1.14.7 -d ./vfeed.db > bash.build.out  
clean:  
    rm bash.build.out  
  
Execution:  
$ make  
./falco.py -b -n bash  -v 1.14.7 -d ./vfeed.db > bash.build.out  
make: *** [bash.build.out] Error 1  
  
Explanation, -b cause falco to return fail if any information is returned from searching NVD for the package and version.  
  
Suggested action:  
File a bug in the bug tracker, change the makefile, remove -b, and when the bug is fixed, update the makefile again to reflect the latest package number and reinstate the -b   
  
Dependencies: Falco uses the NVD database from the vfeed project:   https://github.com/toolswatch/vFeed  
1) Download the vfeed package and use the update to gather the nvd database in sqlite3 form.   
    Ex: /vfeedcli.py update  
2) write down where you put the vfeed.db file, and store that for future use, falco uses that as an argument.   
3) Note it would be a good idea to put this "vfeedcli.py update" in a cron job, since falco counts on using updated NVD data to see when new vulnerabilities exist.  
  
bug reports:  
send bug reports to menkhus@icloud.com  