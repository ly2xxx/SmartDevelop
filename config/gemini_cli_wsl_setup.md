# Complete Guide: Setting Up Gemini CLI on WSL2 Ubuntu

Google's Gemini CLI brings powerful AI capabilities directly to your terminal, offering access to Gemini 2.5 Pro with generous free usage limits and advanced features like code analysis and workflow automation. This comprehensive guide covers everything needed for a successful setup on WSL2 Ubuntu, including troubleshooting common issues and optimizing performance.

## Prerequisites and System Requirements

**Essential Requirements:**
- **WSL2**: Must be WSL2 (not WSL1) - verify with `wsl --version` from Windows PowerShell
- **Ubuntu Version**: 20.04 LTS, 22.04 LTS, or 24.04 LTS recommended
- **Node.js**: Version 18.0 or higher (critical dependency)
- **Internet Connection**: Required for AI model access and authentication
- **Google Account**: For free tier access (60 requests/minute, 1,000/day)

**Optional but Recommended:**
- **VS Code with Remote-WSL extension**: For optimal development experience
- **Windows Terminal**: For better terminal experience
- **Git**: For project management and source installations

**Verify WSL2 Setup:**
```bash
# From Windows PowerShell
wsl --version                # Check WSL version
wsl --list --verbose        # List distributions

# From WSL2 Ubuntu
uname -a                    # Verify Linux environment
lsb_release -a             # Check Ubuntu version
```

## Installation Methods Overview

**Primary Method (Recommended):**
- NPM global installation via Node.js package manager
- Most reliable and officially supported approach

**Alternative Methods:**
- NPX direct execution (no permanent installation)
- GitHub source compilation (for developers)
- Go-based community alternatives
- Python-based alternatives

**Important Note:** No official Ubuntu APT packages exist - NPM remains the primary distribution method.

## Step-by-Step Installation Process

### Step 1: Install Node.js (Prerequisites)

**Method A: Using Node Version Manager (Recommended)**
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# Restart terminal or reload configuration
source ~/.bashrc

# Install and use latest Node.js LTS
nvm install --lts
nvm use --lts

# Verify installation
node -v    # Should show v18+ or higher
npm -v     # Should show npm version
```

**Method B: NodeSource Repository (Latest Versions)**
```bash
# Add official NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

# Install Node.js
sudo apt-get install -y nodejs

# Verify installation
node -v && npm -v
```

**Method C: Ubuntu APT (Basic)**
```bash
# Update package index
sudo apt update

# Install Node.js (may be older version)
sudo apt install nodejs npm

# Check version and upgrade if needed
node -v    # If below v18, use Method A or B
```

### Step 2: Configure NPM for WSL2

**Prevent Permission Issues:**
```bash
# Create global npm directory
mkdir ~/.npm-global

# Configure npm to use it
npm config set prefix '~/.npm-global'

# Add to PATH
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Step 3: Install Gemini CLI

**Primary Installation:**
```bash
# Global installation (recommended)
npm install -g @google/gemini-cli

# Verify installation
which gemini              # Should show installation path
npm list -g @google/gemini-cli  # Confirm global installation
```

**Alternative: Direct Execution (No Installation)**
```bash
# Run directly without installing
npx @google/gemini-cli

# Or from GitHub
npx https://github.com/google-gemini/gemini-cli
```

## Configuration Steps and API Key Setup

### Initial Authentication Setup

**Option 1: Google Account Authentication (Free Tier)**
```bash
# Launch Gemini CLI
gemini

# Follow interactive setup:
# 1. Select color theme
# 2. Choose "Login with Google"
# 3. Browser window opens automatically
# 4. Sign in and grant permissions
```

**WSL2 Browser Authentication Fix:**
If browser doesn't open automatically, configure WSL2 browser access:
```bash
# Method 1: Set browser environment variable
echo 'export BROWSER="/mnt/c/Windows/explorer.exe"' >> ~/.bashrc

# Method 2: Install WSL utilities
sudo apt install wslu
echo 'export BROWSER="wslview"' >> ~/.bashrc

# Reload configuration
source ~/.bashrc
```

**Option 2: API Key Authentication (Higher Limits)**

