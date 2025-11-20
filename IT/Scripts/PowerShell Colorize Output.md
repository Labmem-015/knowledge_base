```PowerShell
#----------------------------------------------------------------------------
# Convert-ErrorLine.ps1
#----------------------------------------------------------------------------
function ColorizeErrors {
    <#
    .SYNOPSIS
        Colours lines that contain the word "error" (case‑insensitive).

    .PARAMETER InputObject
        A single line of text taken from the pipeline.

    .OUTPUTS
        None – writes directly to the host (console).
    #>
    [CmdletBinding()]
    param (
        [Parameter(ValueFromPipeline)]
        [string]$InputObject
    )

    process {
        if ($InputObject -match '(?i): fatal error ') {
            Write-Host $InputObject -ForegroundColor DarkRed
        } elseif ($InputObject -match '(?i): error ') {
            Write-Host $InputObject -ForegroundColor Red
        } elseif ($InputObject -match '(?i): warning ') {
            Write-Host $InputObject -ForegroundColor Yellow
        } else {
            Write-Host $InputObject
        }
    }
}
```