<#
.SYNOPSIS
    http://tech-savvy.nl DMARC reporting script. 
    This script will function as a end to end report tool to pull DMARC reports from a maibox and
    Create nice HTML reports with GEO data that can be directly publisched in a IIS server.  
.DESCRIPTION
    DMARC Reporting tool by Tech-savvy.nl

    This tool will utilised EWS to connect to the mailbox used by DMARC reporting.
    Pull the reports and convert them into nice readable HTML reports. Reports can be enhanced with 
    GEO location data. The Raw data can also be uploaded to PowerBI to create rich reports. The script
    supports incremental data upload to the PowerBI API.
    The tool is compatible with multiple file formats used by DMARC report senders. 

    The following table will outline the steps in more detail:
        1. Connect to the reporting mailbox
        2. Downloads the report attachments
        3. Move the processed item to different folder
        4. Move non DMARC items to a different folder
        5. Unzip the downloaded attachment
        6. Load all the xml data and rearrange the data into a usable csv database
        7. if failed or passed only param is present filter the data
        8. Create disk structure for per month reporting
        9. Create a master html report containing data of last 6 months
       10. Create sub html report per running month
       11. If GEO is enabled enhance the reports with GEO country location information
       12. If GEO is enabled Create country specific reports
       13. If PowerBI is enable upload a incremental dataset to PowerBI API
       14. HTML can be saved directly in a IIS published folder to have end to end automated reporting process

.EXAMPLE
    .\Create-DMARCreport.ps1 -dmarcmailbox "scriptkiddie@tech-savvy.nl" -dmarcmailboxpassword "1234"
    
    This sample will start the reporting generation process using the minimal needed switches. 
    It will use all the default folders and assumes 7zip is present in it. the reports in the mailbox are in
    the default "Inbox" folder.

        -dmarcmailbox "scriptkiddie@tech-savvy.nl"
            The DMARC mailbox to pull the reports from ( This will also be used as the username )

        -dmarcmailboxpassword "1234"
            The DMARC mailbox password

.EXAMPLE
    .\Create-DMARCreport.ps1 -generatepasswordfile

    start the script in secure password generation mode. Note when creating the password its only valid in the
    user context and on the computer wher is was create. So create it under the user that the scheduled task will
    run under.

.EXAMPLE
    .\Create-DMARCreport.ps1 -dmarcmailbox "scriptkiddie@tech-savvy.nl" -dmarcmailboxusername "13th\script" -passwordfile "c:\mailboxpassword.bin" -DMARCpassedonly
    
    This sample will start the reporting generation process using minimal switches. 
    It will use all the default folders and assumes 7zip is present in it. the reports in the mailbox are in
    the default "Inbox" folder. It will filter and report only DMARC passed items.

        -dmarcmailbox "scriptkiddie@tech-savvy.nl"
            The DMARC mailbox to pull the reports from ( This will also be used as the username )
        
        -dmarcmailboxusername "13th\script"
            Use a different username than the mailbox emailadress as username for authentication

        -passwordfile .\mailboxpassword.bin
            Use the secure credential in the file as the password for the mailbox

        -DMARCpassedonly
            Only emails that passed DMARC checks will be included in the reports

.EXAMPLE
    .\Create-DMARCreport.ps1 -dmarcmailbox "scriptkiddie@tech-savvy.nl" -dmarcmailboxusername "13th\script" -passwordfile "c:\mailboxpassword.bin" -DMARCfailedonly
    
    This sample will start the reporting generation process using minimal switches. 
    It will use all the default folders and assumes 7zip is present in it. the reports in the mailbox are in
    the default "Inbox" folder. It will filter and report only DMARC failed items.

        -dmarcmailbox "scriptkiddie@tech-savvy.nl"
            The DMARC mailbox to pull the reports from ( This will also be used as the username )
        
        -dmarcmailboxusername "13th\script"
            Use a different username than the mailbox emailadress as username for authentication

        -passwordfile .\mailboxpassword.bin
            Use the secure credential in the file as the password for the mailbox

        -DMARCfailedonly
            Only emails that failed DMARC checks will be included in the reports

.EXAMPLE
    .\Create-DMARCreport.ps1 -ziplocation "C:\dmarcsite" -customsourcefolder "dmarc" -reportstoragedir "C:\dmarcsite"  -dmarcmailbox "scriptkiddie@tech-savvy.nl" -dmarcmailboxpassword "1234" -deleteprocesseditem -TopXresults 25 -iptabledaysold 10

   
    This sample will start the reporting generation process using the following switches.

        -ziplocation "C:\dmarcsite" 
            The 7zip executable is located in a custom folder called "C:\dmarcsite"
            Default value is "c:\dmarcworkfolder"

        -customsourcefolder "dmarc"
            The DMARC report are stored in the mailox in a separate folder under the inbox called "dmarc"      

        -reportstoragedir "C:\dmarcsite"
            The script will export the reports downloaded and generated in subfolders of the folder "C:\dmarcsite"
            Default value is "c:\dmarcworkfolder"

        -dmarcmailbox "scriptkiddie@tech-savvy.nl"
            The DMARC mailbox to pull the reports from ( This will also be used as the username )

        -dmarcmailboxpassword "1234"
            The DMARC mailbox password
            
        -deleteprocesseditem
            Delete items processed to keep source mailbox size lower. If you need to controle where they go
            change the static variable in the code under the static header
        
        -topXresults 25
            The sub tables will be limits to maximum top 25 results
            (The default value is 15)
        -iptabledaysold 10
            Flush the IP table if it is older than 10 days
            (The default value is 30) 

.EXAMPLE
    .\Create-DMARCreport.ps1 -ziplocation "C:\dmarcsite" -customsourcefolder "dmarc" -reportstoragedir "C:\dmarcsite"  -dmarcmailbox "scriptkiddie@tech-savvy.nl" -dmarcmailboxpassword "1234" -deleteprocesseditem -TopXresults 25 -iptabledaysold 10 -geolookupenabled -PowerBIuploadenabled

   
    This sample will start the reporting generation process using the following switches.

        -ziplocation "C:\dmarcsite" 
            The 7zip executable is located in a custom folder called "C:\dmarcsite"
            Default value is "c:\dmarcworkfolder"

        -customsourcefolder "dmarc"
            The DMARC report are stored in the mailox in a separate folder under the inbox called "dmarc"      

        -reportstoragedir "C:\dmarcsite"
            The script will export the reports downloaded and generated in subfolders of the folder "C:\dmarcsite"
            Default value is "c:\dmarcworkfolder"

        -dmarcmailbox "scriptkiddie@tech-savvy.nl"
            The DMARC mailbox to pull the reports from ( This will also be used as the username )

        -dmarcmailboxpassword "1234"
            The DMARC mailbox password
            
        -deleteprocesseditem
            Delete items processed to keep source mailbox size lower. If you need to controle where they go
            change the static variable in the code under the static header
        
        -topXresults 25
            The sub tables will be limits to maximum top 25 results
            (The default value is 15)
        -iptabledaysold 10
            Flush the IP table if it is older than 10 days
            (The default value is 30) 

        -geolookupenabled 
            Enable GEO module to resolve country location of the source IP
            This will also enable county specific reports
        
        -PowerBIuploadenabled
            Enable upload to PowerBI
            PowerBI endpoint should be configured in this script and in PowerBi

.INPUTS
    See full commandlet syntax
.OUTPUTS
    Direct HTML files no values are returned to the shell
.NOTES
    -----------------------------------------------------------------------------------------------------------------------------------
    Script name   : Create-DmarcReport.ps1
    Authors       : Martijn (Scriptkiddie) van Geffen
    Version       : 2.3
    dependancies  : EWS Managed API 2.0 or newer ( https://www.nuget.org/packages/Microsoft.Exchange.WebServices/ )
                    A version of open 7zip ( Included GNU LGPL license or via http://www.7-zip.org/ )
                    If PowerBI is used a PowerBI account with a Hybrid streaming API enabled as documented in the attached PowerBI document 
                    If Geolookup is enabled access to http://freegeoip.net
                    If PowerBI is enabled you need internet access to the PowerBI API

    -----------------------------------------------------------------------------------------------------------------------------------
    -----------------------------------------------------------------------------------------------------------------------------------
    Version Changes:
    Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
    12-12-2017            V1.0         Martijn van Geffen    Fixed issue with locating 7 zip executable
                                                             Changed switch to delete attachments into delete item
                                                             Fixed issue where item dit not get deleted from mailbox
                                                             Added a static variable to controle where deleted item goes
                                                             Added some more verbose comment
                                                             Updated some HTML code for better alignment
                                                             Released to technet gallery
    13-02-2018            V1.3         Martijn van Geffen    Fixed issues:
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
    22-02-2018            V1.4         Martijn van Geffen    Fixed issues:
                                                                Parameter binding set for -iptabledaysold changed: Added to password file set" 
                                                                Domain name in month reports was sometimes truncated to only the first character.
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
                                                                Added CSV files to the exception list of expandable types by 7 zip
    07-11-2018            V2.3        Martijn van Geffen    Fixed issues:
                                                                Switched GEO IP provider as the old one depricated.
                                                                Fixed a test-path error on empty variable when first run or first run of the month
                                                                Updated the version of 7ZIP included due to a vulnerability in the old version
    -----------------------------------------------------------------------------------------------------------------------------------

.COMPONENT
    This is the main script
.ROLE
    This is the main script
.FUNCTIONALITY
    Create DMARC reports in HTML pulling report sources from the DMARC mailbox
#>

##########################
#     Parameters         #
##########################

#region Parameters

[CmdletBinding(DefaultParameterSetName='Defaultscriptrun',
    SupportsShouldProcess=$true, 
    HelpUri = 'http://www.tech-savvy.nl'
)]
[OutputType()]

Param
(
    # Param1 The mailbox containing the reports
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]    
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$true)] 
    [ValidateNotNullOrEmpty()]
    [ValidatePattern("[@]")]
    [string]$dmarcmailbox,

    # Param2 the password for the mailbox containing the reports
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)] 
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [ValidateNotNullOrEmpty()]
    [string]$dmarcmailboxusername=$dmarcmailbox,

    # Param3 the password for the mailbox containing the reports
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]  
    [ValidateNotNullOrEmpty()]
    [string]$dmarcmailboxpassword,

    # Param4 The location to 7zip
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]   
    [ValidateNotNullOrEmpty()]
    [string]$ziplocation = "c:\dmarcworkfolder",

    # Param5 reports are not in the postvakin but in a other folder
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]   
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)] 
    [ValidateNotNullOrEmpty()]
    [string]$customsourcefolder,

    # Param6 The folder to store the reports in
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]   
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)] 
    [ValidateNotNullOrEmpty()]
    [string]$reportstoragedir = "c:\dmarcworkfolder",

    # Param7 Delete processed attachments from the work folder
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]   
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)] 
    [switch]$deleteprocesseditem,

    # Param8 Enable EWS tracing
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]   
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)] 
    [switch]$trace,

    # Param9 Limit the top X results to maximum of
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)] 
    [ValidateNotNullOrEmpty()]
    [int32]$topXresults=15,

    # Param10 Location of the encrypted passwordfile
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$true)]
    [ValidateScript({test-path -Path $_})]
    [string]$passwordfile,

    # Param11 Limit the top X results to maximum of
    [Parameter(ParameterSetName='GeneratePasswordfile',
        Mandatory=$true)]
    [ValidateNotNullOrEmpty()]
    [switch]$generatepasswordfile,

    # Param12 IP table date to remove if older
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [ValidateNotNullOrEmpty()]
    [int32]$iptabledaysold=30,

    # Param13 Enable GEO data lookup
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [switch]$geolookupenabled,

    # Param14 Enable upload to powerBI
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [switch]$PowerBIuploadenabled,

    # Param15 Create reports for DMARC failed only
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [switch]$DMARCfailedonly,

    # Param16 Create reports for DMARC passed only
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [switch]$DMARCpassedonly,

    # Param17 Skipp auutodiscover when using Office 365
    [Parameter(ParameterSetName='Defaultscriptrun',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Defaultscriptrunpassed',
        Mandatory=$false)]
    [Parameter(ParameterSetName='Passwordfile',
        Mandatory=$false)]
    [switch]$BypassautodiscoverforO365
)


