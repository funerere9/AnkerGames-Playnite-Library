This guide outlines how to import a custom game library into the Playnite launcher to easily access game download links.
## How to Setup

   1. Download the library: Download the ankergames_library.txt file by selecting the raw file download option.
   2. Open Playnite Console: Launch Playnite, select the controller icon in the top-left, navigate to Extensions, and choose Interactive SDK PowerShell.
   3. Run the Script: Execute the PowerShell script below to read the text file, create the "Pirated" source and "Download" category, and inject the game entries and links into your Playnite database.

```
# Read your text file from the desktop
$filePath = "$env:USERPROFILE\Desktop\ankergames_library.txt"

if (-not (Test-Path $filePath)) {
    Write-Error "Could not find the file at $filePath. Please check the path."
    return
}

$lines = Get-Content $filePath

# 1. Ensure the "Pirated" source exists (Using lowercase $PlayniteApi)
$sourceName = "Pirated"
$source = $PlayniteApi.Database.Sources | Where-Object { $_.Name -eq $sourceName }
if (-not $source) {
    $source = New-Object Playnite.SDK.Models.GameSource -ArgumentList $sourceName
    $PlayniteApi.Database.Sources.Add($source)
}
$sourceId = $source.Id

# 2. Ensure the "Download" category exists (Using lowercase $PlayniteApi)
$categoryName = "Download"
$category = $PlayniteApi.Database.Categories | Where-Object { $_.Name -eq $categoryName }
if (-not $category) {
    $category = New-Object Playnite.SDK.Models.Category -ArgumentList $categoryName
    $PlayniteApi.Database.Categories.Add($category)
}
$categoryId = $category.Id

# Loop through each line and inject it perfectly into Playnite's engine
foreach ($line in $lines) {
    if ($line -match "(.+?)\s*\|\s*(http.+)") {
        $name = $Matches[1].Trim()
        $url = $Matches[2].Trim()
        
        # Create a clean game object
        $game = New-Object Playnite.SDK.Models.Game -ArgumentList $name
        $game.SourceId = $sourceId
        
        # Assign the "Download" category Guid to the game
        $categoryCollection = New-Object System.Collections.ObjectModel.ObservableCollection[Guid]
        $categoryCollection.Add($categoryId)
        $game.CategoryIds = $categoryCollection
        
        # Format and attach the custom clickable link button
        $link = New-Object Playnite.SDK.Models.Link -ArgumentList "AnkerGames Download Page", $url
        $linksCollection = New-Object System.Collections.ObjectModel.ObservableCollection[Playnite.SDK.Models.Link]
        $linksCollection.Add($link)
        $game.Links = $linksCollection
        
        # Commit natively to your active database (Using lowercase $PlayniteApi)
        $PlayniteApi.Database.Games.Add($game)
        Write-Host "Successfully Imported: $name (Category: Download)" -ForegroundColor Cyan
    }
}

Write-Host "Success! All games imported into 'Download' category!" -ForegroundColor Green
```

## How to Get to Download Page
Once the script has finished running, your imported games will appear under the Download category:

* 
* Change your Playnite library grouping from Library to Category.
* Switch to Detail View and click on AnkerGames Download Page to access the corresponding download link.

   ONCE FINISHED YOU MAY DELETE THE LIBRARY FILE
