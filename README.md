Description:

This project enhances the portability and efficiency of managing AI-generated images from Civitai on Android devices. Leveraging Termux, a powerful terminal emulator for Android, and ExifTool, a robust metadata manipulation tool, users can download, organize, and search their Civitai generations directly on their mobile devices.

Key Features:

Civitai Sync Integration: Utilizes civitai-sync to download AI-generated images from Civitai, ensuring your creations are always up-to-date.

Metadata Search with ExifTool: Employs ExifTool to embed and search image metadata, allowing for efficient keyword-based searches directly on your device.

Bash Scripting for Automation: Includes custom Bash scripts to automate the download and organization process, providing a seamless user experience.

Termux Environment: Runs entirely within Termux, eliminating the need for a separate PC and offering true portability.


Background:

Civitai's web-based image explorer lacks advanced browsing capabilities, and existing Android photo viewers do not support EXIF metadata searches. This project addresses these limitations by creating a mobile solution that allows users to manage and search their AI-generated images on the go.

Setting Up CivitAI Sync and Find&Copy Script on Termux for Android

### **Complete Guide: Setting Up CivitAI Sync and Find&Copy Script on Termux for Android**

This guide walks you through setting up **CivitAI Sync** and a custom **Find&Copy Bash Script** on **Termux**, enabling you to efficiently **download, organize, and search your AI-generated images** using EXIF metadata directly from your Android device.

---

## 📲 **1. Install Termux and Enable Shared Storage**

