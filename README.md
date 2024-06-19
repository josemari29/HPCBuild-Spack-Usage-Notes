# HPCBuild-Spack-Usage-Notes

---

## Overview

This guide provides detailed instructions for installing and configuring Spack to manage software installations on a high-performance computing (HPC) system. The configuration sets up custom directories for new applications and module files, including using the generic concretizer to handle various microarchitectures.

## Prerequisites

- Ensure you have `git` installed on your system.
- Have necessary permissions to create directories and modify shell configuration files.

## Steps

### 1. Clone the Spack Repository

Clone the Spack repository from GitHub into the desired installation path:
```bash
git clone https://github.com/spack/spack.git /software/el9/spack
```

### 2. Add Spack to Your Shell Environment

Temporarily add Spack to your environment:
```bash
. /software/el9/spack/share/spack/setup-env.sh
```

To permanently add Spack to your shell environment, append the following line to your shell startup script:
```bash
echo ". /software/el9/spack/share/spack/setup-env.sh" >> ~/.bashrc
source ~/.bashrc
```
> **Note:** Replace `~/.bashrc` with the appropriate startup script for your shell (e.g., `~/.zshrc` for `zsh`).

### 3. Create Directories for Apps and Modules

Create the necessary directories for applications and module files:
```bash
mkdir -p /software/el9/spack/app
mkdir -p /software/el9/spack/lmod
```

### 4. Configure Spack

#### Edit the Configuration Files

Set the installation tree and module paths by editing Spack's configuration files.

**Configuring `config.yaml`**:
```bash
spack config edit config
```
Add the following lines:
```yaml
config:
  install_tree: /software/el9/spack/app
  concretizer: clingo
```

**Configuring `modules.yaml`**:
```bash
spack config edit modules
```
Add the following lines:
```yaml
modules:
  default:
    lmod:
      core_compilers:
      - gcc@11.3.1
    arch_folder: false
    lmod_path: /software/el9/spack/lmod
```

**Configuring `concretizer.yaml`**:
```bash
spack config edit concretizer
```
Add the following lines:
```yaml
concretizer:
  targets:
    granularity: generic
    host_compatible: true
  reuse: true
  unify: true
  duplicates:
    strategy: minimal
```

### 5. Create and Activate a Spack Environment

Create a new environment for managing applications:
```bash
spack env create new-apps-env
```

Activate the newly created environment:
```bash
spack env activate new-apps-env
```

### 6. Install Packages

Within the activated environment, install packages as needed. For example, to install `hdf5`:
```bash
spack install hdf5
```

List installed packages within the environment:
```bash
spack find
```

### 7. Set Up Lmod

#### Add Lmod Path to `MODULEPATH`

Ensure Lmod is set up to use Spack-generated modules by adding the Spack Lmod path to the `MODULEPATH`:
```sh
if [[ ":$MODULEPATH:" != *":/software/el9/spack/lmod:"* ]]; then
    export MODULEPATH=/software/el9/spack/lmod:$MODULEPATH
fi
```

### 8. Verify and Use Modules

#### Verify `MODULEPATH`

Check the `MODULEPATH` to ensure it includes the Spack Lmod path:
```sh
echo $MODULEPATH
```

#### List and Manage Modules

- **List Available Modules:**
  ```sh
  module avail
  ```
- **Load a Module:**
  ```sh
  module load <module_name>
  ```
- **Unload a Module:**
  ```sh
  module unload <module_name>
  ```
- **List Loaded Modules:**
  ```sh
  module list
  ```
- **Search for Modules:**
  ```sh
  module spider <module_name>
  module keyword <keyword>
  ```

### 9. Example `.bashrc` for Spack and Lmod Integration

```sh
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
if [ -d ~/.bashrc.d ]; then
    for rc in ~/.bashrc.d/*; do
        if [ -f "$rc" ]; then
            . "$rc"
        fi
    done
fi

unset rc

# Add Spack's lmod path to MODULEPATH if not already present
if [[ ":$MODULEPATH:" != *":/software/el9/spack/lmod:"* ]]; then
    export MODULEPATH=/software/el9/spack/lmod:$MODULEPATH
fi

# Add Spack to PATH
export PATH=/software/jdizon29/spack/bin:$PATH

# Source Spack environment setup
. /software/el9/spack/share/spack/setup-env.sh
```

### 10. Troubleshooting

