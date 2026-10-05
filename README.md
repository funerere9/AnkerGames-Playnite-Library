# How to install
First: download ankergames_library.txt by clicking download raw file
Second: open playnite and click on the controller in the top left corner, then hover over extensions and click on Interactive SDK PowerShell
Third: Copy and paste the script below
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

Once done all the games will be under downloads category (you will have to switch over from library to category), from there i recommend to select detail view and then simply click on AnkerGames Download Page.
