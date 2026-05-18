# Applying changes to a new version

Rebase the `ddp` branch to the `main` of the new version.

# Upgrading process

1. Backup Database
2. Backup current version's code
3. Accept/Reject all pending changes in all trees 
4. Upgrade Webtrees
5. Copy DDP's changed files

# DDP changed files

- app/Http/Routes/WebRoutes.php
- app/Module/ModuleThemeTrait.php
- app/Services/TreeService.php
- resources/views/admin/trees.phtml