# test-claude-session-manager

## Overview
This repository is designed to test Claude Code session manager collaboration workflows in real mode (non-mock). It validates session persistence, file creation, and workflow coordination capabilities.

## Test Workflow

### Timestamped Test Files
The repository uses timestamped test files to verify collaboration workflow functionality:

- **File Naming Pattern**: `test-<timestamp>`
- **Timestamp Format**: ISO 8601 format (YYYY-MM-DDTHH:MM:SS) in UTC
- **Location**: Repository root directory
- **Purpose**: Validate session management, file creation, and git integration

### Creating Test Files
Test files are created automatically by Claude Code sessions to demonstrate:
1. Successful file creation with consistent naming patterns
2. Accurate timestamp generation at creation time
3. Session context maintenance across operations
4. Git change tracking integration
5. Real workflow execution without mock implementations

### Example
A typical test file follows this pattern:
```
test-2025-12-15T18:25:51
```

This represents a test file created on December 15, 2025 at 18:25:51 UTC.

## Usage
This repository is part of the Claude Code session manager testing suite. Test files are created during collaboration workflow validation and demonstrate proper session management capabilities.
