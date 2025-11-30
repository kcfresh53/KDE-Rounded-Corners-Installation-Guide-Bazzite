# KDE-Rounded-Corners-Installation-Guide-Bazzite
Installation guide for layering the rounded corner kwin effect on Bazzite or other immutable systems.

---

# Installing KDE-Rounded-Corners (Layered RPM)

This guide details the process of persistently installing the **KDE-Rounded-Corners** KWin effect on Bazzite (or any Fedora/Ublue OSTree-based system) by building a custom RPM package and layering it using `rpm-ostree`.

## Prerequisites

- A running Bazzite system.
- **Distrobox** installed and running.
- A **Fedora** Distrobox container created (e.g., named `fedora-build`).

## Step 1: Prepare the Distrobox Build Environment

We must use a standard Linux distribution (like Fedora) inside Distrobox to install the necessary build tools and compile the KWin effect.

1. **Enter the Distrobox:**
  
  ```bash
  distrobox enter <name-of-box>
  ```
  
2. **Install Build Dependencies:**
  Install all required packages, including the tools for building RPMs (`rpmdevtools`, `rpm-build`) and the KDE/Qt development libraries required by the spec file.
  
  ```bash
  sudo dnf install -y \
    rpmdevtools rpm-build git cmake gcc-c++ extra-cmake-modules kf6-rpm-macros \
    qt6-qtbase-devel qt6-qtbase-private-devel kf6-kconfigwidgets-devel \
    kf6-kcmutils-devel kf6-ki18n-devel kf6-kwindowsystem-devel libepoxy-devel \
    libxcb-devel libdrm-devel wayland-devel kwin-devel
  ```
  

## Step 2: Build and Package the RPM

This process uses the project's official `.spec` file to correctly structure the package.

1. **Clone the Repository:**
  
  ```bash
  git clone https://github.com/matinlotfali/KDE-Rounded-Corners
  ```
  
2. **Move Out and Rename the Directory:**
  The spec file expects the source directory to be named `KDE-Rounded-Corners-master`. We must rename the directory outside of it.
  
  ```bash
  cd ..
  mv KDE-Rounded-Corners KDE-Rounded-Corners-master
  ```
  
3. **Create the Source Tarball:**
  The tarball must be named `master.tar.gz` for the spec file to find it.
  
  ```bash
  tar --create --gzip --file master.tar.gz KDE-Rounded-Corners-master
  ```
  
4. **Set up RPM Build Tree:**
  This command creates the necessary `rpmbuild/{SOURCES, SPECS, RPMS, ...}` directories in your home folder.
  
  ```bash
  rpmdev-setuptree
  ```
  
5. **Move Source and Spec Files:**
  Move the prepared files into the appropriate directories within the RPM build tree.
  
  ```bash
  mv master.tar.gz ~/rpmbuild/SOURCES/
  mv KDE-Rounded-Corners-master/.copr/kwin-effect-roundcorners.spec ~/rpmbuild/SPECS/
  ```
  
6. **Build the RPM Package:**
  This compiles the source code and packages the resulting files into the final binary RPM (`.rpm`).
  
  ```bash
  rpmbuild -bb ~/rpmbuild/SPECS/kwin-effect-roundcorners.spec
  ```
  
7. **Locate and Copy the RPM:**
  Find the generated RPM file (e.g., `kwin-effect-roundcorners-0.7.2-1.fc39.x86_64.rpm`) and copy it to the host system's home directory.
  
  ```bash
  RPM_FILE=$(find ~/rpmbuild/RPMS/ -name "kwin-effect-roundcorners-*.rpm" -print -quit)
  cp $RPM_FILE /mnt/usr/home/user/
  ```
  
  *Note: Replace `/mnt/usr/home/user/` with your actual host user path if necessary.*
  
8. **Exit the Distrobox:**
  
  ```bash
  exit
  ```
  

## Step 3: Layer the Package on Bazzite

You are now back on your Bazzite host system.

1. **Layer the Local RPM File:**
  Use `rpm-ostree install` to integrate the package into the immutable OS image. The `--allow-inactive` flag ensures the installation proceeds even if parts of the effect conflict with existing files (e.g., if you had previously tried to install it).
  
  ```bash
  sudo rpm-ostree install --allow-inactive ~/$(basename $RPM_FILE)
  ```
  
2. **Reboot the System:**
  The changes are applied **offline** and require a reboot to take effect.
  
  ```bash
  systemctl reboot
  ```
  

## Step 4: Verification and Configuration

1. **Verify the Layer:**
  After reboot, confirm the package is layered:
  
  ```bash
  rpm-ostree status
  ```
  
  Look for **`kwin-effect-roundcorners`** listed under **Layered Packages**.
  
2. **Enable the Effect:**
  
  - Open **System Settings** in KDE Plasma.
  - Navigate to **Workspace** $\to$ **Desktop Effects**.
  - Find and enable the **Rounded Corners** effect.
