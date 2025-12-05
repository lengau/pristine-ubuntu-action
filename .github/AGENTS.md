# Agent Instructions for pristine-ubuntu-action

This document provides instructions for AI coding agents, automated bots, and agentic workflows working with this repository.

## Repository Overview

**pristine-ubuntu-action** is a GitHub Action that removes bloat from GitHub's hosted Ubuntu runners, freeing up 15-20 GiB of disk space. It's a composite action written primarily in YAML with shell scripts.

### Key Information
- **Type**: GitHub Composite Action
- **Purpose**: Clean up Ubuntu runners by removing unnecessary pre-installed tools
- **Primary Language**: YAML (action definition) + Bash (cleanup scripts)
- **Supported Platforms**: Ubuntu 22.04, Ubuntu 24.04
- **Must Test On**: Actual GitHub-hosted runners (local testing is limited)

## Repository Structure

```
.
├── action.yaml              # Main action definition (CRITICAL FILE)
├── README.md                # User documentation
├── .github/
│   ├── workflows/
│   │   ├── test_action.yaml # Test workflow
│   │   └── debug.yaml       # Debug workflow
│   └── AGENTS.md           # This file
└── LICENSE
```

## Getting Started for Agents

### 1. Understanding the Action

Before making changes, read these files in order:
1. `README.md` - Understand usage and purpose
2. `action.yaml` - Review the action structure and cleanup steps
3. `.github/workflows/test_action.yaml` - See how the action is tested

### 2. Key Concepts

**Composite Action Structure:**
- The action is defined in `action.yaml`
- Uses `runs.using: 'composite'` with multiple steps
- Each step runs shell commands to remove specific tools/packages

**Cleanup Pattern:**
- Each cleanup step targets a specific tool (Android SDK, browsers, etc.)
- Steps include error handling for cases where packages aren't installed
- Large deletions use background processes (`&`) for efficiency
- Steps often reference upstream installation scripts from `actions/runner-images`

**Configuration Options:**
- Input parameters use `keep-*` naming convention (e.g., `keep-android`)
- Users specify `keep-<tool>: 'true'` to preserve specific tools
- Default behavior removes everything unless explicitly kept

## Common Tasks

### Adding a New Cleanup Step

When adding a new cleanup step to remove additional bloat:

1. **Research the tool:**
   - Find how it's installed in [actions/runner-images](https://github.com/actions/runner-images/tree/main/images/ubuntu/scripts/build)
   - Note the installation paths and package names

2. **Add input parameter** (if optional):
   ```yaml
   inputs:
     keep-newtool:
       description: Keep NewTool
       default: ''
   ```

3. **Add cleanup step**:
   ```yaml
   - name: Remove NewTool
     # https://github.com/actions/runner-images/blob/main/images/ubuntu/scripts/build/install-newtool.sh
     shell: bash
     id: newtool
     if: ${{ inputs.keep-newtool == '' }}
     run: |
       echo "Removing NewTool"
       # Your cleanup commands here
       sudo rm -rf /path/to/newtool &
   ```

4. **Add error handling for package removal**:
   ```bash
   if ! sudo apt-get --yes purge package-name; then
     if ! apt list --installed 2>/dev/null | grep -q '^package-name/'; then
       echo "package-name not installed, ignoring error."
     else
       echo "Unexpected error removing package-name" >&2
       exit 1
     fi
   fi
   ```

5. **Update documentation**:
   - Add the new `keep-*` option to README.md
   - Update the configuration section

6. **Add tests**:
   - Update `.github/workflows/test_action.yaml` to test the new option

### Fixing Bugs

1. **Identify the problematic step** in `action.yaml`
2. **Reproduce the issue** (check workflow logs, issue reports)
3. **Apply the fix** with appropriate error handling
4. **Test on GitHub-hosted runner** via workflow run
5. **Verify** no other steps are affected

### Updating Documentation

1. **Keep README.md synchronized** with `action.yaml` inputs
2. **Ensure examples are accurate** and tested
3. **Update version references** if applicable
4. **Check links** to external resources

## Testing Requirements

**CRITICAL**: This action MUST be tested on actual GitHub-hosted Ubuntu runners. Local testing is insufficient.

### Testing Checklist
- [ ] Action completes successfully
- [ ] Disk space is freed up (verify with `df -h`)
- [ ] Required GitHub Actions tools still work (setup-* actions)
- [ ] Configuration options (`keep-*` flags) work as documented
- [ ] Error handling works for missing packages

### How to Test
1. Push changes to a branch
2. Run the test workflow (`.github/workflows/test_action.yaml`)
3. Review workflow logs for errors
4. Verify disk space metrics
5. Ensure subsequent steps in the workflow succeed

## Code Conventions

### YAML Style
- Use 2-space indentation
- Use lowercase with hyphens for names (e.g., `keep-android`)
- Include descriptive step names
- Add comments with links to upstream installation scripts

### Shell Script Style
- Use `bash` for all shell steps
- Echo informative messages for debugging
- Use background processes (`&`) for large deletions when safe
- Always include error handling for `apt-get` operations
- Suppress "not installed" errors, but surface unexpected errors

