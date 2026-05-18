# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Webtrees is an online genealogy application (v2.2.6-dev) for managing GEDCOM family tree data. It is a custom PSR-compliant PHP framework (not Laravel/Symfony), licensed GPL-3.0. Namespace: `Fisharebest\Webtrees`.

## Common Commands

### Development Server
```bash
php -S localhost:8080
```

### Testing
```bash
composer webtrees:test                # Run all tests (PHPUnit)
vendor/bin/phpunit                    # Run all tests directly
vendor/bin/phpunit tests/app/AgeTest.php                    # Run single test file
vendor/bin/phpunit --filter testMethodName                   # Run single test method
composer webtrees:coverage            # Coverage report (requires XDEBUG)
```

### Code Quality
```bash
composer webtrees:phpcs               # PHP_CodeSniffer (PSR-12, excludes line length)
composer webtrees:phpstan             # PHPStan static analysis (level 2)
composer webtrees:pre-commit-hook     # Run all pre-commit checks
composer ci                           # Full CI pipeline (validate, install, test, phpcs, phpstan)
```

### Frontend Assets
```bash
npm install                           # Install JS dependencies
npm run production                    # Build minified CSS/JS
npm run development                   # Build unminified CSS/JS
npm run watch                         # Watch and rebuild on changes
```

### Build
```bash
composer webtrees:build               # Create distribution webtrees.zip
```

### Local Database (Docker)
```bash
docker-compose up -d                  # Start MySQL 8.0 on port 3306
```

## Architecture

### Request Lifecycle
Entry point is `index.php` which calls `Webtrees::new()->run()`. Requests flow through a PSR-15 middleware pipeline defined in `app/Webtrees.php::MIDDLEWARE` (24 layers including error handling, database, sessions, routing, modules). The router (Aura Router) dispatches to RequestHandler classes in `app/Http/RequestHandlers/`.

### Key Architectural Patterns
- **PSR compliance**: PSR-4 autoloading, PSR-7 HTTP messages, PSR-11 DI container (`app/Container.php` with reflection-based autowiring), PSR-15 middleware/handlers
- **Routing**: Defined in `app/Http/Routes/WebRoutes.php` and `ApiRoutes.php`, dispatched to handler classes in `app/Http/RequestHandlers/`
- **Database**: Uses Illuminate Database (Laravel's query builder) standalone via `app/DB.php`. Supports MySQL, PostgreSQL, SQLite, SQL Server. Migrations in `app/Schema/` (Migration0 through Migration43)
- **Templating**: Custom view system (`app/View.php`) using `.phtml` templates in `resources/views/`. Namespace separator is `::`. Not Blade.
- **DI Container**: Custom PSR-11 container in `app/Container.php` with reflection-based autowiring
- **Registry pattern**: `app/Registry.php` holds factories (28+) for creating domain objects

### Module System
Interface-based plugin architecture. Modules implement `ModuleInterface` (base: `AbstractModule`) plus specialized interfaces like `ModuleChartInterface`, `ModuleBlockInterface`, `ModuleReportInterface`, `ModuleThemeInterface`, etc. (~20 interfaces). Built-in modules live in `app/Module/`. Third-party modules go in `modules_v4/`.

### Domain Model
Core GEDCOM record types: `Individual`, `Family`, `Source`, `Repository`, `Media`, `Note`, `SharedNote` (all extend `GedcomRecord`). GEDCOM parsing/elements in `app/GedcomElements/`, `app/GedcomFilters/`, `app/GedcomRecords/`. The `Tree` class represents a family tree.

### Services Layer
Business logic lives in `app/Services/` (e.g., `GedcomImportService`, `GedcomExportService`, `TreeService`, `ModuleService`).

## Coding Standards

- **PSR-12** extended coding style (enforced via PHP_CodeSniffer, line length excluded)
- **PHPStan level 2** with bleeding edge rules
- PHP 8.3+ required (tested on 8.3, 8.4, 8.5, 8.6)
- JavaScript follows semistandard style
- Frontend stack: Bootstrap 5, jQuery, Leaflet (maps), DataTables, FontAwesome, Laravel Mix (webpack)

## Configuration

- Site config stored in `data/config.ini.php` (read by `ReadConfigIni` middleware)
- Database schema version tracked via `Webtrees::SCHEMA_VERSION` (currently 45)
- Translations managed at translate.webtrees.net, stored in `resources/lang/`
