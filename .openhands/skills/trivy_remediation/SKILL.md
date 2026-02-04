---
name: trivy_remediation
type: procedural
version: 1.1.0
agent: CodeActAgent
triggers:
  - vulnerability scan
  - trivy scan
  - fix vulnerabilities
  - security remediation
---

# Multi-Ecosystem Vulnerability Remediation Skill

This skill provides automated security vulnerability scanning and remediation for projects across multiple languages and frameworks. It performs code changes (not just dependency updates), validates builds, and tracks all modifications with comprehensive reporting.

## Key Capability: Code Changes + Dependency Updates

Unlike simple version bump tools, this skill:
- Updates vulnerable dependencies to fixed versions
- **Applies code migrations** when major versions require API changes
- Handles namespace and import statement migrations
- Rewrites configuration files when needed
- Validates changes with build/test loops
- Tracks and documents ALL modified files

## Vulnerability Categories & Remediation Approaches

### Category 1: Dependency-Only Fixes (No Code Changes)

These vulnerabilities fix via version updates alone:

**Pattern: Backward-Compatible Version Upgrades**
- Vulnerability: Security issues in dependencies that maintain API compatibility
- Fix: Update version in dependency manifest file
- Code Changes: None required – the new version maintains the same public API
- Examples: Bug fixes, security patches, performance improvements in minor/patch versions
- Risk: Low
- Files Affected: Only dependency manifest and lock files updated

### Category 2: Minor API Changes (Light Code Updates)

These require small code adjustments after dependency upgrade:

**Pattern: Security Library Updates (1.x → 2.x)**
- Vulnerability: API changes requiring code refactoring
- Fix: Dependency update + code changes in affected modules
- Examples: YAML parsing security improvements, JWT token handling, serialization patterns
- Files Affected: Service classes using the deprecated APIs
- Risk: Medium
- Changes Required:
  - Updated imports (new namespaces, new packages)
  - Constructor/initialization patterns changed
  - Method calls refactored (parameter order, naming conventions)
  - Security patterns applied (e.g., SafeConstructor instead of default constructor)

### Category 3: Major Framework Migrations (Extensive Code Rewrites)

These require significant API changes and structural modifications:

**Pattern: Namespace/Module Migrations**
- Vulnerability: Deprecated namespaces or module structures
- Fix: Framework update + complete refactoring of affected code
- Examples: Enterprise Library upgrades, EE to SE transitions, major version framework updates
- Files Affected: Multiple across codebase (controllers, entities, services, configuration)
- Risk: High (requires thorough testing)
- Changes Required:
  - Update all imports/includes to new namespace
  - Refactor all class inheritance patterns (adapters, base classes removed)
  - Update all configuration classes (annotation-based or builder patterns)
  - Refactor method call patterns throughout codebase
  - Update configuration file syntax if needed

**Pattern: API/Behavior Breaking Changes**
- Vulnerability: Core library rewrites in major versions
- Fix: Dependency update + rewrite of all affected integration points
- Examples: File upload handlers, form processing, authentication handlers
- Files Affected: Controllers, handlers, utility classes
- Risk: High (API completely different)
- Changes Required:
  - Replace deprecated method calls with new API
  - Update initialization patterns (constructors, builders, factories)
  - Migrate from imperative to declarative patterns where applicable
  - Update error handling if exception types changed
  - Update I/O patterns (streaming, buffering, etc.)