### Example Error Handling Pattern
```bash
# Only suppress "not installed" error, surface unexpected errors
if ! sudo apt-get --yes purge package-name; then
  if ! apt list --installed 2>/dev/null | grep -q '^package-name/'; then
    echo "package-name not installed, ignoring error."
  else
    echo "Unexpected error removing package-name" >&2
    exit 1
  fi
fi
```

## Important Constraints

### Do NOT Remove
- Git (required by actions/checkout)
- Docker (required by container actions)
- Node.js (required by GitHub Actions runtime)
- Python (required by setup-python and many actions)
- Core system utilities
- GitHub Actions runner dependencies

### Safe to Remove (Current Defaults)
- Android SDK (~15 GiB)
- Browsers (Chrome, Firefox, Edge) and drivers
- Java (Temurin)
- Julia
- Rust (unless `keep-rustup: 'true'`)
- Azure CLI (unless `keep-azure: 'true'`)
- PowerShell (unless `keep-powershell: 'true'`)
- Ansible (unless `keep-ansible: 'true'`)

## Development Workflow

### Standard Process
1. Create a feature branch from `main`
2. Make changes to relevant files
3. Update tests and documentation
4. Push branch and run workflows
5. Review workflow results
6. Create pull request with description of changes
7. Address review feedback
8. Merge after approval

### Branch Naming
Use descriptive names:
- `feature/add-cleanup-<tool>`
- `fix/<issue-description>`
- `docs/<what-updated>`

## Integration with Agentic Workflows

### Using This Action in Agentic Workflows

When creating workflows that use this action, **ALWAYS** make it the first step:

```yaml
steps:
  - name: Pristine Ubuntu
    if: ${{ runner.os == 'Linux' && runner.environment == 'github-hosted' }}
    uses: lengau/pristine-ubuntu-action@v0
    with:
      # Configure based on your needs
      # keep-rustup: 'true'  # Example: keep Rust if needed
```

### Why First Step?
- Frees up disk space before other actions run
- Prevents disk space issues during builds
- Removes tools that won't be used, reducing confusion

### Configuration Examples

**Minimal (remove everything possible):**
```yaml
- uses: lengau/pristine-ubuntu-action@v0
```

**Keep Rust:**
```yaml
- uses: lengau/pristine-ubuntu-action@v0
  with:
    keep-rustup: 'true'
```

**Keep multiple tools:**
```yaml
- uses: lengau/pristine-ubuntu-action@v0
  with:
    keep-rustup: 'true'
    keep-azure: 'true'
```

## Resources and References

### Upstream References
- **Runner Images**: https://github.com/actions/runner-images
- **Installation Scripts**: https://github.com/actions/runner-images/tree/main/images/ubuntu/scripts/build
- **Composite Actions Docs**: https://docs.github.com/en/actions/creating-actions/creating-a-composite-action

### Finding Installation Paths
When adding a new cleanup step, check the upstream installation scripts to find:
- Where files are installed
- What packages are added
- What environment variables are set
- What symbolic links are created

## Security Considerations

### This Action
- Requires `sudo` privileges
- Modifies system packages and files
- Runs on GitHub-hosted runners (not self-hosted)

### Security Review Required For
- New cleanup steps affecting system packages
- Changes to permission requirements
- Modifications to critical system paths

### Safe Operations
- Removing documented packages
- Cleaning well-known tool directories
- Using `apt-get purge` for package cleanup

## Troubleshooting

### Common Issues

**"Package not found" errors:**
- This is expected if a package isn't installed on a particular runner version
- Error handling should suppress these (see error handling pattern above)

**"Permission denied" errors:**
- Ensure commands use `sudo` where needed
- Check file permissions before deletion

**"Disk space still low" after running:**
- Verify background processes completed (`wait` or check with `jobs`)
- Check if additional tools need cleanup
- Review runner images for new bloat

**Tests failing:**
- Check workflow logs for specific errors
- Verify syntax in action.yaml
- Ensure all referenced files exist
- Test locally with `actionlint` if available

## Decision-Making Guidelines

### Autonomous Decisions (No Review Needed)
- Fixing typos in documentation
- Updating comments
- Improving error messages
- Formatting code

### Requires Review
- Adding new cleanup steps
- Modifying existing cleanup logic
- Changing input parameters
- Removing error handling
- Updating version numbers

### Requires Maintainer Approval
- Breaking changes
- Security-sensitive modifications
- Changes to action metadata (name, description)
- License changes

## Questions?

If you're an AI agent and encounter ambiguity:
1. Check the upstream runner-images repository for context
2. Review similar existing cleanup steps for patterns
3. Prefer conservative changes that maintain backward compatibility
4. Include thorough error handling
5. Document your assumptions in pull request descriptions

## Summary for Quick Reference

**Key Files:**
- `action.yaml` - Main action (edit here for cleanup steps)
- `README.md` - User docs (update when changing inputs)
- `.github/workflows/test_action.yaml` - Tests (update when adding features)

**Key Principles:**
1. Always test on GitHub-hosted runners
2. Include error handling for missing packages
3. Document new `keep-*` options
4. Reference upstream installation scripts
5. Use background processes for large deletions
6. Keep backward compatibility

**Remember**: This action helps developers by freeing up disk space on constrained GitHub runners. Changes should focus on removing bloat while preserving tools needed for common development workflows.
