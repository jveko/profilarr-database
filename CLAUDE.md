# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Dictionarry Database repository - the core database powering the Profilarr ecosystem for building sophisticated Radarr/Sonarr custom formats and quality profiles. It serves both the Dictionarry website (dictionarry.dev) and provides structured data for the Profilarr configuration management application.

## Key Development Commands

### Bundle Generation (Primary Build Command)
```bash
python scripts/bundle.py
```
This is the main build process that generates the JSON bundles consumed by applications:
- Loads all regex patterns from `/regex_patterns`
- Resolves pattern references in custom formats (replaces pattern names with actual regex)
- Bundles custom_formats, profiles, regex_patterns, group_tiers, dev_logs, and wiki
- Outputs processed JSON files to `/bundles` directory
- Creates timestamped version.json

### Tier Creator Script (Release Group Management)
```bash
python scripts/tierCreator.py <json_file> --resolution <resolution> --type <type> [--dry-run]
```
Generates custom formats and regex patterns from tier data files:
- `<json_file>`: Path to JSON file in `/group_tiers` (e.g., `group_tiers/2160p Quality.json`)
- `--resolution`: SD, 720p, 1080p, 2160p
- `--type`: Quality, Balanced
- `--dry-run`: Preview changes without writing files

Example:
```bash
python scripts/tierCreator.py group_tiers/2160p\ Quality.json --resolution 2160p --type Quality
```

## Architecture & Data Flow

### Core Components

**Regex Patterns** (`/regex_patterns/`):
- Individual YAML files defining regex patterns for release groups, streaming services, codecs, etc.
- Include comprehensive test cases with expected matches/non-matches
- Pattern examples: `DON.yml`, `Netflix.yml`, `Dolby Vision.yml`, `x265.yml`

**Custom Formats** (`/custom_formats/`):
- YAML files defining Radarr/Sonarr custom formats that reference regex patterns
- Include conditions (resolution, source, release_group, release_title)
- Support complex logic with negate, required flags
- Examples: `1080p Quality Tier 1.yml`, `Netflix.yml`, `Dolby Vision.yml`

**Quality Profiles** (`/profiles/`):
- Complete scoring systems for different use cases
- Define custom format scores, quality groups, upgrade behavior
- Three main types: Quality (transparency-focused), Balanced (WEB-DL-focused), Remux (lossless-focused)
- Examples: `1080p Quality.yml`, `2160p Balanced.yml`, `2160p Remux.yml`

**Group Tiers** (`/group_tiers/`):
- JSON metadata defining release group performance tiers based on metrics
- Uses Golden Popcorn Performance Index (GPPi) for 1080p groups
- Uses Encode Efficiency Index (EEi) for 2160p groups at 55% target ratio
- Automated tier generation using K-means clustering

### Template System
**Templates** (`/templates/`):
- `releaseGroup.yml`: Template for generating regex patterns for release groups
- `groupTier.yml`: Template for generating tiered custom formats

### Bundle Resolution Process
1. `bundle.py` loads all regex patterns into memory
2. For custom formats, it resolves pattern references:
   - Finds conditions with `type: release_title/release_group/edition`
   - Replaces pattern names with actual regex from `/regex_patterns`
3. Outputs fully resolved data as JSON bundles for application consumption

### Scoring Philosophy
- **Quality Profiles**: Target transparent encodes using GPPi/EEi metrics
- **Balanced Profiles**: Prioritize WEB-DLs with encode fallbacks
- **Remux Profiles**: Focus on lossless audio/video features over release groups
- Negative scores (-9999) block unwanted content (wrong resolution, codecs, etc.)

## File Structure & Naming Conventions

### Custom Formats
- **Tiered Groups**: `{resolution} {type} Tier {number}.yml` (e.g., `1080p Quality Tier 1.yml`)
- **Features**: `{feature}.yml` (e.g., `Dolby Vision.yml`, `HDR10+.yml`)
- **Sources**: `{service}.yml` or `{source}.yml` (e.g., `Netflix.yml`, `Blu-ray Remux.yml`)

### Regex Patterns
- **Release Groups**: `{group}.yml` (e.g., `DON.yml`, `DEPTH.yml`)
- **Features**: Match custom format names (e.g., `Dolby Vision.yml`, `x265.yml`)

### Quality Profiles
- **Format**: `{resolution} {type}.yml` (e.g., `1080p Quality.yml`, `2160p Balanced.yml`)

## Testing & Validation

### Regex Pattern Testing
Each regex pattern includes test cases:
```yaml
tests:
- expected: true
  id: 1
  input: "Sample.Movie.2024.1080p.NF.WEB-DL.DDP5.1.H.264-NTb"
  matchSpan: {start: 34, end: 36}
  matchedContent: "NF"
```

### Custom Format Testing
Custom formats can include test cases validating condition logic:
```yaml
tests:
- input: "Movie.2024.1080p.x265.HEVC-GROUP"
  expected: false  # Should not match due to x265 negation
```

### Validation Workflow
1. Use `--dry-run` with scripts to preview changes
2. Run `bundle.py` to verify successful bundle generation
3. Test with Profilarr application for integration validation
4. Validate regex patterns match expected release names

## Commit Message Standards

Format: `type(component): Description`

**Types:**
- `create`: New components/systems (e.g., new profiles, major features)
- `add`: Adding entries to existing systems (e.g., new release groups)
- `tweak`: Fine-tuning existing components (e.g., score adjustments)
- `fix`: Corrections and bug fixes (e.g., regex pattern fixes)

**Components:**
- `format`: Custom format changes
- `regex`: Regex pattern changes  
- `profile`: Quality profile changes

**Examples:**
- `create(profile): 2160p Remux quality profile`
- `add(format): ZoroSenpai as Tier 2 Quality group`
- `tweak(profile): Adjust streaming service scores for balance`
- `fix(regex): Dolby Vision pattern excludes SDR/HLG variants`

## Key Metrics & Algorithms

### Golden Popcorn Performance Index (GPPi)  
- Used for 1080p release group quality assessment
- Considers transparency, consistency, and reliability
- Groups automatically clustered into 5 tiers using K-means

### Encode Efficiency Index (EEi)
- Used for 2160p release group quality assessment  
- Targets 55% compression ratio for transparency evaluation
- Enables codec-agnostic quality assessment
- Eliminates need for codec parsing as quality proxy

## Development Workflow

1. **Pattern Development**: Create/modify regex patterns with comprehensive tests
2. **Format Development**: Build custom formats referencing patterns
3. **Profile Development**: Compose scoring systems using custom formats
4. **Testing**: Use dry-run modes and bundle generation for validation
5. **Integration**: Test with Profilarr for end-to-end validation

## Branches & Releases

- **stable**: Production-ready, thoroughly tested configurations
- **dev**: Latest updates pending verification
- **scoring-refactor**: Experimental branch with major scoring changes

The repository follows a data-driven approach where release group performance is measured algorithmically rather than manually curated, enabling scalable and objective quality assessment.