1. **Download Termux:**  
   - Install Termux from [F-Droid](https://f-droid.org/en/packages/com.termux/).

2. **Update and Upgrade Packages:**  
   ```bash
   pkg update && pkg upgrade
   ```

3. **Enable Shared Storage Access:**  
   ```bash
   termux-setup-storage
   ```
   - Grant the necessary permissions when prompted.

4. **Verify Storage Access:**  
   ```bash
   ls ~/storage/shared
   ```
   You should see folders like `DCIM`, `Download`, etc.

---

## ⚙️ **2. Install Required Tools**

1. **Install Node.js, Git, and ExifTool:**  
   ```bash
   pkg install nodejs git exiftool
   ```

2. **Verify Installations:**  
   ```bash
   node -v
   git --version
   exiftool -ver
   ```

---

## 💻 **3. Install CivitAI Sync**

1. **Clone the CivitAI Sync Repository:**  
   ```bash
        cd ~/storage/shared
        mkdir -p civitai-sync
        cd civitai-sync
        git clone https://github.com/monkeypuzzl/civitai-sync.git
   ```

2. **Install Dependencies:**  
   ```bash
   npm install
   ```

3. **Run the Configuration Wizard:**  
   Instead of manually editing the configuration file, use the built-in wizard:  
   ```bash
   npm run cli
   ```
   - Follow the prompts to set up your **API Key** and preferences interactively.
   - Images should download to `~/civitai-sync/generations`.
     
4. **Test CivitAI Sync:**  
   - Run   `npm run cli` and download the latest generations to `~/civitai-sync/generations`.
  
---

## 🔍 **4. Set Up the Find&Copy Bash Script**

1. **Create the Script Directory:**  
   ```bash
   mkdir -p ~/scripts
   cd ~/scripts
   ```

2. **Create the Script File:**  
   ```bash
   nano fnc.sh
   ```

3. **Paste the Script:**  
   ```bash
   #!/bin/bash

   # Basic Configuration
   CARPETA_RAIZ="$HOME/storage/shared/xImages"
   CARPETA_ORIGEN="$HOME/storage/shared/civitai_generations"

   if [ -z "$1" ]; then
     echo "Usage: $0 '<search condition>'"
     echo "       $0 -purge"
     exit 1
   fi

   # Option: -purge
   if [ "$1" == "-purge" ]; then
     echo "Removing all folders in $CARPETA_RAIZ..."
     rm -rf "$CARPETA_RAIZ"/*
     echo "Folders removed."
     exit 0
   fi

   mkdir -p "$CARPETA_RAIZ"

   # Get search phrase and clean folder name
   PHRASE="$1"
   CARPETA_LIMPIA=$(echo "$PHRASE" | sed 's/&&/and/g; s/||/or/g; s/!/not/g' | tr ' ' '_')
   CARPETA_DESTINO="$CARPETA_RAIZ/$CARPETA_LIMPIA"
   mkdir -p "$CARPETA_DESTINO"

   # Generate condition explicitly handling logical operators
   CONDITION=""
   for TOKEN in $PHRASE; do
     case "$TOKEN" in
       "&&")
         CONDITION="$CONDITION &&"
         ;;
       "||")
         CONDITION="$CONDITION ||"
         ;;
       "!")
         CONDITION="$CONDITION !"
         ;;
       *)
         CONDITION="$CONDITION (\$UserComment =~ /$TOKEN/i)"
         ;;
     esac
   done

   echo "Generated condition: $CONDITION"

   # Run exiftool to copy matching files
   exiftool -r -if "$CONDITION" -p "\$Directory/\$Filename" "$CARPETA_ORIGEN" | while read FILE; do
     cp "$FILE" "$CARPETA_DESTINO/" || echo "Error copying $FILE"
   done

   echo "Images copied to $CARPETA_DESTINO."
   ```

4. **Make the Script Executable:**  
   ```bash
   chmod +x fnc.sh
   ```

---

## 🚀 **5. How to Use the Script**

1. **Search for Images with Keywords:**  
   ```bash
   ~/scripts/fnc.sh 'elf && Christmas || !Santa'
   ```
   - Matching images will be copied to `~/storage/shared/xImages/elf_and_Christmas_or_notSanta`.

2. **Purge All Results:**  
   ```bash
   ~/scripts/fnc.sh -purge
   ```

3. **Verify Copied Files:**  
   ```bash
   ls ~/storage/shared/xImages
   ```

---

## 🧹 **6. Create Aliases for Convenience**

To simplify usage, add these aliases to your `.bashrc`:  
```bash
echo "alias civitai-latest='node ~/civitai-sync/src/cli.mjs download-latest-generations'" >> ~/.bashrc
echo "alias findcivitai='~/scripts/fnc.sh'" >> ~/.bashrc
source ~/.bashrc
```

**Usage:**  
- `civitai-latest` → Download latest generations.  
- `findcivitai "elf && Christmas"` → Search and copy images.  
- `findcivitai -purge` → Clean all results.

---

## 📂 **7. Final Check**

1. **Download New Images:**  
   ```bash
   civitai-latest
   ```

2. **Search with Keywords:**  
   ```bash
   findcivitai "angel && wings"
   ```

3. **Purge Old Results (if needed):**  
   ```bash
   findcivitai -purge
   ```

---

## 🎯 **Done!**
You now have a **portable and efficient setup** for managing and searching CivitAI-generated images directly on your Android device.

Let me know if you encounter any issues! 🚀

🎯 Done!

You now have a portable and efficient setup for managing and searching CivitAI-generated images directly on your Android device.

Let me know if you encounter any issues! 🚀

Note:

While civitai-sync and Windows File Explorer offer robust solutions on their respective platforms, this project focuses on providing similar functionality with the added portability of an Android device.

Future Enhancements:

User-Friendly Interface: Develop a more intuitive interface, possibly with interactive menus, to make the tool accessible to users unfamiliar with command-line operations.

Performance Optimization: Evaluate the impact of recursive scans on devices with limited hardware and include tips to optimize performance.

Community Sharing: Publish the scripts and documentation on a platform like GitHub to facilitate collaboration, updates, and feedback.


By following these steps, users can effectively manage their AI-generated images from Civitai directly on their Android devices, overcoming the limitations of existing tools and achieving true portability.