#endregion Parameters

#############################
#    static configuration   #
#############################

#region staticconfiguration

$version= "2.3"

#this is the graph character
#9608= Blok , 164 = star 
[string]$barsign = [char]164
[string]$barempty = [char]164

#report range of days to report on
$reportrange = 180

#delete items from the mailbox to ( Can be HardDelete, softDelete or MoveToDeletedItems) 
$deletetoaction = "MoveToDeletedItems"

#powerBI endpoint 
#sample endpoint below
#$endpoint = "https://api.powerbi.com/beta/c7ba2aa0-618b-4041-854f-44643d718003/datasets/8f69ca58-9a7d-4817-b273-64875be5b7c9/rows?key=RzT%2BBgaU0sNF%2BWH8XhdfgdKTSlJEkoBspQGghdfghdNbG8NWBtlQy6HqjczmsXyyJqaS5b6xhKphgg64nAv%2F5sIgENdXI%2Fg%3D%3D"
$endpoint = ""

#endregion staticconfiguration

##########################
#     functions          #
##########################

#region functions

Function Generate-TableWithSubTable
{
    <#
    .SYNOPSIS
        Generate the DMARC HTML table code with subtables 
    .DESCRIPTION
        Generate the code used by the DMARC reporter script. The code will be dynamically generated based on the input.
        The code generated will be the master table including per issue subtable
    .EXAMPLE
        Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain
    
        Generate the main table with per issue subtables. Use the required switches.

            -name "Dmarc per domain"
                Name of the main table to be created
                
            -columnname "Dmarc domains"
                Name of the column from the Datatable to use as statistics count
                
            -table $grouptotaldataperdomain
                Grouped Data table to display the statistics of 
    .EXAMPLE
        Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain -topXresults 25
    
        Generate the main table with per issue subtables. Use the required switches.

            -name "Dmarc per domain"
                Name of the main table to be created
                
            -columnname "Dmarc domains"
                Name of the column from the Datatable to use as statistics count
                
            -table $grouptotaldataperdomain
                Grouped Data table to display the statistics of

            -topXresults 25
                The sub tables will be limits to maximum top 25 results
                (The default value is 15)
    .INPUTS
        [string]$name, [string]$columnname, [array]$table
    .OUTPUTS
        [System.Array]$tempbody
    .NOTES
        -----------------------------------------------------------------------------------------------------------------------------------
        Function name : Generate-TableWithSubTable
        Authors       : Martijn (Scriptkiddie) van Geffen 
        Version       : 1.0
        dependancies  : None
        -----------------------------------------------------------------------------------------------------------------------------------
        -----------------------------------------------------------------------------------------------------------------------------------
        Version Changes:
        Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
        05-12-2017            V1.0         Martijn van Geffen    Initial Function
                                                                 Added support for variable Top X results in the sub tables
        -----------------------------------------------------------------------------------------------------------------------------------
    .COMPONENT
        Create-DMARCreport.ps1
    .ROLE
        Create HTML code from the data processed by the main script
    .FUNCTIONALITY
        Create HTML code from the reordered data harvested by the main script
    #>

    [CmdletBinding()]
    [Alias()]
    [OutputType([System.Array])]
   
    Param
    (
        # Param1 Name of the table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$name,

        # Param2 Property to use from master table to generate table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$columnname,

        # Param3 Table input
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [array]$table,

        # Param4 Limit the top X results to maximum of
        [Parameter(Mandatory=$false)]
        [ValidateNotNullOrEmpty()]
        [int32]$topXresults=15
    )

    write-verbose -Message "In function Generate-TableWithSubTable"
    write-debug -Message "Running function for $name with column $columnname"

    $tempbody = @()

    #Generate HTML report Headers
    If ([string]::IsNullOrWhiteSpace($name))
    {
        $tempbody += "<H2>Main report: Blank field value</H2>" 
    }else
    {
        $tempbody += "<H2>Main report: $name</H2>" 
    }


    #Convert HTML code to table with Precent bar attached
    $htmltemp = $table | select @{name="$columnname";expression={$_.name}},
        @{name="number of records";expression={$_.count}},
        @{name="% part of total";expression={
            $bartotalobjects = ($table | Measure-Object -Property count -sum).sum
            $barpercent = (100 / $bartotalobjects) * $_.count
            $barleft = $barsign * ($barpercent )
            $barright = $barempty * ((100 - $barpercent) )
            "xopenFont color=greenxclose{0}xopen/FontxclosexopenFont color=blackxclose{1}%xopen/FontxclosexopenFont Color=blackxclose{2}xopen/fontxclose" -f $barleft,$barpercent.tostring("000"),$barright

        }
    } | ConvertTo-Html -Fragment 

    $htmltemp = $htmltemp -replace "xopen","<"
    $htmltemp = $htmltemp -replace "xclose",">"
    $htmltemp = $htmltemp -replace "<table>",'<table class="style1">'

    $tempbody += $htmltemp
    $tempbody += "<br>"

    write-debug -Message "Var tempbody HTML code before creating sub tables $tempbody"

    foreach ($status in $table)
    {
        #Clear itterative variables
        $15result = $null
        $htmltemp = $null
        $sourcedomain = $null

        #do the loop
        write-verbose -Message "In function Generate-TableWithSubTable - generating sub table status $status"

        $tempbody += "<br>"
        $tempbody += "<H3>Top $topXresults sources for object $($status.name)</H3>" 

        $15result = $status.group| Group-Object -Property sourceip | Sort-Object -Descending -Property count | Select-Object -First $topXresults

        $htmltemp = $15result | select @{name="count";expression={$_.count}},
            @{name="source IP";expression={$_.name}},
            @{name="Domain";expression={
                [array]$sourcedomain = $_.group.sourcedomain
                $sourcedomain[0]
            }
        } | ConvertTo-Html -Fragment 

        $htmltemp = $htmltemp -replace "xopen","<"
        $htmltemp = $htmltemp -replace "xclose",">"
        $htmltemp = $htmltemp -replace "<table>",'<table class="style2">'

        $tempbody += $htmltemp
        $tempbody += "<br>"
    }

    write-debug -Message "Var tempbody HTML code after creating subtables $tempbody"
    return $tempbody
}