#### Common Issues and Solutions

- **Module Not Found:**
  Ensure the `MODULEPATH` is correctly set and includes the Spack Lmod path:
  ```sh
  echo $MODULEPATH
  ```

- **Concretization Errors:**
  Verify the `config.yaml` and `concretizer.yaml` settings. Ensure the target and host compatibility settings are appropriate for your system.

#### Check Logs and Error Messages

- Review system logs and error messages for insights into issues.
- Use Spack's debug mode for more detailed output:
  ```sh
  spack -d install <package_name>
  ```

#### Re-create Configuration Files if Necessary

If errors persist, move or rename the problematic configuration files and re-create them:
```sh
mv ~/.spack/modules.yaml ~/.spack/modules.yaml.backup
spack config edit modules
```

#### Check File Permissions

Ensure the configuration files have correct permissions and are readable by Spack:
```sh
ls -l ~/.spack/modules.yaml
```

### 11. Common Spack Commands

#### General Commands

- **Initialize Spack Environment:**
  ```bash
  . /software/el9/spack/share/spack/setup-env.sh
  ```

- **Display Spack Version:**
  ```bash
  spack --version
  ```

- **Display Help Information:**
  ```bash
  spack help
  ```

#### Configuration Commands

- **Edit Configuration Files:**
  - `config.yaml`
    ```bash
    spack config edit config
    ```
  - `modules.yaml`
    ```bash
    spack config edit modules
    ```
  - `concretizer.yaml`
    ```bash
    spack config edit concretizer
    ```

- **List All Configuration Sections:**
  ```bash
  spack config list
  ```

- **Get Specific Configuration Section:**
  ```bash
  spack config get <section>
  ```
  *Example:*
  ```bash
  spack config get config
  ```

#### Environment Commands

- **Create a New Environment:**
  ```bash
  spack env create <env-name>
  ```
  *Example:*
  ```bash
  spack env create new-apps-env
  ```

- **Activate an Environment:**
  ```bash
  spack env activate <env-name>
  ```
  *Example:*
  ```bash
  spack env activate new-apps-env
  ```

- **Deactivate the Current Environment:**
  ```bash
  spack env deactivate
  ```

- **List All Environments:**
  ```bash
  spack env list
  ```

- **View Environment Details:**
  ```bash
  spack env status
  ```

#### Package Management Commands

- **Install a Package:**
  ```bash
  spack install <package-name>
  ```
  *Example:*
  ```bash
  spack install hdf5
  ```

- **List Installed Packages:**
  ```bash
  spack find
  ```

- **Find Specific Package Information:**
  ```bash
  spack info <package-name>
  ```
  *Example:*
  ```bash
  spack info hdf5
  ```

- **Search for Packages:**
  ```bash
  spack search <query>
  ```
  *Example:*
  ```bash
  spack search hdf5
  ```

- **Remove an Installed Package:**
  ```bash
  spack uninstall <package-name>
  ```
  *Example:*
  ```bash
  spack uninstall hdf5
  ```

#### Module Management Commands

- **Refresh Lmod Module Files:**
  ```bash
  spack module lmod refresh
  ```

- **List Available Modules:**
  ```bash
  module avail
  ```

#### Debugging and Troubleshooting Commands

- **Debug a Package Installation:**
  ```bash
  spack -d install <package-name>
  ```
  *Example:*
  ```bash
  spack -d install

 hdf5
  ```

- **Show Concretized Specs for a Package:**
  ```bash
  spack spec <package-name>
  ```
  *Example:*
  ```bash
  spack spec hdf5
  ```

- **Get Detailed Environment Status:**
  ```bash
  spack debug report
  ```

#### Additional Commands

- **Generate a Mirror:**
  ```bash
  spack mirror create -d <directory> <spec>
  ```
  *Example:*
  ```bash
  spack mirror create -d /path/to/mirror hdf5
  ```

- **List Compiler Information:**
  ```bash
  spack compilers
  ```

### 12. Consult Spack Documentation and Community

Refer to the [Spack documentation](https://spack.readthedocs.io/en/latest/) or seek help from the Spack community for further assistance.

---

By following these steps, you can set up and manage Spack effectively, ensuring a smooth experience for users of all experience levels. This guide provides detailed instructions for installation, configuration, environment setup, module management, troubleshooting, and verification, including using the generic concretizer to handle various microarchitectures.
