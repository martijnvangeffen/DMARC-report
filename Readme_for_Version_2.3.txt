To install the dmarc report tool make sure the prerequisit software is present:

1: Install exchange managed API for EWS 2.0 or newer. 
2: have a version of the free 7zip installed or use the attached one. 
3: save the script on a server with connection to the exchange servers EWS directory and access to Internet DNS information zones ( needed to translate source IP`s ). 
4: Optional Create PowerBI Endpoint see word document

Run the script sample:
get-help .\Create-DMARCreport.ps1
get-help .\Create-DMARCreport.ps1 -full
get-help .\Create-DMARCreport.ps1 -examples

Todo:
* Investigate if real report date can be preserved, now using the date the report is downloaded

Full version history notes 
Version Changes:

    Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
    24-11-2017            V0.3         Martijn van Geffen    Initial Alpha script
    02-12-2017            V0.4         Martijn van Geffen    Updated code and bugs
    04-12-2017            V0.5         Martijn van Geffen    Added help system
                                                             Added sub function help system
                                                             Added support for dynamic TopXresults in per issue tables 
                                                             (default is still 15)
                                                             Fix no name sub table for empty value records in subfunctions
                                                             Redone functions with more solid code 
                                                             Added verbose and debug to functions
                                                             Fixed typo in culumnname variable
    06-12-2017            V0.6         Martijn van Geffen    Added Secure password functionality
                                                             Added HTML hover over highlights
                                                             Tweaked HTML code and CSS
                                                             Improved DNS lookup table
    11-12-2017            V0.7         Martijn van Geffen    Added new per domain tabbed reports
                                                             Improved code and code comment
    12-12-2017            V1.0         Martijn van Geffen    Fixed issue with locating 7 zip executable
                                                             Changed switch to delete attachments into delete item
                                                             Fixed issue where item dit not get deleted from mailbox
                                                             Added a static variable to controle where deleted item goes
                                                             Added some more verbose comment
                                                             Updated some HTML code for better alignment
                                                             Released to technet gallery
    14-02-2018            V1.3         Martijn van Geffen    Fixed issues:
                                                                Fixed a issue when a shared mailbox was used the processed folder
                                                                could not be created
                                                                Fixed multiple issue where a shared mailbox was used but the script
                                                                was targeting the credential users mailbox
                                                                Fixed a issue when a reverse hostname resolve would return also NS servers
                                                                This would result in System.objects[] in the report tables
                                                                Fixed a issue when a reverse hostname resolve would result in a wrong 
                                                                object type export
                                                             New Features:
                                                                 Added support for NON Rua mail to be moved to a NoDMARCrua folder. This 
                                                                 enhances performance if RUF is also targeted at the same mailbox or other mail
                                                                 got into the dmarc folder.
                                                                 Added in and export functionality for the IP reverse resolve table with a date
                                                                 of expiration on the file. ( default 30 days )
                                                             HTML changes:
                                                                Removed the double border of tables
                                                                Added zebra coding in tables
                                                                Change colours to make it nicer to watch the tables
                                                                Added padding for information tables
 12-03-2018            V2.0        Martijn van Geffen    Fixed issues:
                                                                IP table sometimes truncated the hostname to only 1 character.
                                                                Export of IP table did not export incremental causing errors of duplicate keys in 
                                                                hash table.
                                                            New Features:
                                                                Added possibility to add GEO IP lookups using "-geolookupenabled" switch. If the
                                                                switch is used the script will add a additional table with the sources per country.
                                                                Additionaly the country, city, lattitude and longitude will also be send to
                                                                PowerBI if PowerBI switch is enabled.
                                                                Added per domain per country reports to zoom in on countrys.
                                                                Added Power BI integration with the switch "-powerbiuploadenabled". If the switch
                                                                is used the script will do a incremental upload with the powerBI API. This makes it
                                                                possible to create rich reports in PowerBI.  
                                                                Added possibility to add report on failed items only using the "-DMARCfailedonly"
                                                                switch. Only items failing  both alignment checks will be in the report.                                                               
                                                            HTML changes:
                                                                Added Reports total of DMARC passed VS DMARC failed.
                                                                Added tables for DMARC alignment of SPF.
                                                                Added tables for DMARC alignment of DKIM.
20-04-2018            V2.1        Martijn van Geffen    Fixed issues:
                                                                Fix filter logic for failed only parameter
                                                            New Features:
								Added new parameter DMARCfailedonly to generate reports on items failing DMARC only
								Added new parameter DMARCpassedonly to generate reports on items passing DMARC only
								Added Bypass parameter for office 365 autodiscover
02-05-2018            V2.2        Martijn van Geffen    Fixed issues:
                                                                $Geocounter allways remained above 9950 due to not being cleared during the wait
                                                                Fix a issue with DMARC domain being created as a systemobject[] in the reports
								Updated 7 zip to lates version due to vulnerability
07-11-2018            V2.3        Martijn van Geffen    Fixed issues:
                                                                Switched GEO IP provider as the old one depricated.
                                                                Fixed a test-path error on empty variable when first run or first run of the month
                                                                Updated the version of 7ZIP included due to a vulnerability in the old version

                                                