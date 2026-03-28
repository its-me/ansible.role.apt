# apt
Ansible role for installing packages with APT.
## Examples
### Inventory
#### autoinstall recommended and suggested packages:
```yaml
apt_install_recommends: False
apt_install_suggests: False
```
#### latest supported version of debian in repositories (not moved to archive):
```yaml
apt_latest_supported_debian: 10
```
#### add debian backports repository:
```yaml
apt_all:
  backports:
    repo: deb http://deb.debian.org/debian/ {{ ansible_distribution_release }}-backports main contrib non-free-firmware
```
#### add external repository:
```yaml
apt_all:
  repo_name:
    repo: <repository in debian format>
    key: <repository key url>
```
#### change external repository for environment or host:
```yaml
apt_env:
  repo_name:
    repo: <repository in debian format>
    key: <repository key url>
```
### dependencies:
#### install packages from debian repository:
```yaml
dependencies:
  - role: apt
    apt_packages: []
```
#### install packages from debian backports:
```yaml
dependencies:
  - role: apt
    apt_repo: backports
    apt_packages: []
```
#### install packages from external repository:
```yaml
dependencies:
  - role: apt
    apt_repo: repo_name
    apt_packages: []
```