**Generate API Key:**
1. Visit [Google AI Studio](https://aistudio.google.com/apikey)
2. Sign in with Google account
3. Click "Create API key"
4. Copy the generated key

**Configure API Key:**
```bash
# Method 1: Environment variable (recommended)
echo 'export GEMINI_API_KEY="your_api_key_here"' >> ~/.bashrc
source ~/.bashrc

# Method 2: Global .env file
echo 'GEMINI_API_KEY=your_api_key_here' > ~/.env
chmod 600 ~/.env  # Secure the file

# Method 3: Project-specific .env
echo 'GEMINI_API_KEY=your_api_key_here' > .env
```

### Advanced Configuration

**Create Settings File:**
```bash
# Create configuration directory
mkdir -p ~/.gemini

# Create settings file
cat > ~/.gemini/settings.json << 'EOF'
{
  "model": "gemini-2.5-flash",
  "usageStatisticsEnabled": false,
  "fileFiltering": {
    "respectGitIgnore": true,
    "enableRecursiveFileSearch": true
  }
}
EOF

# Secure the configuration
chmod 600 ~/.gemini/settings.json
```

**Environment Variables Reference:**
```bash
# Add to ~/.bashrc for persistence
export GEMINI_API_KEY="your_api_key_here"    # Primary authentication
export GEMINI_MODEL="gemini-2.5-flash"       # Default model
export GEMINI_SANDBOX="true"                 # Enable sandboxing
export PATH="~/.npm-global/bin:$PATH"        # NPM global binaries
```

## Basic Usage Examples and Verification

### Verification Steps

**1. Installation Verification:**
```bash
# Check command availability
which gemini                    # Should show path
node -v                        # Verify Node.js version

# Test basic launch
gemini                         # Should open CLI without errors
```

**2. Authentication Verification:**
```bash
# In Gemini CLI
/auth                          # Check authentication method
/stats                         # Verify API connection and quotas
/help                          # Display available commands
```

**3. Functionality Test:**
```bash
# Basic AI interaction
> Hello, can you confirm you're working?
> What is 2+2?
> Explain what Node.js is in one sentence

# File operation test
> @package.json               # Reference a file
> !ls -la                     # Execute shell command
```

### Essential Commands

**In-CLI Commands:**
```bash
/help                   # Display help information
/auth                   # Switch authentication method
/tools                  # List available tools
/stats                  # Show usage statistics
/memory add <text>      # Add to AI memory
/clear                  # Clear chat context
/quit                   # Exit CLI
```

**File Operations:**
```bash
@filename               # Include file content
@directory/             # Include directory contents
!command                # Execute shell commands
```

**Built-in Tools:**
- `ReadFile` - Read single file content
- `ReadManyFiles` - Read multiple files
- `FindFiles` - Search files by pattern
- `SearchText` - Search within files
- `Edit` - Apply code changes
- `WriteFile` - Create new files
- `Shell` - Execute commands
- `WebFetch` - Fetch web content

### Common Usage Patterns

```bash
# Code analysis
> Describe the architecture of this system
> What are the security implications of @src/auth.js?

# Development tasks
> Write unit tests for @src/utils.js
> Help me debug this error: [paste error]
> Create documentation for the API endpoints

# Project management
> Generate a summary of changes in the last commit
> Create a README.md for this project
```

## Common Troubleshooting Issues and Solutions

### Node.js and NPM Issues

**Problem: "Command not found: gemini"**
```bash
# Check installation
npm list -g @google/gemini-cli

# Reinstall if needed
npm uninstall -g @google/gemini-cli
npm install -g @google/gemini-cli

# Check PATH
echo $PATH
export PATH="~/.npm-global/bin:$PATH"
```

**Problem: Permission denied during installation**
```bash
# Fix npm permissions
sudo chown -R $(whoami) ~/.npm
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# Alternative: Use NVM (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install --lts
```

### WSL2-Specific Issues

**Problem: Browser authentication fails**
```bash
# Solution 1: Configure browser environment
export BROWSER="/mnt/c/Windows/explorer.exe"
# Or for specific browsers:
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"

# Solution 2: Install WSL utilities
sudo apt install wslu
export BROWSER="wslview"

# Solution 3: Manual authentication
# Copy authentication URL from terminal
# Open in Windows browser manually
```

**Problem: Extremely slow performance**
```bash
# Issue: Working in Windows filesystem (/mnt/c/)
# Solution: Move projects to Linux filesystem

# Instead of: /mnt/c/Users/username/projects
# Use: /home/username/projects

# Performance comparison:
# Windows FS: 10+ minutes for npm install
# Linux FS: 1-2 minutes for npm install
```

**Problem: Line ending issues**
```bash
# Error: '/usr/bin/env: 'bash\r': No such file or directory'
# Solution: Fix line endings
sudo apt-get install dos2unix
find ~/.npm -name "*.sh" -exec dos2unix {} \;

# Git configuration
git config --global core.autocrlf false
git config --global core.eol lf
```

### Network and DNS Issues

**Problem: DNS resolution failures**
```bash
# Fix DNS configuration
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "nameserver 8.8.4.4" >> /etc/resolv.conf'

# Configure WSL to not generate resolv.conf
sudo bash -c 'echo "[network]" > /etc/wsl.conf'
sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf'

# Restart WSL2
wsl --shutdown    # From PowerShell
wsl              # Restart
```

**Problem: API connection failures**
```bash
# Test connectivity
curl -I https://generativelanguage.googleapis.com

# Check firewall/proxy settings
# Configure proxy if needed:
export HTTP_PROXY=http://proxy:port
export HTTPS_PROXY=http://proxy:port
```

### Authentication Issues

**Problem: API key not recognized**
```bash
# Verify environment variable
echo $GEMINI_API_KEY

# Re-source shell profile
source ~/.bashrc

# Check .env file location
cat ~/.env

# Alternative: Set in CLI
# Use /auth command to switch methods
```

## Best Practices for WSL2 Environment

### File System Organization

**Optimal Structure:**
```
/home/username/
├── projects/              # All development work here
│   ├── project1/
│   └── project2/
├── .gemini/              # CLI configuration
│   └── settings.json
├── .npm-global/          # NPM global packages
├── .env                  # Global environment variables
└── .bashrc               # Shell configuration
```

**Avoid These Locations:**
- `/mnt/c/` - Windows filesystem (extremely slow)
- `/mnt/d/` - Other Windows drives
- Any path starting with `/mnt/`

### Performance Optimization

**Memory Configuration:**
Create `C:\Users\<username>\.wslconfig`:
```ini
[wsl2]
memory=6GB              # Adjust based on system RAM
processors=4            # Limit CPU cores
swap=2GB               # Set swap size
localhostForwarding=true
```

**Node.js Optimization:**
```bash
# Add to ~/.bashrc
export NODE_OPTIONS="--max-old-space-size=4096"
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

### Security Best Practices

**API Key Security:**
```bash
# Set restrictive permissions
chmod 600 ~/.env
chmod 600 ~/.gemini/settings.json

# Never commit secrets
echo ".env" >> .gitignore
echo "*.key" >> .gitignore
```

**Environment Isolation:**
```bash
# Use project-specific .env files
cd /path/to/project
echo 'GEMINI_API_KEY=project_specific_key' > .env

# Consider separate API keys for different projects
```

### Development Workflow Integration

**VS Code Integration:**
```bash
# Install Remote-WSL extension
# Open projects from WSL: code .
# Configure VS Code settings:
"remote.WSL.fileWatcher.polling": true
"remote.WSL.fileWatcher.pollingInterval": 5000
```

**Git Configuration:**
```bash
# Optimize for cross-platform work
git config --global core.autocrlf false
git config --global core.eol lf
git config --global init.defaultBranch main
```

## Complete Setup Script

```bash
#!/bin/bash
# Gemini CLI Complete Setup Script for WSL2 Ubuntu

set -e  # Exit on any error

echo "🚀 Setting up Gemini CLI for WSL2 Ubuntu..."

# Update system
echo "📦 Updating system packages..."
sudo apt update && sudo apt upgrade -y

# Install dependencies
echo "🔧 Installing dependencies..."
sudo apt install -y curl git build-essential dos2unix wslu

# Install NVM and Node.js
echo "📗 Installing Node.js via NVM..."
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts
nvm use --lts

# Configure NPM
echo "⚙️  Configuring NPM..."
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'

# Install Gemini CLI
echo "🤖 Installing Gemini CLI..."
npm install -g @google/gemini-cli

# Create configuration directory
echo "📁 Setting up configuration..."
mkdir -p ~/.gemini

# Configure shell environment
echo "🐚 Configuring shell environment..."
cat >> ~/.bashrc << 'EOF'

# Gemini CLI Configuration
export PATH="~/.npm-global/bin:$PATH"
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export NODE_OPTIONS="--max-old-space-size=4096"
export BROWSER="wslview"

EOF

# Create default settings
cat > ~/.gemini/settings.json << 'EOF'
{
  "model": "gemini-2.5-flash",
  "usageStatisticsEnabled": false,
  "fileFiltering": {
    "respectGitIgnore": true,
    "enableRecursiveFileSearch": true
  }
}
EOF

chmod 600 ~/.gemini/settings.json

# Prompt for API key
echo "🔑 API Key Configuration:"
read -p "Enter your Gemini API key (press Enter to skip and use Google login): " api_key
if [ ! -z "$api_key" ]; then
    echo "GEMINI_API_KEY=$api_key" > ~/.env
    chmod 600 ~/.env
    echo "export GEMINI_API_KEY=\"$api_key\"" >> ~/.bashrc
    echo "✅ API key configured"
else
    echo "⏭️  Skipped API key setup - you can authenticate with Google account"
fi

# Source configuration
source ~/.bashrc

echo "✅ Setup complete!"
echo ""
echo "🎉 Gemini CLI is ready to use!"
echo "   • Run 'gemini' to start the CLI"
echo "   • Use /help for available commands"
echo "   • Check /stats for usage information"
echo ""
echo "📚 Next steps:"
echo "   1. Restart your terminal or run: source ~/.bashrc"
echo "   2. Run: gemini"
echo "   3. Follow authentication prompts"
echo "   4. Try: 'Hello, can you help me test the CLI?'"
```

This complete guide provides everything needed to successfully set up and use Gemini CLI on WSL2 Ubuntu, with specific attention to common issues and optimization techniques for the WSL2 environment. The setup script automates the entire process for quick deployment.