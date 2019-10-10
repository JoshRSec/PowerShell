Function Add-ToZIP {
    <#
.Synopsis
Adds files to ZIP archive based on last write time

.DESCRIPTION
This function adds filetypes of your choice to an ZIP archive, with the option to group by day as well as delete source file after archiving,

.NOTES   
Name       : Add-ToZIP
Author     : Joshua Robbins
Version    : 1
DateUpdated: 10/10/2019

.PARAMETER FileSourceDirectory
The directory in which the source files are located, can be UNC.

EXAMPLE:
C:\Folder
\\Machine01\C$\Folder

.PARAMETER ZipDestinationDirectory
The directory in which the ZIP should be output, can be UNC.

EXAMPLE:
C:\Folder
\\Machine01\C$\Folder

.PARAMETER FindFilesOlderThanDays
The minimum LastWriteTime to look for, default is 7 days.

.PARAMETER GroupFilesByDay
Allows source files to be archived by day of creation instead archiving by individual file. Default is false.

.PARAMETER CleanOnZip
When true, this will delete the source file from the source directory. Default is false.

.PARAMETER FileType
When true, this will delete the source file from the source directory. Default is false.
#>

    param(
        [Parameter(Position = 0, mandatory = $true)]
        [string]
        $FileSourceDirectory,
        [Parameter(Position = 1, mandatory = $true)]
        [string]
        $ZipDestinationDirectory,
		[Parameter(Position = 2, mandatory = $true)]
        [string]
        $FileType,
        [int]
        $FindFilesOlderThanDays = 7,
        [Switch]
        $GroupFilesByDay,
        [Switch]
        $CleanOnZip
    )

    $stopwatch = [system.diagnostics.stopwatch]::StartNew()
    $_pwd = Get-Location
    $_fileAgeDate = (Get-Date).AddDays(-$FindFilesOlderThanDays)

    #Test if the source folder exists, if not break.
    $_fileSource = $($FileSourceDirectory)
    $_testSourceBool = Test-Path -Path $($_fileSource)
    if ($_testSourceBool -eq $FALSE) {
        Write-Error "Cannot find source directory: $($_fileSource)"
        Break
        Write-Verbose "Break"
    }
    else {
        $_allFilesArr = Get-ChildItem -Path "$($FileSourceDirectory)\*.$($FileType)" #Get all .$($FileType) files in present directory
        Write-Verbose "Looking for .$($FileType) files within '$($FileSourceDirectory)'"
    }

    #Test if the destination folder exists, if not offer option to create folder or break.
    $_fileDestination = $($ZipDestinationDirectory)
    $_testDestinationBool = Test-Path -Path $($_fileDestination)
    if ($_testDestinationBool -eq $FALSE) {
        $_input = Read-Host "Path: ($($_fileDestination)) doesn't exist, should I make it? (Y/N)"
        if ($_input -eq "Y") {
            mkdir $_fileDestination
            Write-Verbose "Creating destination directory"
        }
        else {
            Break
            Write-Verbose "Break"
        }
    }

    #Loop through all files in source folder and if property "LastWriteTime" 
    #is older than "FindFilesOlderThanDays" add the file to archive
    Set-Location $_fileSource #Change dir to source
    #Loop through each file within the specified source folder
    foreach ($_file in $_allFilesArr) {
        #If a file was last written to after the set number of days to check for then proceed
        if ($_file.LastWriteTime -le $($_fileAgeDate)) {
            Write-Verbose "'$($_file.Name)' was last written to over $($FindFilesOlderThanDays) days ago, attempting to Zip..."
            #If the user has requested the ZIP'ed files be grouped by day then they are, if not they are Zip'ed individually
            if ($GroupFilesByDay) {
                Write-Verbose "Grouping '$($_file.Name)' by day last written to"
                $_fileGroupDay = $($_file.LastWriteTime).ToString("dd.MM.yyyy")
                #ZIP Em by date last written to
                $_zipDestination = "$($_fileDestination)\$($_fileGroupDay).Zip"
                $_fileSource = "$($_file.Name)"
                Compress-Archive -Path $($_fileSource) -CompressionLevel Optimal -Update -DestinationPath $($_zipDestination)
                Write-Verbose "Compressed: '$($_fileSource)' To: '$($_zipDestination)'"
                $_zipFileCheck = $false
                if ($CleanOnZip) {
                    $_zipFileCheck = Test-Path -Path $($_zipDestination)
                    #If the ZIP file is found the source files will be deleted
                    if ($_zipFileCheck) {
                        Remove-Item -Path $($_fileSource)
                        Write-Verbose "'$($_fileGroupDay).Zip' was found in '$($_zipDestination)', removing source file"
                    }#End of Zip File Check
                    else {
                        Write-Verbose "'$($_fileGroupDay).Zip' was not found in '$($_zipDestination)', Not deleting source file"
                    }#End of Zip File Check Else
                }#End of Clean On Zip
            }#End of Group Files By Days
            else {            
                #ZIP Em individualy 
                $_zipDestination = "$($_fileDestination)\$($_file.Name).Zip"
                $_fileSource = "$($_file.Name)"
                Compress-Archive -Path $($_fileSource) -CompressionLevel Optimal -Update -DestinationPath $($_zipDestination)
                Write-Verbose "Compressed: '$($_fileSource)' To: '$($_zipDestination)'"
                $_zipFileCheck = $false
                if ($CleanOnZip) {
                    $_zipFileCheck = Test-Path -Path $($_zipDestination)
                     Write-Verbose "Clean on ZIP = $True, Checking for file..."
                    #If the ZIP file is found the source files will be deleted
                    if ($_zipFileCheck) {
                        Remove-Item -Path $($_fileSource)
                        Write-Verbose "The Zip File was found in the destination directory, removing source file"
                    }#End of Zip File Check
                    else {
                        Write-Verbose "The Zip File was not found in the destination directory, Not deleting source file"
                    }#End of Zip File Check Else
                }#End of $CleanOnZip
            }#End of Group By Days else
        }#End of Last Write Time check
    }#End of foreach
    Set-Location $_pwd # Change dir to last working directory before script was run 
    Write-Verbose "Script completed in $($stopwatch.Elapsed)"
    $stopwatch.Stop()
}#End of function