Function Generate-TableWithSubTablecountry
{
    <#
    .SYNOPSIS
        Generate the DMARC HTML table code with subtables and hyperlinks in the country
    .DESCRIPTION
        Generate the code used by the DMARC reporter script. The code will be dynamically generated based on the input.
        The code generated will be the master table including per issue subtable
    .EXAMPLE
        Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain
    
        Generate the main table with per issue subtables. Use the required switches.

            -name "Dmarc per domain"
                Name of the main table to be created
                
            -columnname "Dmarc domains"
                Name of the column from the Datatable to use as statistics count
                
            -table $grouptotaldataperdomain
                Grouped Data table to display the statistics of 
    .EXAMPLE
        Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain -topXresults 25
    
        Generate the main table with per issue subtables. Use the required switches.

            -name "Dmarc per domain"
                Name of the main table to be created
                
            -columnname "Dmarc domains"
                Name of the column from the Datatable to use as statistics count
                
            -table $grouptotaldataperdomain
                Grouped Data table to display the statistics of

            -topXresults 25
                The sub tables will be limits to maximum top 25 results
                (The default value is 15)
    .INPUTS
        [string]$name, [string]$columnname, [array]$table
    .OUTPUTS
        [System.Array]$tempbody
    .NOTES
        -----------------------------------------------------------------------------------------------------------------------------------
        Function name : Generate-TableWithSubTable
        Authors       : Martijn (Scriptkiddie) van Geffen 
        Version       : 1.0
        dependancies  : None
        -----------------------------------------------------------------------------------------------------------------------------------
        -----------------------------------------------------------------------------------------------------------------------------------
        Version Changes:
        Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
        05-12-2017            V1.0         Martijn van Geffen    Initial Function
                                                                 Added support for variable Top X results in the sub tables
        -----------------------------------------------------------------------------------------------------------------------------------
    .COMPONENT
        Create-DMARCreport.ps1
    .ROLE
        Create HTML code from the data processed by the main script
    .FUNCTIONALITY
        Create HTML code from the reordered data harvested by the main script
    #>

    [CmdletBinding()]
    [Alias()]
    [OutputType([System.Array])]
   
    Param
    (
        # Param1 Name of the table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$name,

        # Param2 Property to use from master table to generate table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$columnname,

        # Param3 Table input
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [array]$table,

        # Param4 Limit the top X results to maximum of
        [Parameter(Mandatory=$false)]
        [ValidateNotNullOrEmpty()]
        [int32]$topXresults=15
    )

    write-verbose -Message "In function Generate-TableWithSubTable"
    write-debug -Message "Running function for $name with column $columnname"

    $tempbody = @()

    #Generate HTML report Headers
    If ([string]::IsNullOrWhiteSpace($name))
    {
        $tempbody += "<H2>Main report: Blank field value</H2>" 
    }else
    {
        $tempbody += "<H2>Main report: $name</H2>" 
    }


    #Convert HTML code to table with Precent bar attached
    $domainname = $table.group.dmarcdomain[0]
    $htmltemp = $table | select @{name="$columnname";expression={'xopena href=xqoute.\{0}_{1}.htmlxqoute target=xqoute_blankxqoutexclose{2}xopen/axclose' -f $domainname,$_.name,$_.name}},
        @{name="number of records";expression={$_.count}},
        @{name="% part of total";expression={
            $bartotalobjects = ($table | Measure-Object -Property count -sum).sum
            $barpercent = (100 / $bartotalobjects) * $_.count
            $barleft = $barsign * ($barpercent )
            $barright = $barempty * ((100 - $barpercent) )
            "xopenFont color=greenxclose{0}xopen/FontxclosexopenFont color=blackxclose{1}%xopen/FontxclosexopenFont Color=blackxclose{2}xopen/fontxclose" -f $barleft,$barpercent.tostring("000"),$barright

        }
    } | ConvertTo-Html -Fragment 

    $htmltemp = $htmltemp -replace "xqoute",'"'
    $htmltemp = $htmltemp -replace "xopen","<"
    $htmltemp = $htmltemp -replace "xclose",">"
    $htmltemp = $htmltemp -replace "<table>",'<table class="style1">'

    $tempbody += $htmltemp
    $tempbody += "<br>"

    write-debug -Message "Var tempbody HTML code before creating sub tables $tempbody"

    foreach ($status in $table)
    {
        #Clear itterative variables
        $15result = $null
        $htmltemp = $null
        $sourcedomain = $null

        #do the loop
        write-verbose -Message "In function Generate-TableWithSubTablecountry - generating sub table status $status"

        $tempbody += "<br>"
        $tempbody += @"
            <H3>Top $topXresults sources for object <a href=".\$($domainname)_$($status.name).html" target="_blank">$($status.name)</a></H3>
"@
        $15result = $status.group| Group-Object -Property sourceip | Sort-Object -Descending -Property count | Select-Object -First $topXresults

        $htmltemp = $15result | select @{name="count";expression={$_.count}},
            @{name="source IP";expression={$_.name}},
            @{name="Domain";expression={
                [array]$sourcedomain = $_.group.sourcedomain
                $sourcedomain[0]
            }
        } | ConvertTo-Html -Fragment 

        $htmltemp = $htmltemp -replace "xopen","<"
        $htmltemp = $htmltemp -replace "xclose",">"
        $htmltemp = $htmltemp -replace "<table>",'<table class="style2">'

        $tempbody += $htmltemp
        $tempbody += "<br>"
    }

    write-debug -Message "Var tempbody HTML code after creating subtables $tempbody country"
    return $tempbody
}

Function Generate-Table
{
    <#
    .SYNOPSIS
        Generate the DMARC HTML table code. 
    .DESCRIPTION
        Generate the code used by the DMARC reporter script. The code will be dynamically generated based on the input.
        The code generated will be the master table only.
    .EXAMPLE
        Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain
    
        Generate the main table. Use the required switches.

            -name "Dmarc per domain"
                Name of the main table to be created
                
            -columnname "Dmarc domains"
                Name of the column from the Datatable to use as statistics count
                
            -table $grouptotaldataperdomain
                Grouped Data table to display the statistics of 
    .INPUTS
        [string]$name, [string]$columnname, [array]$table
    .OUTPUTS
        [System.Array]$tempbody
    .NOTES
        -----------------------------------------------------------------------------------------------------------------------------------
        Function name : Generate-Table
        Authors       : Martijn (Scriptkiddie) van Geffen 
        Version       : 1.0
        dependancies  : None
        -----------------------------------------------------------------------------------------------------------------------------------
        -----------------------------------------------------------------------------------------------------------------------------------
        Version Changes:
        Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
        05-12-2017            V1.0         Martijn van Geffen    Initial Function
                                                                 
        -----------------------------------------------------------------------------------------------------------------------------------
    .COMPONENT
        Create-DMARCreport.ps1
    .ROLE
        Create HTML code from the data processed by the main script
    .FUNCTIONALITY
        Create HTML code from the reordered data harvested by the main script
    #>

    [CmdletBinding()]
    [Alias()]
    [OutputType()]
   
    Param
    (
        # Param1 Name of the table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$name,

        # Param2 Property to use from master table to generate table
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$columnname,

        # Param3 Table input
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [array]$table,
        
        # Param4 Enable Geo data
        [Parameter(Mandatory=$false)]
        [switch]$geolookupenabled
    )

    write-verbose -Message "In function Generate-Table"
    write-debug -Message "Running function for $name with column $columnname"

    $tempbody = @()

    #Generate HTML report Headers
    If ([string]::IsNullOrWhiteSpace($name))
    {
        $tempbody += "<H2>Additional Report Blank field value</H2>" 
    }else
    {
        $tempbody += "<H2>Additional Report: $name</H2>" 
    }

    #Convert HTML code to table with Precent bar attached
    if ($geolookupenabled.IsPresent)
    {
        $htmltemp = $table | select @{name="$columnname";expression={$_.name}},
            @{name="domainname";expression={
                [array]$tempdomains = $_.group.sourcedomain
                $tempdomains[0] 
            }},
            @{name="country";expression={
                [array]$tempcountry = $_.group.country_name
                $tempcountry[0]    
            }},
            @{name="number of records";expression={$_.count}},
            @{name="% part of total";expression={
                $bartotalobjects = ($table | Measure-Object -Property count -sum).sum
                $barpercent = (100 / $bartotalobjects) * $_.count
                $barleft = $barsign * ($barpercent )
                $barright = $barempty * ((100 - $barpercent) )
                "xopenFont color=bluexclose{0}xopen/FontxclosexopenFont color=blackxclose{1}%xopen/FontxclosexopenFont Color=blackxclose{2}xopen/fontxclose" -f $barleft,$barpercent.tostring("000"),$barright

            }
        } | ConvertTo-Html -Fragment 
    }else
    {
        $htmltemp = $table | select @{name="$columnname";expression={$_.name}},
            @{name="domainname";expression={
                [array]$tempdomains = $_.group.sourcedomain
                $tempdomains[0] 
            }},
            @{name="number of records";expression={$_.count}},
            @{name="% part of total";expression={
                $bartotalobjects = ($table | Measure-Object -Property count -sum).sum
                $barpercent = (100 / $bartotalobjects) * $_.count
                $barleft = $barsign * ($barpercent )
                $barright = $barempty * ((100 - $barpercent) )
                "xopenFont color=bluexclose{0}xopen/FontxclosexopenFont color=blackxclose{1}%xopen/FontxclosexopenFont Color=blackxclose{2}xopen/fontxclose" -f $barleft,$barpercent.tostring("000"),$barright

            }
        } | ConvertTo-Html -Fragment 
    }

    $htmltemp = $htmltemp -replace "xopen","<"
    $htmltemp = $htmltemp -replace "xclose",">"
    $htmltemp = $htmltemp -replace "<table>",'<table class="style1">'

    $tempbody += $htmltemp
    $tempbody += "<br>"

    return $tempbody

}

function Write-PowerBI
{
    <#
    .SYNOPSIS
        Write data to PowerBI API 
    .DESCRIPTION
        This function will write data to the PowerBI API. It take a array as input and pushes the data in JSON to the API
    .EXAMPLE
        Write-PowerBI -payload $payload.inputobject -endpoint $endpoint
    
        Write 1 entry to PowerBI API

            -Payload $payload.inputobject
                Use the array $payload.inputobject as payload in the JSON call
                
            -endpoint $endpoint
                Use the HTTP endpoint to push the data to. This endpoint can be found on the connector in POWERBI 
    .INPUTS
        [array]$payload, [string]$endpoint
    .OUTPUTS
        none
    .NOTES
        -----------------------------------------------------------------------------------------------------------------------------------
        Function name : Write-PowerBI
        Authors       : Martijn (Scriptkiddie) van Geffen 
        Version       : 1.0
        dependancies  : Internet access to PowerBI API
        -----------------------------------------------------------------------------------------------------------------------------------
        -----------------------------------------------------------------------------------------------------------------------------------
        Version Changes:
        Date: (dd-MM-YYYY)    Version:     Changed By:           Info:
        08-03-2018            V1.0         Martijn van Geffen    Initial Function
                                                                 
        -----------------------------------------------------------------------------------------------------------------------------------
    .COMPONENT
        Create-DMARCreport.ps1
    .ROLE
        Push PowerBI data to API
    .FUNCTIONALITY
        Push PowerBI data to API
    #>

    [CmdletBinding()]
    [Alias()]
    [OutputType()]

    param( 
    
        # Param1 Array with the data to push
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [array]$payload,
                
        # Param2 Endpoint URL to push to
        [Parameter(Mandatory=$true)]
        [ValidateNotNullOrEmpty()]
        [string]$endpoint
    
    )
    
    Invoke-RestMethod -Method Post -Uri "$endpoint" -Body (ConvertTo-Json @($payload))    
}

#endregion functions

##########################
#     HTML Stylesheet    #
##########################

#region HTML stylesheet

$stylesheet = @"

<style>

body { background-color:#f8f9f9;

       font-family:Tahoma;

       font-size:12pt; }

td, th { border:1px solid #000033; 

         border-collapse:collapse; }

