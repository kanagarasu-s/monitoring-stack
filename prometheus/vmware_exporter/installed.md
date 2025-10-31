# Update all packages
sudo dnf update -y

# Install required packages
sudo dnf install -y python3 python3-pip python3-venv git

# (Optional) Create a dedicated service user
sudo useradd --system --no-create-home --shell /sbin/nologin vmwareexp

