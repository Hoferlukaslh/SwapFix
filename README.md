# SwapFix
Fichier permettant de fixer la taille du swap windows

```powershell
#Requires -RunAsAdministrator

$TailleFixeMo = 8192
$PageFilePath = "C:\pagefile.sys"

try {
    # Désactiver la gestion automatique
    $ComputerSystem = Get-CimInstance Win32_ComputerSystem
    if ($ComputerSystem.AutomaticManagedPagefile) {
        Set-CimInstance -InputObject $ComputerSystem -Property @{AutomaticManagedPagefile = $false} -ErrorAction Stop
    }

    # Recherche du fichier d'échange cible existant
    $ExistingPageFile = Get-CimInstance Win32_PageFileSetting | Where-Object {
        $_.Name -eq $PageFilePath
    }

    if ($ExistingPageFile) {
        # Modification si existant
        $null = Set-CimInstance -InputObject $ExistingPageFile -Property @{
            InitialSize = $TailleFixeMo
            MaximumSize = $TailleFixeMo
        } -ErrorAction Stop
    } else {
        # Supprimer les autres (ex: si le système avait créé un pagefile sur D:)
        Get-CimInstance Win32_PageFileSetting | Where-Object {
            $_.Name -ne $PageFilePath
        } | Remove-CimInstance -ErrorAction Stop

        # Création du nouveau pagefile
        $null = New-CimInstance -ClassName Win32_PageFileSetting -Property @{
            Name = $PageFilePath
            InitialSize = $TailleFixeMo
            MaximumSize = $TailleFixeMo
        } -ErrorAction Stop
    }

    Write-Host "Pagefile fixé à $TailleFixeMo Mo. Un redémarrage de la machine est requis." -ForegroundColor Green
}
catch {
    Write-Error "Erreur lors de la configuration du Pagefile : $_"
}
```