th { color:white;

     background-color:#000033; }

table.style1, tr, td, th { padding: 10px; margin:0px }

table.style1 { margin-left:10px; border-collapse: collapse; }

table.style1 tr:nth-child(even) {background-color: #e6f2ff;} 

table.style1 tr:hover { background-color:#fff7e6; }

table.style2, tr, td, th { padding: 10px; margin-left:0px; }

table.style2 { margin-left:70px; border-collapse: collapse; }

table.style2 tr:nth-child(even) {background-color: #e6f2ff;}

table.style2 tr:hover { background-color:#fff7e6; }

sourceipiframe { padding: 22px; }

h2 { margin-left:10px; }

h3 { margin-left:70px; }

.tabs {
  position: relative;   
  min-height: 800px;
  clear: both;
  margin: 25px 0;
}

.tab {
  float: left;
}

.tab label {
  background: #eee; 
  padding: 10px; 
  border: 1px solid #ccc; 
  margin-left: -1px; 
  position: relative;
  left: 1px; 
}

.tab [type=radio] {
  display: none;   
}

.content {
  position: absolute;
  top: 28px;
  left: 0;
  background-color:#f8f9f9
  right: 0;
  bottom: 0;
  padding: 20px;
  border: 1px solid #ccc; 
}

[type=radio]:checked ~ label {
  background-color:#f8f9f9
  border-bottom: 1px solid white;
  z-index: 2;
}

[type=radio]:checked ~ label ~ .content {
  z-index: 1;
}

</style>

<Title>DMARC email security report by Tech-savvy.nl</Title>

<br>

"@ 

#endregion HTML stylesheet

##########################
#     main script        #
##########################

#region terminate whatif

if ([bool]$WhatIfPreference.IsPresent)
{
    Write-Output -InputObject "Whatif switch is not supported in this script"
    exit
}

#endregion terminate whatif

#region generate passwordfile

if ($generatepasswordfile.IsPresent)
{

    Write-Output -InputObject "Script called with -generatepassword switch.`n"
    
    [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") 
    [void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 

    $objForm = New-Object System.Windows.Forms.Form 
    $objForm.Text = "DMARC Secure Password file generator"
    $objForm.Size = New-Object System.Drawing.Size(400,300) 
    $objForm.StartPosition = "CenterScreen"

    $objForm.KeyPreview = $True

    $saveButton = New-Object System.Windows.Forms.Button
    $saveButton.Location = New-Object System.Drawing.Size(50,180)
    $saveButton.Size = New-Object System.Drawing.Size(290,40)
    $saveButton.Text = "Save password file"
    $saveButton.Add_Click({
        $objForm.Close()
    })
    $objForm.Controls.Add($saveButton)

    $objLabel = New-Object System.Windows.Forms.Label
    $objLabel.Location = New-Object System.Drawing.Size(10,20) 
    $objLabel.Size = New-Object System.Drawing.Size(360,60) 
    $objLabel.Text = "Note: the password file is saved using powershell secure credential.`n`nThe the password file is only valid under the user that created it. If you run the script as a scheduled task you should create the password file under the user account that runs the scheduled task."
    $objForm.Controls.Add($objLabel) 

    $objTextBox = New-Object System.Windows.Forms.TextBox 
    $objTextBox.Location = New-Object System.Drawing.Size(10,90) 
    $objTextBox.Size = New-Object System.Drawing.Size(360,20) 
    $objForm.Controls.Add($objTextBox) 
        
    $objLabel2 = New-Object System.Windows.Forms.Label
    $objLabel2.Location = New-Object System.Drawing.Size(10,120) 
    $objLabel2.Size = New-Object System.Drawing.Size(280,20) 
    $objLabel2.Text = "File location"
    $objForm.Controls.Add($objLabel2) 

    $objTextBox2 = New-Object System.Windows.Forms.TextBox 
    $objTextBox2.Location = New-Object System.Drawing.Size(10,140) 
    $objTextBox2.Size = New-Object System.Drawing.Size(360,20) 
    if ($psscriptroot)
    {
       $filelocation = Join-Path -Path $psscriptroot -ChildPath "mailboxpassword.bin" 
    }else
    {
        $filelocation = "c:\dmarcworkfolder\mailboxpassword.bin"
    }
    $objTextBox2.Text = $filelocation 
    $objForm.Controls.Add($objTextBox2)

    $objForm.Topmost = $True

    $objForm.Add_Shown({$objForm.Activate()})
    [void] $objForm.ShowDialog()

    Write-debug -Message "password: $password , Filelocation: $pwfilelocation"
    
    if (([string]::IsNullOrEmpty($objTextBox.Text)) -or ([string]::IsNullOrEmpty($objTextBox2.Text)))
    {
        Write-warning -Message "Either password or file location was empty. Please use valid value"
    }else
    {        
        $objTextBox.Text | ConvertTo-SecureString -AsPlainText -Force | ConvertFrom-SecureString | Out-File -FilePath $objTextBox2.Text
        $objTextBox.Text = $null
    }
    exit
}

#endregion generate passwordfile

#region test prerequisits 

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 1/10 Test Prerequisits:' -PercentComplete "10"

#region Test EWS Managed API

Write-Progress -id 1 -Activity "Loading local EWS API" -Status 'Progress:' -PercentComplete "10"
 
$exchangeewsavailable = $false

if (test-path "HKlM:\SOFTWARE\Microsoft\Exchange\Web Services" )
{
    $allEWSversions = Get-ChildItem "HKlM:\SOFTWARE\Microsoft\Exchange\Web Services"
    $sortedEWSversions = Sort-Object -InputObject $allEWSversions -Property name -Descending
    $latestEWSversion = Select-Object -InputObject $sortedEWSversions -First 1
    $ewspath = join-path -Path "HKlM:\SOFTWARE\Microsoft\Exchange\Web Services" -ChildPath $latestEWSversion.pschildname
    $EWSlocation = (Get-ItemProperty $ewspath)."install directory"
    $EWSdlllocation = Join-Path -Path $EWSlocation -ChildPath "Microsoft.Exchange.WebServices.dll"
    if (test-path -path $EWSdlllocation)
    {
        Import-Module -Name $EWSdlllocation
        $exchangeewsavailable = $true
    }else
    {
        throw "The EWS managed APi module could not be found on disk, make sure it is installed correctly - Tested path $EWSdlllocation"
        exit
    }
}else
{
    throw "The EWS managed API module could not be found in registry, make sure it is installed correctly - Tested path HKlM:\SOFTWARE\M1icrosoft\Exchange\Web Services"
    exit
}

#endregion Test EWS Managed API 

#region create and test EWS connection

Write-Progress -id 1 -Activity "Testing EWS connectivity" -Status 'Progress:' -PercentComplete "25"

if ($PsCmdlet.ParameterSetName -eq "Passwordfile")
{
    $securecredential = get-content $passwordfile | convertto-securestring
    $credentials = new-object -typename System.Management.Automation.PSCredential -argumentlist $dmarcmailboxusername,$securecredential
}else
{
    $dmarcmailboxpasswordsec = $dmarcmailboxpassword | ConvertTo-SecureString -AsPlainText -Force
    Remove-Variable -Name "dmarcmailboxpassword"
    Write-Debug -Message "$dmarcmailboxpasswordsec"
    $credentials = new-object -typename System.Management.Automation.PSCredential -argumentlist $dmarcmailboxusername,$dmarcmailboxpasswordsec
}

$EWSconnection = New-Object -TypeName Microsoft.Exchange.WebServices.Data.ExchangeService([Microsoft.Exchange.WebServices.Data.ExchangeVersion]::Exchange2007_SP1);   
$EWSconnection.Credentials = New-Object -TypeName Microsoft.Exchange.WebServices.Data.WebCredentials($credentials.UserName,$credentials.GetNetworkCredential().password)
$EWSconnection.UserAgent = "Tech-savvy dmarc parser www.tech-savvy.nl"


try
{
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$true
    }

    if ($BypassautodiscoverforO365.IsPresent)
    {
        $EWSconnection.Url = "https://outlook.office365.com/EWS/Exchange.asmx"
    }else
    {
        $EWSconnection.AutodiscoverUrl($dmarcmailbox, {$true})

        if (!($EWSconnection.url))
        {
            throw "no connection" 
        }
    }

}catch
{
    write-error -message "connection test failed for account $dmarcmailbox with real error : $($_.exception)"
    exit
}finally
{
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$false
    }
    $exchangeewsavailable = $false
}

#endregion create and test EWS connection

#region test 7zip location

Write-Progress -id 1 -Activity "locating 7 zip" -Status 'Progress:' -PercentComplete "50"

$7zippath = Join-Path -Path $ziplocation -ChildPath "7z.exe" 
if (test-path -path $7zippath )
{
    Write-Output -InputObject " 7 ZIP has been succesfully located " 
}else
{
    write-error -message "Error locating 7zip in the directory : $7zippath "
    exit
}

#endregion test 7zip location

#region test file locations 

Write-Progress -id 1 -Activity "test XML file directorys" -Status 'Progress:' -PercentComplete "90"

try
{
    $reportstoragedirxmltemp = join-path -Path $reportstoragedir -ChildPath "xml"
    $reportstoragedirxml = join-path -Path $reportstoragedirxmltemp -ChildPath (get-date -f "yyyy-MM")

    if (!(test-path -path $reportstoragedir))
    {
        $null = New-Item -Type "directory" -Path $reportstoragedir
        $null = New-Item -Type "directory" -Path $reportstoragedirxmltemp
        $null = New-Item -Type "directory" -Path $reportstoragedirxml
    }elseif (!(test-path -path $reportstoragedirxmltemp))
    {
        $null = New-Item -Type "directory" -Path $reportstoragedirxmltemp
        $null = New-Item -Type "directory" -Path $reportstoragedirxml
    }elseif (!(test-path -path $reportstoragedirxml))
    {
        $null = New-Item -Type "directory" -Path $reportstoragedirxml
    }
}catch
{
    Write-Error -Message "Error validating and creating file storage directorys at $reportstoragedir real error: $($_.exception)"
}

Write-Progress -id 1 -Activity "test XML file directorys" -Status 'Progress:' -Completed

#endregion test file locations

#region validate EWS folders and create if needed

#region get all folders

Write-Progress -id 1 -Activity "test XML file directorys" -Status 'Progress:' -PercentComplete "50"

#define folder id and type
$folderproptype = new-object Microsoft.Exchange.WebServices.Data.ExtendedPropertyDefinition(13825,[Microsoft.Exchange.WebServices.Data.MapiPropertyType]::Integer)
$folderid = new-object Microsoft.Exchange.WebServices.Data.FolderId([Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::inbox,$dmarcmailbox) 

#create folder view properties
$folderviewproptype = new-object Microsoft.Exchange.WebServices.Data.ExtendedPropertyDefinition(26293, [Microsoft.Exchange.WebServices.Data.MapiPropertyType]::String)   
$folderviewPropSet = new-object Microsoft.Exchange.WebServices.Data.PropertySet([Microsoft.Exchange.WebServices.Data.BasePropertySet]::FirstClassProperties)    
$folderviewPropSet.Add($folderviewproptype)
   
#create folder view filter
$FolderView =  New-Object Microsoft.Exchange.WebServices.Data.FolderView(10)
$FolderView.Traversal = [Microsoft.Exchange.WebServices.Data.FolderTraversal]::Deep
$FolderView.PropertySet = $folderviewPropSet

#The Search filter will exclude any Search Folders    
$SearchFilter = new-object Microsoft.Exchange.WebServices.Data.SearchFilter+IsEqualTo($folderproptype,"1")    
$folders = $null 
$allfolders = $null 

try
{ 
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$true
    }

    do {    
        $folders = $EWSconnection.FindFolders($folderid,$SearchFilter,$FolderView)
        $allfolders += $folders
        $FolderView.Offset += $folders.Item.Count  
     }while($folders.MoreAvailable -eq $true)

}catch
{
    write-error -message "Could not harvest all folders from $dmarcmailbox real error : $($_.exception)"

}finally
{
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$false
    }
}

#endregion get all folders
    
#region validate processed folder

try
{
    if ($allfolders.displayname -inotcontains "processed")
    {
        $newFolder = new-object Microsoft.Exchange.WebServices.Data.Folder($ewsconnection)
        $newFolder.DisplayName = "processed"
        $savelocation = new-object Microsoft.Exchange.WebServices.Data.FolderId([Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::inbox,$dmarcmailbox) 
        $newFolder.Save($savelocation)

        $dmarcprocessedfolder = $newFolder.id
    }else
    {
        $dmarcprocessedfoldertemp = $allfolders | Where-Object -FilterScript { $_.displayname -like "processed"} 
        $dmarcprocessedfolder = $dmarcprocessedfoldertemp.id
    }
    
    if ($allfolders.displayname -inotcontains "NodmarcRua")
    {
        $newFoldernonrua = new-object Microsoft.Exchange.WebServices.Data.Folder($ewsconnection)
        $newFoldernonrua.DisplayName = "NodmarcRua"
        $savelocationnonrua = new-object Microsoft.Exchange.WebServices.Data.FolderId([Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::inbox,$dmarcmailbox) 
        $newFoldernonrua.Save($savelocationnonrua)

        $dmarcnonruafolder = $newFoldernonrua.id
    }else
    {
        $dmarcnonruafoldertemp = $allfolders | Where-Object -FilterScript { $_.displayname -like "NodmarcRua"} 
        $dmarcnonruafolder = $dmarcnonruafoldertemp.id
    }
}catch
{
    write-error -message "Error creating the processed folders in mailbox $dmarcmailbox real error : $($_.exception)"

}finally
{
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$false
    }
}

#endregion validate processed folder

#region validate custom folder

if (!([string]::IsNullOrWhiteSpace($customsourcefolder)))
{
    if ($allfolders.displayname -inotcontains $customsourcefolder )
    {
        write-error -message "Error the custom folder $customsourcefolder is not present in the mailbox $dmarcmailbox real error : $($_.exception)"
        exit
    }
}

#endregion validate custom folder

#endregion validate EWS folders and create if needed

#endregion test prerequisits

#region process items 

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 2/10 process mailbox items:' -PercentComplete "20"

#region harvest items

#region set source folder

Write-Progress -id 1 -Activity "Configure source folders" -Status 'Progress:' -PercentComplete "10"

if (!([string]::IsNullOrWhiteSpace($customsourcefolder)))
{
    $dmarcsourcefolder = $allfolders | Where-Object -FilterScript { $_.displayname -like $customsourcefolder} 
}else
{   
    $dmarcsourcefoldername = new-object Microsoft.Exchange.WebServices.Data.FolderId([Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::inbox,$dmarcmailbox) 
    $dmarcsourceFolder = [Microsoft.Exchange.WebServices.Data.Folder]::Bind($EWSconnection,$dmarcsourcefoldername) 
}

#endregion set source folder

#region get all items

Write-Progress -id 1 -Activity "Harvest mail items" -Status 'Progress:' -PercentComplete "20"

$items = $null 
$allitems = $null 

try
{ 
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$true
    }

    #set the source folder id to the folder used to search for dmarc reports
    $dmarcsourcefolderid = $dmarcsourcefolder.id
    
    #create search filter to only get items with attachments
    $attachmentsfilter = new-object Microsoft.Exchange.WebServices.Data.SearchFilter+IsEqualTo([Microsoft.Exchange.WebServices.Data.EmailMessageSchema]::HasAttachments, $true)
    
    #create Item view filter
    $itemView = New-Object Microsoft.Exchange.WebServices.Data.ItemView(20) 
       
    do {
        
        #clear itterative variables
        $items = $null
        
        $items = $EWSconnection.Finditems($dmarcsourcefolderid,$attachmentsfilter,$itemView)
        $allitems += $items
        $itemView.Offset += $items.Item.Count  
        
        Write-Progress -id 1 -Activity "Harvest mail items" -Status 'Progress:' -PercentComplete "50" -CurrentOperation "discoverd $($allitems.count) items"

    }while($items.MoreAvailable -eq $true)

}catch
{
    write-error -message "Could not harvest all items from folder $($dmarcsourcefolder.DisplayName) in mailbox $dmarcmailbox real error : $($_.exception)"
}finally
{
    if ($trace.ispresent)
    {
        $EWSconnection.traceenabled=$false
    }
}

Write-Progress -id 1 -Activity "Harvest mail items" -Status 'Progress:' -Completed

#endregion get all items 

#endregion harvest items

#region save to disk and move processed items

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 3/10 Save reports to temp storage:' -PercentComplete "30"
$progress = 1 
[array]$counter = $allitems

foreach ( $report in $allitems ) 
{
    try
    {
        Write-Progress -id 1 -Activity "Save report to disk and move mail" -Status 'Progress:' -PercentComplete ( 100/$counter.count * $progress)
        
        # clear interative variables
        $attachment = $null
        $Filestream = $null
        $reportfilepath = $null


        if ($trace.ispresent)
        {
            $EWSconnection.traceenabled=$true
        }        

        $report.load()

        if ($report.Subject -match "Report Domain.*Submitter.*Report.*ID")
        {
            foreach ( $attachment in $report.attachments)
            {
                $attachment.load()
        
                #save the attachment to disk
                $reportfilepath = join-path -path $reportstoragedir -ChildPath $attachment.Name.ToString()
                $Filestream = new-object System.IO.FileStream($reportfilepath, [System.IO.FileMode]::Create)
                $Filestream.Write($attachment.Content, 0, $attachment.Content.Length)
                $Filestream.Close()
            }
            
            if ($deleteprocesseditem.ispresent)
            {            
                #delete item
                $null = $report.delete([Microsoft.Exchange.WebServices.Data.DeleteMode]::$deletetoaction)
                Write-verbose -Message "Deleting item $($report.Subject)"
            }else
            {
                #move the item to processed folder
                $null = $report.Move($dmarcprocessedfolder)
                Write-verbose -Message "moving item $($report.Subject)"
            }

        }else
        {
            #move the item to Non rua folder
            $null = $report.Move($dmarcnonruafolder)
            Write-verbose -Message "moving item to non RUA folder $($report.Subject)"
        }

        $progress++
    }catch
    {
        write-error -message "Error processing report with message id $($report.InternetMessageId) with error : $($_.exception)"
        $progress++
    }finally
    {
        if ($trace.ispresent)
        {
            $EWSconnection.traceenabled=$false
        }
        $exchangeewsavailable = $false
    }
}

Write-Progress -id 1 -Activity "Save report to disk and move mail" -Completed

#endregion save to disk and move processed items

#endregion process items

#region extract reports from compressed templocation

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 4/10 unpack reports:' -PercentComplete "40"

$progress = 1 

#Get all compressed files
[array]$allcompressedfiles = Get-ChildItem -Path $reportstoragedir -File

if ([string]::IsNullOrWhiteSpace($allcompressedfiles))
{
    Write-Warning -Message "No new compressed files to process detected"
}else
{
    $filesinerror = @()

    foreach ($compressedfile in $allcompressedfiles)
    {
        Write-Progress -id 1 -Activity "decompress report files" -Status 'Progress:' -PercentComplete ( 100/$allcompressedfiles.count * $progress)
        #clean iterative variables
        $compressedfilelocation = $null
        $AllArgs = $null
        $zipresult = $null
        $fileextractable = $null

        switch -wildcard ($compressedfile.Extension)
        {
        
            "*dll" {write-output -InputObject "A Dll file cant be extracted";$fileextractable = "No"}
            "*exe" {write-output -InputObject "A exe file cant be extracted";$fileextractable = "No"}
            "*ps1" {write-output -InputObject "A ps1 file cant be extracted";$fileextractable = "No"}
            "*txt" {write-output -InputObject "A txt file cant be extracted";$fileextractable = "No"}
            "*bin" {write-output -InputObject "A bin file cant be extracted";$fileextractable = "No"}
            "*csv" {write-output -InputObject "A csv file cant be extracted";$fileextractable = "No"}
            Default {$fileextractable = "Yes"}
        }
        
        try
        {
            if ($fileextractable -eq "yes")
            {
                #unzip all the reports from the compressed files
                $compressedfilelocation = $compressedfile.fullname
                $AllArgs =  @('e','-y',"-o$reportstoragedirxml",$compressedfilelocation)
                $7zipexe = join-path -path $ziplocation -ChildPath "7z.exe"
                write-verbose "Extracting file $compressedfile"
                $zipresult = & $7zipexe $AllArgs

                #remove the compressed file

                if (test-path $compressedfilelocation)
                {
                    if ($zipresult.Contains("Everything is Ok"))
                    {
                        remove-item -path $compressedfilelocation -Confirm:$false
                    }else
                    {
                        Throw "something went wrong"
                    
                    }
                }else
                {
                    Throw "file does not exsist"
               
                }
            }
            else
            {
                Throw "Not a zip file" 
            }

            $progress++
        }catch
        {
            $filesinerror += $compressedfile
            $progress++
        }
        
    }

    if ($filesinerror.count -gt 0)
    {
        Write-Warning -Message "the following files cloud not be extracted $($filesinerror.fullname)"
    }

    Write-Progress -id 1 -Activity "Save report to disk and move mail" -Completed
}

#endregion extract reports from compressed templocation

#region create report

#region analyse data

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 5/10 analysing the data:' -PercentComplete "50"

#region import xml files

Write-Progress -id 1 -Activity "Reading report files" -Status 'Progress:' -PercentComplete 1

$progress = 1
[array]$xmlfiles = Get-ChildItem -Path $reportstoragedirxml -Filter "*.xml"
$xml = @()

if ($xmlfiles.count -ge 1)
{
    foreach ( $item in $xmlfiles  )
    {

        if (!($progress % 5))
        {
            Write-Progress -id 1 -Activity "Reading report files" -Status 'Progress:' -PercentComplete ( 100/$xmlfiles.count * $progress)
        }
    
        if ( $item.extension -eq ".xml")
        {
            $xml += [xml](Get-Content -Path $item.fullname)
        }
    
        $progress ++
    }
}else
{
    write-error -message "Error cloud not locate any XML file in the folder $reportstoragedirxml"
    exit
}

Write-Progress -id 1 -Activity "Reading report files" -Status 'Progress:' -Completed

#endregion import xml files

#region restructure the data

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 6/10 Restructure the data:' -PercentComplete "60"
Write-Progress -id 1 -Activity "Creating new database table" -Status 'Progress:' -PercentComplete 1

$progress = 1
$recordcount = $null
$mastertable = @()
$date = (get-date -f "yyyy-MM")
$iptable = @{}

$iptabledatabasepath = join-path -Path $reportstoragedir -ChildPath "iptabledatabase.csv"

if (test-path -Path $iptabledatabasepath)
{
    $iptabledate = (Get-Item -Path $iptabledatabasepath).CreationTime
    $iptablerenewdate = (get-date).AddDays(-$iptabledaysold)

    if ( $iptablerenewdate -lt $iptabledate )
    {
        $iptableimported = Import-Csv -Path $iptabledatabasepath -Delimiter ","
        Write-Verbose -Message "Loading IP database"
        Write-Verbose -Message "converting to hash table"
        
        for ($i=1 ; $i -lt $iptableimported.count ; $i++)
        {
             $iptable.Add($iptableimported[$i].key,$iptableimported[$i].value)   
        }

    }else
    {
        Remove-Item -Path $iptabledatabasepath -force
        add-content -value "key,value" -Path $iptabledatabasepath
        Write-Verbose -Message "IP database is to old creating new one"
    }
}else
{
    Add-content -value "key,value" -Path $iptabledatabasepath
}

#set geo counter for max of 10000 querys per hour
$geocounter = 0

foreach ($record in $xml)
{
    #clear itterative variables
    $recordcount = $null
    $geouri = $null

    if (!($progress % 3))
    {
        Write-Progress -id 1 -Activity "Creating new database table" -Status 'Progress:' -PercentComplete ( 100/$xml.count * $progress)
    }

    Write-Progress -id 2 -Activity "Creating new subdatabase table" -Status 'Progress:' -PercentComplete 1
    $progress1 = 1

    $recordcount = $record.feedback.record.count
    if (!($recordcount))
    {
        $recordcount = 1
    }

    foreach ($domainrecord in $record.feedback.record)
    {
        Write-Progress -id 2 -Activity "Creating new subdatabase table" -Status 'Progress:' -PercentComplete ( 100/$recordcount * $progress1)
        
        #clear itterative variables
        $hostname = $null
        $ip = $null
        $sleepcounter = $null

        #start loop
        $ip = $domainrecord.row.source_ip
        if ( $ip ) 
        {
            if ( $iptable.ContainsKey($ip))
            {
                [array]$hostname = $iptable.$ip
            }else
            {
                [array]$hostname = (Resolve-DnsName -name $domainrecord.row.source_ip -Type ptr -QuickTimeout -ErrorAction SilentlyContinue).namehost
                if ([string]::IsNullOrEmpty($hostname))
                {
                    [array]$hostname = "hostname is not resolved"
                }
                
                $iptable += @{ $ip = $($hostname[0])}
                
            }
        }

        $temptable = new-object psobject

        if ( $PowerBIuploadenabled.IsPresent )
        {
            $sdate = get-date (get-date -format "yyyy-MM-dd") -Format "s"
            $temptable | Add-Member -Type NoteProperty -Name "Realprocessdate" -Value $sdate
        }

        $temptable | Add-Member -Type NoteProperty -Name "processdate" -Value $date
        $temptable | Add-Member -Type NoteProperty -Name "orgname" -Value $record.feedback.report_metadata.org_name
        $temptable | Add-Member -Type NoteProperty -Name "dmarcdomain" -Value ([array]$record.feedback.policy_published.domain)[0]
        $temptable | Add-Member -Type NoteProperty -Name "sourceip" -Value $domainrecord.row.source_ip
        $temptable | Add-Member -Type NoteProperty -Name "sourcedomain" -Value $($hostname[0])
        $temptable | Add-Member -Type NoteProperty -Name "sourceipcount" -Value $domainrecord.row.count
        $temptable | Add-Member -Type NoteProperty -Name "dmarcdisposition" -Value $domainrecord.row.policy_evaluated.disposition
        $temptable | Add-Member -Type NoteProperty -Name "dmarcspf" -Value $domainrecord.row.policy_evaluated.spf
        $temptable | Add-Member -Type NoteProperty -Name "dmarcdkim" -Value $domainrecord.row.policy_evaluated.dkim
        $temptable | Add-Member -Type NoteProperty -Name "headerfrom" -Value $domainrecord.identifiers.header_from
        $temptable | Add-Member -Type NoteProperty -Name "dkimdomain" -Value $domainrecord.auth_results.dkim.domain
        $temptable | Add-Member -Type NoteProperty -Name "dkimresult" -Value $domainrecord.auth_results.dkim.result
        $temptable | Add-Member -Type NoteProperty -Name "spfdomain" -Value $domainrecord.auth_results.spf.domain
        $temptable | Add-Member -Type NoteProperty -Name "spfscope" -Value $domainrecord.auth_results.spf.scope
        $temptable | Add-Member -Type NoteProperty -Name "spfresult" -Value $domainrecord.auth_results.spf.result
        
        if ($geolookupenabled.IsPresent)
        {
            #$geouri = "http://freegeoip.net/json/" + $domainrecord.row.source_ip This one is depricated and now needs APIKEYS
            $geouri = "http://ip-api.com/json/" + $domainrecord.row.source_ip
            
            try
            {
                #introduce sleep to reduce calls to below 150 per minute threshold now 120 due to delay
                start-sleep -Milliseconds 500
                $geodata = Invoke-RestMethod -Method Get -Uri $geouri
                
            }catch
            {
            
            }
             
            $temptable | Add-Member -Type NoteProperty -Name "latitude" -Value $geodata.lat
            $temptable | Add-Member -Type NoteProperty -Name "longitude" -Value $geodata.lon
            $temptable | Add-Member -Type NoteProperty -Name "country_name" -Value $geodata.country
            $temptable | Add-Member -Type NoteProperty -Name "region_name" -Value $geodata.regionname
            $temptable | Add-Member -Type NoteProperty -Name "city" -Value $geodata.city

        }
        
        $mastertable += $temptable
        $progress1++

    }
    $progress++    
}

Write-Progress -id 2 -Activity "Creating new subdatabase table" -Status 'Progress:' -completed

#remove ipdatabase 
if (test-path -path $iptabledatabasepath)
{
    Clear-Content -Path $iptabledatabasepath -Confirm:$false
    Add-Content -Path $iptabledatabasepath -Value "key,value"
}     

$iptable.GetEnumerator() | foreach-object -Process {add-content -value ("$($_.name),$($_.value)") -Path $iptabledatabasepath}

$mastertablefile = Join-Path -path $reportstoragedirxml -ChildPath ((get-date -f "yyyyMMddhhmmss") + ".csv")

#if mastertable exists some how do it again so new file will be selected due to seconds in filename
if (Test-Path -Path $mastertablefile)
{ 
    $mastertablefile = Join-Path -path $reportstoragedirxml -ChildPath ((get-date -f "yyyyMMddhhmmss") + ".csv")
}

Try
{
    $mastertable | export-csv -Path $mastertablefile
}catch
{
    write-error -message "Error saving the new master data table to disk location $mastertablefile with error : $($_.exception)"
    exit
}

Write-Progress -id 1 -Activity "Creating new database table" -Status 'Progress:' -Completed

#endregion restructure the data

#region generate new report from all data tables

#region Load all data 

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 7/10 Reload all data:' -PercentComplete "70"

$reporttable = @()

[array]$reportrootdir = Get-ChildItem -Path $reportstoragedirxmltemp

if ( $reportrootdir.count -ne 0 )
{
    $reportdirs = $reportrootdir | where-object -filterscript { $_.CreationTime -gt (get-date).adddays(-$reportrange) }
}else
{
    Write-Warning -Message "No report directorys found "
    exit
}

foreach ($reportdir in $reportdirs)
{
    #clearing itterative variables
    $compiledreports = $null
    $lastcsvfile = $null

    [array]$compiledreports = Get-ChildItem -Path $reportdir.FullName -Filter "*.csv" | Sort-Object -Property CreationTime
    
    if ($compiledreports.count -ge 1)
    {
        $lastcsvfile = $compiledreports[-1].fullname
        $reporttable += Import-Csv -Path $compiledreports[-1].fullname
    }else
    {
        Write-Warning -Message "No mastertable csv found in directorys $($reportdir.FullName)"
    }   
}


#region Filter failed or passed based on switches

if ($DMARCfailedonly.ispresent -or $DMARCpassedonly.ispresent)
{
    $reporttableoriginal = $reporttable
    $reporttable = $reporttable.where({($_.dmarcspf -eq "pass" -and $_.spfresult -eq "pass") -or ($_.dmarcdkim -eq "pass" -and $_.dkimresult -eq "pass")}, "split")   

    if ($DMARCpassedonly.ispresent)
    {
        $reporttable = $reporttable[0]
    }

    if ($DMARCfailedonly.ispresent -or $DMARCpassedonly.ispresent)
    {
        $reporttable = $reporttable[1]
    }
}

#endregion Filter failed or passed based on switches 

#endregion load all data 

#region create main report data

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 8/10 Format report data:' -PercentComplete "80"

$reportpath = join-path -path $reportstoragedir -ChildPath "report"

try
{
    if (!(test-path -Path $reportpath))
    {
        new-item -ItemType "directory" -Path $reportpath
    }
}catch
{
    Write-Error -Message "Error creating the report folder $reportpath"
    exit
}

#populate views of the mastertable and current month table
$grouptotaldatapermonth = $reporttable | Group-Object -Property "processdate" | Sort-Object -Property count -Descending
$grouptotaldataperdomain = $reporttable | Group-Object -Property "dmarcdomain" | Sort-Object -Property count -Descending
$grouptotaldatapersourceip = $reporttable | Group-Object -Property "sourceip" | Sort-Object -Property count -Descending

$grouptotalcurrentdataperdomain = $reporttable | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "dmarcdomain" | Sort-Object -Property count -Descending
$grouptotalcurrentdatapersourceip = $reporttable | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "sourceip" | Sort-Object -Property count -Descending
$grouptotalcurrentdataperorgname = $reporttable | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "orgname" | Sort-Object -Property count -Descending

#endregion create main report data 

#endregion generate new report from all data tables

#endregion analyse data

#region upload to powerBI

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 8.5/10 Main report Page:' -PercentComplete "90"
$progress = 1

if ($PowerBIuploadenabled.IsPresent)
{

    [array]$datafiles = Get-ChildItem -Path $reportstoragedirxml -Filter "*.csv" | Sort-Object -Property CreationTime -Descending
    $differancefile = $datafiles[0]
    $sourcefile = $datafiles[1]
    if ($sourcefile)
    {
        if (test-path -path $sourcefile.fullname)
        {
            $sourceobject = Import-Csv -Path $sourcefile.fullname
        }else
        {
            $sourceobject = $null
        }
    }
    $differanceobject = import-csv -Path $differancefile.fullname
    $newrecords = Compare-Object -ReferenceObject $sourceobject -DifferenceObject $differanceobject

    if ($DMARCfailedonly.ispresent -or $DMARCpassedonly.ispresent)
    {
        $newrecords = $newrecords.inputobject.where({($_.dmarcspf -eq "pass" -and $_.spfresult -eq "pass") -or ($_.dmarcdkim -eq "pass" -and $_.dkimresult -eq "pass")}, "split")   

        if ($DMARCpassedonly.ispresent)
        {
            $reporttable = $reporttable[0]
        }

        if ($DMARCfailedonly.ispresent -or $DMARCpassedonly.ispresent)
        {
            $reporttable = $reporttable[1]
        }

    }else
    {
        $newrecords = $newrecords.inputobject
    }

    foreach ( $payload in $newrecords )
    {
        Write-Progress -id 1 -Activity "uploading to powerBI" -Status 'Progress:' -PercentComplete ( 100/$newrecords.count * $progress)
        Write-PowerBI -payload $payload -endpoint $endpoint
        
        #adding throttle to prevent throttling at the API side ( API accepts 5 per second )
        start-sleep -Milliseconds 300
        $progress++

    }

}

#endregion upload to PowerBI

#region create mainpage report

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 9/10 Main report Page:' -PercentComplete "95"

$htmlbodyparts = @()
$htmlbodyparts += "<H1>This report is generated with DMARC report script version $version from http://www.tech-savvy.nl</H1>"
$htmlbodyparts += "<H1>Per month reports are generated for last $reportrange days</H1>" 

#create shortcut menu for the per month reports
foreach ($dateentry in $grouptotaldatapermonth)
{
    $dateformat = get-date $dateentry.name -format "yyyy-MM"
    $htmlbodyparts += @" 
        <a href=".\$($dateformat).html" target="_blank">Report for $($dateformat) </a>
        <br>
"@
}

#create main report for all data
$htmlbodyparts += "<H1>Reports totals for last $reportrange days</H1><br>" 

if (!($DMARCfailedonly.ispresent -or $DMARCpassedonly.ispresent))
{
    $htmlbodyparts += "<H3>Total Failed DMARC Items VS Total passed DMARC items</H3><br>" 

    $reporttablefailed = $reporttable.where({($_.dmarcspf -eq "pass" -and $_.spfresult -eq "pass") -or ($_.dmarcdkim -eq "pass" -and $_.dkimresult -eq "pass")}, "split")   
    $reporttablefailed = $reporttablefailed[1]

    $failedhtmltemp = $null  
    $failedhtmltemp = "1" | select @{name="Failed DMARC Checks of Total in report";expression={"$($reporttablefailed.count)/$($reporttable.count)"}},
        @{name="% part of total";expression={
            $bartotalobjects = $reporttable.count
            $barpercent = (100 / $bartotalobjects) * $reporttablefailed.count
            $barleft = $barsign * ($barpercent )
            $barright = $barempty * ((100 - $barpercent) )
            "xopenFont color=bluexclose{0}xopen/FontxclosexopenFont color=blackxclose{1}%xopen/FontxclosexopenFont Color=greenxclose{2}xopen/fontxclose" -f $barleft,$barpercent.tostring("000"),$barright
        }
    } | ConvertTo-Html -Fragment 

        $failedhtmltemp = $failedhtmltemp -replace "xopen","<"
        $failedhtmltemp = $failedhtmltemp -replace "xclose",">"
        $failedhtmltemp = $failedhtmltemp -replace "<table>",'<table class="style2">'

    $htmlbodyparts += $failedhtmltemp
}       


$htmlbodyparts += Generate-TableWithSubTable -name "Dmarc per domain" -columnname "Dmarc domains" -table $grouptotaldataperdomain -topXresults $topXresults

#create tabed class
$htmlbodyparts += '<div class="tabs">'   

#set checked radio button for first tab report
$checked = "checked"

foreach ( $domain in $grouptotaldataperdomain )
{
    #clear itterative variables
    $htmlbodyparts3 = $null
    $domainname = $null
    $grouptotaldataperorgname = $null
    $grouptotaldataperspfresult = $null
    $grouptotaldataperdkimresult = $null
    $grouptotaldataperaspfresult = $null
    $grouptotaldataperadkimresult = $null
    $grouptotaldataperDMARCresult = $null

    
    #start loop
    $domainname = $domain.name
    
    #add tab for each domain to the main HTML code
    $htmlbodyparts += @"
     
       <div class="tab">
           <input type="radio" id="$domainname" name="Tabset1" $checked>
           <label for="$domainname">$domainname</label>       
           <div class="content">
               <iframe src=$domainname.html height="730" width="1300"></iframe>
           </div> 
       </div>
     
"@
    #disable checked for other tabs
    $checked = $null

    #generate data for the tab content per domain
    $grouptotaldataperorgname = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "orgname" | Sort-Object -Property count -Descending
    $grouptotaldataperspfresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "spfresult" | Sort-Object -Property count -Descending
    $grouptotaldataperdkimresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "dkimresult" | Sort-Object -Property count -Descending
    $grouptotaldataperaspfresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "dmarcspf" | Sort-Object -Property count -Descending
    $grouptotaldataperadkimresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "dmarcdkim" | Sort-Object -Property count -Descending
    $grouptotaldataperDMARCresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | Group-Object -Property "dmarcdisposition" | Sort-Object -Property count -Descending
    if ($geolookupenabled.IsPresent)
    {
        [array]$grouptotaldatapercountry = $reporttable | ?{$_.dmarcdomain -like $domainname} | Where-Object -FilterScript {$_."dmarcspf" -eq "fail" -and $_."dmarcdkim" -eq "fail"} | Group-Object -Property "country_name" | Sort-Object -Property count -Descending
    }

    if ( $grouptotaldataperDMARCresult) {$htmlbodyparts3 += Generate-TableWithSubTable -name "Dmarc Policy applied" -columnname "DMARC policy action" -table $grouptotaldataperDMARCresult -topXresults $topXresults}
    if ( $grouptotaldataperspfresult) {$htmlbodyparts3 += Generate-TableWithSubTable -name "SPF results" -columnname "SPF result" -table $grouptotaldataperspfresult -topXresults $topXresults}
    if ( $grouptotaldataperdkimresult) {$htmlbodyparts3 += Generate-TableWithSubTable -name "DKIM results" -columnname "DKIM result" -table $grouptotaldataperdkimresult -topXresults $topXresults}
    if ( $grouptotaldataperaspfresult) {$htmlbodyparts3 += Generate-TableWithSubTable -name "DMARC SPF Align results" -columnname "SPF alignment result" -table $grouptotaldataperaspfresult -topXresults $topXresults}
    if ( $grouptotaldataperadkimresult) {$htmlbodyparts3 += Generate-TableWithSubTable -name "DMARC DKIM Align results" -columnname "DKIM alignment result" -table $grouptotaldataperadkimresult -topXresults $topXresults}
    if ( $grouptotaldataperorgname) {$htmlbodyparts3 += Generate-TableWithSubTable -name "Received reports per org" -columnname "DMARC report sender" -table $grouptotaldataperorgname -topXresults $topXresults}
    if ($geolookupenabled.IsPresent)
    {
        if ( $grouptotaldatapercountry) {$htmlbodyparts3 += Generate-TableWithSubTablecountry -name "DMARC checks failed per country" -columnname "Country" -table $grouptotaldatapercountry -topXresults $topXresults}
    }


    #generate the sub HTML page 
    ConvertTo-Html -head $stylesheet -body $htmlbodyparts3 | Out-File -FilePath (join-path -path ($reportpath) -childpath "$domainname.html")


    #region Create country data html per domain
    
    if ($geolookupenabled.IsPresent)
    {
        if ($grouptotaldatapercountry.count -ge 1)
        {
            
            foreach ($countryreport in $grouptotaldatapercountry )
            { 

                #clear itterative variables
                $htmlbodypartsgeo = $null
                $geogrouptotaldataperorgname = $null
                $geogrouptotaldataperspfresult = $null
                $geogrouptotaldataperdkimresult = $null
                $geogrouptotaldataperDMARCresult = $null
                $geogrouptotaldataperaspfresult = $null
                $geogrouptotaldataperadkimresult = $null
            

                $htmlbodypartsgeo = @()
                $htmlbodypartsgeo += "<H1>Report generated for $($countryreport.name)</H1><br>" 
           
                #generate the geo country HTML page

                #generate data for the country
           
                $geogrouptotaldataperorgname = $countryreport.group |  Group-Object -Property "orgname" | Sort-Object -Property count -Descending
                $geogrouptotaldataperspfresult = $countryreport.group | Group-Object -Property "spfresult" | Sort-Object -Property count -Descending
                $geogrouptotaldataperdkimresult = $countryreport.group | Group-Object -Property "dkimresult" | Sort-Object -Property count -Descending
                $geogrouptotaldataperaspfresult = $countryreport.group | Group-Object -Property "dmarcspf" | Sort-Object -Property count -Descending
                $geogrouptotaldataperadkimresult = $countryreport.group | Group-Object -Property "dmarcdkim" | Sort-Object -Property count -Descending
                $geogrouptotaldataperDMARCresult = $countryreport.group | Group-Object -Property "dmarcdisposition" | Sort-Object -Property count -Descending

                #write HTML code
                if ( $geogrouptotaldataperDMARCresult) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "Dmarc Policy applied" -columnname "DMARC policy action" -table $geogrouptotaldataperDMARCresult -topXresults $topXresults}
                if ( $geogrouptotaldataperspfresult) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "SPF results" -columnname "SPF result" -table $geogrouptotaldataperspfresult -topXresults $topXresults}
                if ( $geogrouptotaldataperdkimresult) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "DKIM results" -columnname "DKIM result" -table $geogrouptotaldataperdkimresult -topXresults $topXresults}
                if ( $geogrouptotaldataperaspfresult) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "DMARC SPF Align results" -columnname "SPF alignment result" -table $geogrouptotaldataperaspfresult -topXresults $topXresults}
                if ( $geogrouptotaldataperadkimresult) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "DMARC DKIM Align results" -columnname "DKIM alignment result" -table $geogrouptotaldataperadkimresult -topXresults $topXresults}
                if ( $geogrouptotaldataperorgname) {$htmlbodypartsgeo += Generate-TableWithSubTable -name "Received reports per org" -columnname "DMARC report sender" -table $geogrouptotaldataperorgname -topXresults $topXresults}
      
                #save the report to disk
                $geofilelocation = join-path -path ($reportpath) -childpath "$($domainname)_$($countryreport.name).html"

                ConvertTo-Html -head $stylesheet -body $htmlbodypartsgeo | Out-File -FilePath $geofilelocation
 
            }
        }
    }

    #endregion Create country data html per domain

}