## Remediation Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ 0. BACKUP: Create full project backup (pre-changes)         │
│    Output: remediation-backup-YYYYMMDD-HHMMSS.tar.gz        │
│    EXECUTED IMMEDIATELY - BEFORE ANY OTHER STEPS            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 1. SCAN: Run Trivy baseline (HIGH/CRITICAL only)           │
│    Output: trivy.before.json (EXACT NAME - NO VARIATIONS)   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. ANALYZE: Categorize vulnerabilities by fix complexity    │
│    - Category 1: Dependency-only (low risk)                 │
│    - Category 2: Light code changes (medium risk)           │
│    - Category 3: Major rewrites (high risk)                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. FIX: Apply remediation in phases (NO CONFIRMATION)      │
│    Phase 1: Dependency updates (manifest files)            │
│    Phase 2: Code changes (imports, API calls, config)      │
│    Phase 3: Framework-specific rewrites (configs)          │
│    CHANGES APPLIED DIRECTLY TO REPOSITORY                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. BUILD/TEST: Validate changes with bounded attempts       │
│    - Run: Build/test commands (specific to ecosystem)       │
│    - Max attempts: 10 (stagnation detection)                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. VALIDATE: Re-scan with Trivy to confirm fixes            │
│    Output: trivy.after.json (EXACT NAME - NO VARIATIONS)    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. REPORT: Generate comprehensive remediation report        │
│    Output: TRIVY_REMEDIATION_REPORT.md (EXACT NAME)        │
│    - CVE counts (before/after from JSON files)              │
│    - Fixed vulnerabilities list                             │
│    - Remaining HIGH/CRITICAL items                          │
│    - MODIFIED FILES (complete traceability)                 │
│    - Next steps (recommendations)                           │
└─────────────────────────────────────────────────────────────┘
```

## Direct Repository Changes

**YES** - This skill makes actual changes directly to your repository WITHOUT asking for confirmation:

### Execution Model
-  Backup is created FIRST (before any modifications)
-  Code changes are applied DIRECTLY to the repository
-  No confirmation prompts or manual approval needed
-  All changes are automatic and immediate
-  Build/test validation runs automatically
-  Report is generated automatically after completion

### Files Modified in Repository

1. **Dependency Manifest & Lock Files**
   - Version numbers updated to patch/minor versions
   - Lock files regenerated (language-specific formats)
   - All dependency changes traced and documented

2. **Source Code Files**
   - Namespace/import migrations applied where needed
   - API calls refactored for new versions
   - Configuration classes rewritten when APIs change
   - Code patterns updated for security requirements

3. **Configuration Files**
   - Complete rewrites when major framework versions change
   - New patterns adopted based on framework requirements

### Backup & Reversibility

-  Complete backup created IMMEDIATELY at start (`remediation-backup-YYYYMMDD-HHMMSS.tar.gz`)
-  All modified files documented in final report
-  No automatic Git commits (manual review before pushing)
-  Full rollback capability if needed: `tar -xzf remediation-backup-YYYYMMDD-HHMMSS.tar.gz`

---

## Output Artifacts Generated

This skill produces THREE primary output files in your project directory:

### 1. `trivy.before.json` (Baseline Vulnerability Scan)
**Created:** At the START of remediation (Step 1)

**Contents:** Complete Trivy vulnerability report (JSON format), filtered to HIGH and CRITICAL severities only, pre-remediation CVE list and counts, package inventory with installed versions.

**Purpose:** Establishes baseline for before/after comparison and documents initial vulnerability state.

### 2. `trivy.after.json` (Post-Remediation Vulnerability Scan)
**Created:** After fixes applied & build succeeds (Step 6)

**Contents:** Complete Trivy vulnerability report (JSON format), filtered to HIGH and CRITICAL severities only, post-remediation CVE list and counts, package inventory with updated versions.

**Purpose:** Validates remediation effectiveness by comparing against `trivy.before.json` to show fixed vs. remaining CVEs.

### 3. `TRIVY_REMEDIATION_REPORT.md` (MANDATORY - Comprehensive Summary Report)
**Created:** AFTER both scans complete (Step 6)
**Filename:** Must be exactly `TRIVY_REMEDIATION_REPORT.md` (no variations)
**Location:** Project root directory
**Contents Include:**
- Executive Summary (vulnerabilities fixed, remaining, reduction %)
- Fixed Vulnerabilities Table (CVE ID, package, version changes, severity)
- Remaining HIGH/CRITICAL Vulnerabilities Table (CVE ID, current version, suggestions)
- **Modified Files Section** (detailed file-by-file list of changes)
- Remediation Actions Performed (step-by-step what was done)
- Build & Test Results (pass/fail, number of attempts, test counts)
- Vulnerability Reduction Visualization (before/after charts)
- Rollback Instructions (how to restore from backup if needed)
- Next Steps & Recommendations (what to do with remaining CVEs)

**Example Report Structure:**
```
# Trivy Vulnerability Remediation Report

Date: 2026-02-03
Project: /path/to/project
Ecosystem: Auto-detected by Trivy
Build Status: SUCCESS

## Executive Summary
- Vulnerabilities Fixed: 8 CVEs
- Remaining HIGH/CRITICAL: 6 CVEs
- Reduction: 57.1% (14 → 6)

