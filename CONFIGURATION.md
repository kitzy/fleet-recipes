# FleetImporter Configuration Guide

The FleetImporter processor supports two methods of configuration: **AutoPkg preferences** (recommended) and **environment variables** (alternative).

## Configuration Priority

The processor checks for configuration values in this order:

1. **Environment variables** (highest priority)
2. **AutoPkg preferences** (com.github.autopkg plist)
3. **Recipe input variables** (lowest priority)

## Method 1: AutoPkg Preferences (Recommended)

This is the most AutoPkg-native way to configure the processor. Values are stored in the `com.github.autopkg` preference domain and persist across sessions.

### Setting Preferences

```bash
# Fleet API Configuration (Direct Mode)
defaults write com.github.autopkg FLEET_API_BASE "https://fleet.example.com"
defaults write com.github.autopkg FLEET_API_TOKEN "your-fleet-api-token"
defaults write com.github.autopkg FLEET_TEAM_ID -int 1

# AWS Configuration (GitOps Mode)
defaults write com.github.autopkg AWS_ACCESS_KEY_ID "your-access-key-id"
defaults write com.github.autopkg AWS_SECRET_ACCESS_KEY "your-secret-key"
defaults write com.github.autopkg AWS_DEFAULT_REGION "us-east-1"
defaults write com.github.autopkg AWS_S3_BUCKET "your-fleet-packages-bucket"
defaults write com.github.autopkg AWS_CLOUDFRONT_DOMAIN "d1234567890abc.cloudfront.net"

# GitOps Configuration
defaults write com.github.autopkg FLEET_GITOPS_REPO_URL "https://github.com/org/fleet-gitops.git"
defaults write com.github.autopkg FLEET_GITOPS_GITHUB_TOKEN "ghp_yourtoken"
defaults write com.github.autopkg FLEET_GITOPS_SOFTWARE_DIR "lib/macos/software"
defaults write com.github.autopkg FLEET_GITOPS_TEAM_YAML_PATH "teams/team-name.yml"
```

### Reading Preferences

```bash
# View all AutoPkg preferences
defaults read com.github.autopkg

# View a specific preference
defaults read com.github.autopkg AWS_S3_BUCKET

# Delete a preference
defaults delete com.github.autopkg AWS_SECRET_ACCESS_KEY
```

### Preference File Location

Preferences are stored in:
```
~/Library/Preferences/com.github.autopkg.plist
```

## Method 2: Environment Variables (Alternative)

Environment variables take precedence over AutoPkg preferences. This is useful for:
- CI/CD environments
- Temporary overrides
- Non-macOS systems (where AutoPkg preferences aren't available)

### Using a .env File

1. Copy the example file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` with your actual values:
   ```bash
   vim .env
   ```

3. Load environment variables before running AutoPkg:
   ```bash
   export $(grep -v '^#' .env | xargs)
   autopkg run Google/GoogleChrome.fleet.recipe.yaml
   ```

### Direct Export

```bash
export FLEET_API_BASE="https://fleet.example.com"
export FLEET_API_TOKEN="your-fleet-api-token"
export FLEET_TEAM_ID="1"
# etc.
```

### Using direnv (Advanced)

Install and configure [direnv](https://direnv.net/) for automatic environment loading:

```bash
brew install direnv
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
```

Then create `.envrc` in your project directory:
```bash
export $(grep -v '^#' .env | xargs)
```

Allow the directory:
```bash
direnv allow
```

## Configuration Reference

### Direct Mode (Upload to Fleet API)

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `FLEET_API_BASE` | Yes | Fleet server URL | `https://fleet.example.com` |
| `FLEET_API_TOKEN` | Yes | Fleet API token | `eyJhbGc...` |
| `FLEET_TEAM_ID` | Yes | Fleet team ID | `1` |

### GitOps Mode (Upload to S3 + PR)

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `AWS_ACCESS_KEY_ID` | Yes | AWS access key | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | Yes | AWS secret key | `wJalrXUtnFEMI/K7MDENG/...` |
| `AWS_DEFAULT_REGION` | No | AWS region | `us-east-1` (default) |
| `AWS_S3_BUCKET` | Yes | S3 bucket name | `fleet-packages` |
| `AWS_CLOUDFRONT_DOMAIN` | Yes | CloudFront domain | `d1234.cloudfront.net` |
| `FLEET_GITOPS_REPO_URL` | Yes | GitOps repo URL | `https://github.com/org/repo.git` |
| `FLEET_GITOPS_GITHUB_TOKEN` | Yes | GitHub PAT | `ghp_...` |
| `FLEET_GITOPS_SOFTWARE_DIR` | No | Software YAML dir | `lib/macos/software` (default) |
| `FLEET_GITOPS_TEAM_YAML_PATH` | Yes | Team YAML path | `teams/engineering.yml` |
| `S3_RETENTION_VERSIONS` | No | S3 version retention | `3` (default) |

## Security Best Practices

1. **Never commit secrets** - Always add `.env` to `.gitignore`
2. **Use minimal permissions** - AWS IAM users should have least-privilege access
3. **Rotate credentials** - Regularly rotate API tokens and access keys
4. **Use read-only tokens** - For GitHub, use fine-grained PATs with minimal scopes
5. **Protect plist files** - Ensure `com.github.autopkg.plist` has proper file permissions

## Troubleshooting

### Preference Not Found

If preferences aren't being read:
```bash
# Check if preference exists
defaults read com.github.autopkg AWS_ACCESS_KEY_ID

# If not found, set it
defaults write com.github.autopkg AWS_ACCESS_KEY_ID "your-key"
```

### Environment Variable Not Loaded

```bash
# Verify environment variable is set
echo $AWS_ACCESS_KEY_ID

# Re-source your environment file
export $(grep -v '^#' .env | xargs)
```

### Permission Denied on Preferences

```bash
# Fix plist permissions
chmod 600 ~/Library/Preferences/com.github.autopkg.plist
```

## Examples

### Running with AutoPkg Preferences

```bash
# Set preferences once
defaults write com.github.autopkg FLEET_API_BASE "https://fleet.example.com"
defaults write com.github.autopkg FLEET_API_TOKEN "abc123..."
defaults write com.github.autopkg FLEET_TEAM_ID -int 1

# Run recipe (preferences are automatically loaded)
autopkg run Google/GoogleChrome.fleet.recipe.yaml
```

### Running with Environment Variables

```bash
# Load environment variables
export FLEET_API_BASE="https://fleet.example.com"
export FLEET_API_TOKEN="abc123..."
export FLEET_TEAM_ID="1"

# Run recipe
autopkg run Google/GoogleChrome.fleet.recipe.yaml
```

### Temporary Override

```bash
# Override a preference for one run
FLEET_TEAM_ID=2 autopkg run Google/GoogleChrome.fleet.recipe.yaml
```
