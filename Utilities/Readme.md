Neovim Installation Guide with AstroVim Setup

This comprehensive guide covers installing the latest stable Neovim from binary, setting up essential tools, and configuring AstroVim on Ubuntu/Linux systems.
Install Latest Neovim Stable (v0.11.4)

Download and extract the latest stable Neovim binary :

bash
# Download the latest stable release
curl -LO https://github.com/neovim/neovim/releases/download/v0.11.4/nvim-linux-x86_64.tar.gz

# Extract to /opt directory
sudo tar xzf nvim-linux-x86_64.tar.gz -C /opt/

# Create symbolic link for system-wide access
sudo ln -sf /opt/nvim-linux-x86_64/bin/nvim /usr/local/bin/nvim

# Verify installation
nvim --version

Install Nerd Font (JetBrains Mono NL)

Install JetBrains Mono Nerd Font and reset font cache :

bash
# Download JetBrains Mono Nerd Font
curl -LO https://github.com/ryanoasis/nerd-fonts/releases/download/v3.0.2/JetBrainsMono.zip

# Create fonts directory and extract
mkdir -p ~/.local/share/fonts
unzip JetBrainsMono.zip -d ~/.local/share/fonts/JetBrainsMono/

# Reset font cache
fc-cache -fv

# Clean up download
rm JetBrainsMono.zip

Install Essential Tools
Core Dependencies

Install xclip, ripgrep, fzf, and Node.js for LSP support :

bash
# Update package manager
sudo apt update

# Install core tools
sudo apt install -y xclip ripgrep fzf nodejs npm

# Verify installations
xclip -version
rg --version
fzf --version
node --version

Optional System Monitoring Tools

Install bottom (btm) for system monitoring :

bash
# Get latest bottom version
BOTTOM_VERSION=$(curl -s "https://api.github.com/repos/ClementTsang/bottom/releases/latest" | grep -Po '"tag_name": "\K[0-9.]+')

# Download and install bottom
curl -Lo bottom.deb "https://github.com/ClementTsang/bottom/releases/latest/download/bottom_${BOTTOM_VERSION}_amd64.deb"
sudo apt install -y ./bottom.deb
rm -rf bottom.deb

# Test installation
btm --version

Install gdu for disk usage analysis :

bash
# Install gdu from snap (alternative to apt)
sudo snap install gdu-disk-usage-analyzer
# or use: sudo apt install gdu

Install lazygit for Git management :

bash
# Get latest lazygit version
LAZYGIT_VERSION=$(curl -s "https://api.github.com/repos/jesseduffield/lazygit/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')

# Download and install lazygit
curl -Lo lazygit.tar.gz "https://github.com/jesseduffield/lazygit/releases/download/v${LAZYGIT_VERSION}/lazygit_${LAZYGIT_VERSION}_Linux_x86_64.tar.gz"
sudo tar xf lazygit.tar.gz -C /usr/local/bin lazygit
rm -rf lazygit.tar.gz

# Verify installation
lazygit --version

Setup AstroVim Configuration

Replace the default Neovim configuration with your custom AstroVim setup:

bash
# Backup existing config (if any)
mv ~/.config/nvim ~/.config/nvim.backup 2>/dev/null || true

# copy your AstroVim configuration
# cp -r /path/to/your/nvim-config ~/.config/nvim

Initial Neovim Setup

Start Neovim and let it install packages automatically:

bash
# Launch Neovim for initial setup
nvim

# Neovim will automatically:
# 1. Install Lazy.nvim plugin manager
# 2. Download and install all plugins
# 3. Install Mason packages (LSP servers, formatters, linters)
# 
# This process may take a few minutes on first run
# Wait for all installations to complete

Customize ASCII Art

Modify the startup ASCII art with your name or logo:

bash
# Edit the user configuration file
nvim ~/.config/nvim/lua/plugins/user.lua

# Look for ASCII art section and customize with your preferred design
# Save and restart Neovim to see changes

Verification Steps

After setup, verify everything works correctly:

bash
# Check Neovim health
nvim +checkhealth

# Test LSP functionality by opening a code file
nvim test.py  # or any programming language file

# Verify plugin manager
nvim +Lazy

# Check Mason installations
nvim +Mason

Troubleshooting

If you encounter issues during setup :

    Font not displaying: Restart terminal application and verify font selection in terminal preferences

    LSP not working: Ensure Node.js is properly installed and accessible via node --version

    Clipboard issues: Verify xclip installation with xclip -version

    Plugin installation errors: Check internet connection and run :Lazy sync in Neovim

This setup provides a complete development environment with the latest stable Neovim, essential command-line tools, and a fully configured AstroVim setup optimized for productivity.
