Following this tutorial will make it where you can easily find (almost)any pc game you want to download from your playnite launcher.

# How to Setup

1. **Download the library:** Download `ankergames_library.txt` by clicking the "Download raw file" button above.
2. **Open Playnite Console:** Open Playnite, click on the controller icon in the top-left corner, hover over **Extensions**, and click on **Interactive SDK PowerShell**.
3. **Run the Script:** Copy and paste the PowerShell script.
```powershell
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

# How to Get to Download Page

Once done, all the games will be under the **Download** category. 
* Switch your Playnite library grouping from **Library** to **Category**.
* Select **Detail View** and simply click on **AnkerGames Download Page** to open the link this will take you to the download page.