$htmlbodyparts += '</div>'

#add Iframe for sources table
$htmlbodyparts += '<sourceipiframe>'
$htmlbodyparts += '<iframe src=Index2.html height="730" width="1300"></iframe>'
$htmlbodyparts += '</sourceipiframe>'
$htmlbodyparts += ("<br><I>Report run on {0} by {1}<I>" -f (Get-Date -displayhint date),"http://tech-savvy.nl")

#convert it all to HTML file
ConvertTo-Html -head $stylesheet -body $htmlbodyparts | Out-File -FilePath (join-path -path ($reportpath) -childpath "Index.html")

#create the HTML code for the iframe of the source ips
$htmlbodyparts2 = @()
if ($geolookupenabled.IsPresent)
{
    $htmlbodyparts2 += Generate-Table "Sender source ips" "Source ip" $grouptotaldatapersourceip -geolookupenabled
}else
{
    $htmlbodyparts2 += Generate-Table "Sender source ips" "Source ip" $grouptotaldatapersourceip
}

ConvertTo-Html -head $stylesheet -body $htmlbodyparts2 | Out-File -FilePath (join-path -path ($reportpath) -childpath "Index2.html")

#endregion create mainpage report

#region create report for current month

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 10/10 monthly report page:' -PercentComplete "95"

