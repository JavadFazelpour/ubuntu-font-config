# ubuntu-font-config

fixing Arabic and Farsi typography on Ubuntu 24.04. This solution ensures that **Vazirmatn** is the primary font for both languages, with **IBM Plex** and **Noto Sans** acting as high-quality fallbacks for Arabic.

---

# 📝 Unified Font Solution: Farsi & Arabic on Ubuntu

## The Problem

By default, Ubuntu and browsers (like Chrome/Firefox) often default to "Arial" or generic system fonts for Arabic scripts. Arial's Arabic glyphs are widely considered ugly and hard to read. Furthermore, many websites (like X/Twitter) hard-code "Arial" in their CSS, forcing the browser to ignore your standard preferences.

## The Solution

We use **Fontconfig** (the Linux system for font configuration) to:

1. **Intercept** requests for "Arial" or "Helvetica" and force the system to provide **Vazirmatn** instead.
2. **Define a hierarchy** for Arabic text: Vazirmatn (1st) → IBM Plex (2nd) → Noto Sans (3rd).
3. **Strictly enforce** Vazirmatn for Farsi text.

---

### Step 1: Install the Necessary Fonts

Ensure all three font families are installed on your system.

```bash
sudo apt update
sudo apt install fonts-ibm-plex fonts-noto-core

```

*(Note: If you haven't installed Vazirmatn yet, download the `.deb` from the [Vazirmatn GitHub](https://github.com/rastikerdar/vazirmatn) or verify it is installed).*

### Step 2: Create the Configuration Directory

Create the local configuration folder if it doesn't already exist.

```bash
mkdir -p ~/.config/fontconfig/conf.d

```

### Step 3: Create the Master Config File

We will create a single file named `99-persian-arabic-fix.conf`. The number `99` ensures this loads *last*, overriding other system defaults.

1. Open the file:
```bash
nano ~/.config/fontconfig/conf.d/99-persian-arabic-fix.conf

```


2. Paste the following XML code exactly:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>

  <match target="pattern">
    <test name="family" qual="any">
      <string>Arial</string>
      <string>Tahoma</string>
      <string>Helvetica</string>
      <string>Times New Roman</string>
      <string>Times</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Vazirmatn</string>
      <string>IBM Plex Sans Arabic</string>
      <string>IBM Plex Sans Hebrew</string>
      <string>Noto Sans Arabic</string>
      <string>Noto Sans Hebrew</string>
    </edit>
  </match>

  <match target="pattern">
    <test name="lang" compare="contains"><string>fa</string></test>
    <test name="family"><string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Vazirmatn</string>
    </edit>
  </match>

  <match target="pattern">
    <test name="lang" compare="contains"><string>ar</string></test>
    <test name="family"><string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Vazirmatn</string>
      <string>IBM Plex Sans Arabic</string>
      <string>Noto Sans Arabic</string>
    </edit>
  </match>

  <match target="pattern">
    <test name="lang" compare="contains"><string>he</string></test>
    <test name="family"><string>sans-serif</string></test>
    <edit name="family" mode="prepend" binding="strong">
      <string>IBM Plex Sans Hebrew</string>
      <string>Noto Sans Hebrew</string>
    </edit>
  </match>

</fontconfig>

```

3. Save and exit: Press `Ctrl+O`, `Enter`, then `Ctrl+X`.

### Step 4: Fix Firefox (Ubuntu 24.04 Snap Issue)

Because Firefox on Ubuntu is a "Snap" package, it cannot see the file you just created in your home folder. You must copy it to the Snap's internal configuration path.

```bash
# Create the directory inside the Snap container
mkdir -p ~/snap/firefox/current/.config/fontconfig/conf.d

# Copy your new config file into it
cp ~/.config/fontconfig/conf.d/99-persian-arabic-fix.conf ~/snap/firefox/current/.config/fontconfig/conf.d/

```

### Step 5: Apply Changes and Verify

Refresh the system font cache.

```bash
fc-cache -f -v

```

Now, verify the system is respecting your priorities.

**Check Arabic Priority:**

```bash
fc-match -s "sans-serif:lang=ar" | head -n 3

```

*Expected Output:*

1. Vazirmatn
2. IBM Plex Sans Arabic
3. Noto Sans Arabic

**Check Farsi Priority:**

```bash
fc-match -s "sans-serif:lang=fa" | head -n 1

```

*Expected Output:*

1. Vazirmatn

---

### Final Note

After completing these steps, **restart your browser**. Twitter (X) and other sites will now use Vazirmatn for both Arabic and Farsi. If Vazirmatn ever lacks a specific Arabic character, the system will seamlessly switch to IBM Plex Sans Arabic for that specific glyph.

Would you like me to show you how to script Step 4 so it runs automatically whenever you restart your computer (in case Firefox updates reset it)?