## Fixed Vulnerabilities
[Table with CVE ID, Package, From→To, Severity]

## Remaining HIGH/CRITICAL Vulnerabilities
[Table with CVE ID, Package, Current Version, Suggested Fix]

## Modified Files

**Dependency Updates:**
- Dependency manifest file (versions updated)

**Code Changes:**
- Authentication/security modules
- Configuration modules
- Service/utility modules
- [... total files modified, total lines changed]

## Rollback Instructions
tar -xzf remediation-backup-YYYYMMDD-HHMMSS.tar.gz

## Next Steps
1. Review changes in modified files
2. Run full test suite
3. Address remaining CVEs
```

---

## Modified Files Tracking (NEW in v1.1.0)

Every remediation report includes a **"Modified Files"** section documenting:

### Dependency File Changes
- Dependency manifest files (specific format depends on ecosystem)
- Lock/freeze files (for reproducible builds)
- Configuration files (if version pinning needed)

### Code Changes Documentation
- Source files modified for API/namespace changes
- Configuration files rewritten (security configs, middleware, etc.)
- Import statements updated (namespace migrations)
- Function/method calls refactored (new APIs, parameter orders)

### Example Report Section

```markdown
## Modified Files

**Dependency Updates:**
- Dependency manifest file
  - security-library: 1.0.0 → 2.5.0 (fixes CVE-2021-XXXXX)
  - framework: 5.0.0 → 6.1.0 (fixes CVE-2022-XXXXX)
  - utility-lib: 1.5.0 → 2.0.0 (API-breaking upgrade)

**Code Changes:**
- Authentication/security modules
  - Updated: import statements (new namespaces)
  - Updated: API calls (new methods, parameters)

- Configuration modules
  - REWRITTEN: Framework migration applied
  - Removed: Deprecated patterns
  - Added: New API patterns

- Service/utility modules
  - Updated: Security handling patterns
  - Added: Configuration options for security requirements

**Backup:**
- remediation-backup-YYYYMMDD-HHMMSS.tar.gz

**Summary:**
- Fixed: 8 CVEs
- Remaining: 6 CVEs (need manual review)
- Files Modified: 5
- Build Status: SUCCESS
```

## Safety Guarantees

-  Complete backup before any changes (FIRST step)
-  Conservative (patch/minor) updates by default
-  Stagnation detection prevents infinite loops (max 10 attempts)
-  Only HIGH/CRITICAL vulnerabilities targeted
-  Build validation required for success
-  No automatic Git commits (manual review before pushing)
-  Detailed logging of all changes
-  Full rollback capability via backup

## Mandatory Output Files (STRICT NAMING)

ALL output files must use these EXACT names with NO variations:

| File Name | Purpose | Timing |
|-----------|---------|--------|
| `trivy.before.json` | Baseline scan (HIGH/CRITICAL only) | Created at START (Step 1) |
| `trivy.after.json` | Post-remediation scan (HIGH/CRITICAL only) | Created after fixes (Step 5) |
| `TRIVY_REMEDIATION_REPORT.md` | Comprehensive summary report | Created at END (Step 6) |
| `remediation-backup-YYYYMMDD-HHMMSS.tar.gz` | Full project backup (pre-changes) | Created IMMEDIATELY (Step 0) |

**Non-Compliance:** Any deviation from these exact file names is a failure. Alternative naming schemes are NOT acceptable.

## Ecosystem-Specific Build & Test Commands

| Ecosystem | Build Command | Test Command |
|-----------|---------------|--------------|
| Java (Maven) | `mvn clean compile` | `mvn test` |
| Java (Gradle) | `./gradlew build` | `./gradlew test` |
| Node.js | `npm install` | `npm test` |
| Python (pip) | `pip install -r requirements.txt` | `pytest` |
| Python (Poetry) | `poetry install` | `poetry run pytest` |
| Go | `go mod download` | `go test ./...` |
| Ruby | `bundle install` | `bundle exec rspec` |
| PHP | `composer install` | `phpunit` |
| Rust | `cargo build` | `cargo test` |

---

*Multi-Ecosystem Vulnerability Remediation Skill v1.1.0*
*Completely Generic - No ecosystem-specific examples*
*Features: Dependency updates + code migrations + build validation + comprehensive file tracking*