$htmlbodyparts = @()
$htmlbodyparts += "<H1>Report generated for $date</H1><br>" 
if ($grouptotalcurrentdataperdomain) {$htmlbodyparts += Generate-TableWithSubTable -name "DMARC per domain" -columnname "DMARC domains" -table $grouptotalcurrentdataperdomain -topXresults $topXresults}

#create the HTML tabs 
$htmlbodyparts += '<div class="tabs">'   
$checked = "checked"
foreach ( $domain in $grouptotaldataperdomain )
{
    #clean itterative variables
    $htmlbodyparts4 = $null
    $grouptotalcurrentdataperspfresult = $null
    $grouptotalcurrentdataperorgname = $null
    $grouptotalcurrentdataperdkimresult = $null
    $grouptotalcurrentdataperaspfresult = $null
    $grouptotalcurrentdataperadkimresult = $null
    $grouptotalcurrentdataperDMARCresult = $null
        
    
    #set domain name for html report filename
    $domainname = $domain.name
    $htmlbodyparts += @"
     
       <div class="tab">
           <input type="radio" id="$domainname" name="Tabset1" $checked>
           <label for="$domainname">$domainname</label>       
           <div class="content">
               <iframe src=$date$domainname.html height="730" width="1300"></iframe>
           </div> 
       </div>
     
"@
    $checked = $null

    $grouptotalcurrentdataperspfresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "spfresult" | Sort-Object -Property count -Descending
    $grouptotalcurrentdataperaspfresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "dmarcspf" | Sort-Object -Property count -Descending
    $grouptotalcurrentdataperorgname = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "orgname" | Sort-Object -Property count -Descending
    $grouptotalcurrentdataperdkimresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "dkimresult" | Sort-Object -Property count -Descending
    $grouptotalcurrentdataperadkimresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "dmarcdkim" | Sort-Object -Property count -Descending
    $grouptotalcurrentdataperDMARCresult = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Group-Object -Property "dmarcdisposition" | Sort-Object -Property count -Descending
    if ($geolookupenabled.IsPresent)
    {
        $grouptotalcurrentdatapercountryname = $reporttable | ?{$_.dmarcdomain -like $domainname} | where-object -FilterScript {$_."processdate" -eq $date} | Where-Object -FilterScript {$_."dmarcspf" -eq "fail" -and $_."dmarcdkim" -eq "fail"} | Group-Object -Property "Country_name" | Sort-Object -Property count -Descending
    }

    #Only add the HTML code if something was found
    if ( $grouptotalcurrentdataperDMARCresult) {$htmlbodyparts4 += Generate-TableWithSubTable -name "DMARC Policy applied" -columnname "DMARC policy action" -table $grouptotalcurrentdataperDMARCresult -topXresults $topXresults}
    if ( $grouptotalcurrentdataperspfresult ) {$htmlbodyparts4 += Generate-TableWithSubTable -name "SPF results" -columnname "SPF result" -table $grouptotalcurrentdataperspfresult -topXresults $topXresults}
    if ( $grouptotalcurrentdataperdkimresult ) {$htmlbodyparts4 += Generate-TableWithSubTable -name "DKIM results" -columnname "DKIM result" -table $grouptotalcurrentdataperdkimresult -topXresults $topXresults}
    if ( $grouptotalcurrentdataperorgname) {$htmlbodyparts4 += Generate-TableWithSubTable -name "Received reports per org" -columnname "DMARC report sender" -table $grouptotalcurrentdataperorgname -topXresults $topXresults}
    if ( $grouptotalcurrentdataperaspfresult ) {$htmlbodyparts4 += Generate-TableWithSubTable -name "DMARC SPF align results" -columnname "SPF alignment result" -table $grouptotalcurrentdataperaspfresult -topXresults $topXresults}
    if ( $grouptotalcurrentdataperadkimresult ) {$htmlbodyparts4 += Generate-TableWithSubTable -name "DMARC DKIM align results" -columnname "DKIM alignment result" -table $grouptotalcurrentdataperadkimresult -topXresults $topXresults}
    if ($geolookupenabled.IsPresent)
    {
        if ( $grouptotalcurrentdatapercountryname ) {$htmlbodyparts4 += Generate-TableWithSubTable -name "DMARC FAIL per country" -columnname "country" -table $grouptotalcurrentdatapercountryname -topXresults $topXresults}
    }

    ConvertTo-Html -head $stylesheet -body $htmlbodyparts4 | Out-File -FilePath (join-path -path ($reportpath) -childpath "$date$domainname.html")
}

#close the html tabs class
$htmlbodyparts += '</div>'

#add Iframe for source ip table
$htmlbodyparts += '<sourceipiframe>'
$htmlbodyparts += '<iframe src='+$date+'_2.html height="730" width="1300"></iframe>'
$htmlbodyparts += '</sourceipiframe>'
$htmlbodyparts += ("<br><I>Report run on {0} by {1}<I>" -f (Get-Date -displayhint date),"http://tech-savvy.nl")

ConvertTo-Html -head $stylesheet -body $htmlbodyparts | Out-File -FilePath (join-path -path ($reportpath) -childpath "$date.html")


#create the HTML code for the iframe of the source ips
$htmlbodyparts2 = @()
if ($geolookupenabled.IsPresent)
{
    if ($grouptotalcurrentdatapersourceip) {$htmlbodyparts2 += Generate-Table "Sender source ips" "Source ip" $grouptotalcurrentdatapersourceip -geolookupenabled}
}else
{
    if ($grouptotalcurrentdatapersourceip) {$htmlbodyparts2 += Generate-Table "Sender source ips" "Source ip" $grouptotalcurrentdatapersourceip}
}

ConvertTo-Html -head $stylesheet -body $htmlbodyparts2 | Out-File -FilePath (join-path -path ($reportpath) -childpath "$($date)_2.html")

Write-Progress -Activity "Generating DMARC Report" -Status 'Progress Phase 10/10 monthly report page:' -Completed

#endregion create report for current month

#endregion create report

