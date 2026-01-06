# Custom Font Setup for BookStack

This guide explains how to implement **Figtree** (Primary Body), **Google Sans** (Fallback), and **JetBrains Mono** (Code) into a self-hosted BookStack instance for offline/local use.

## 1. File Placement

To ensure the fonts are accessible to the web server, they must be placed inside the `public` directory of your BookStack installation.

1. Navigate to your BookStack root directory (e.g., `/var/www/bookstack`).
2. Create a `fonts` folder:
   
   ```
   mkdir -p public/fonts
   ```
   
4. Upload your `.woff2` files into `public/fonts/`. Your structure should look like this:

   ```
   public/fonts/
   ├── figtree-v9-latin-regular.woff2
   ├── google-sans-v67-latin-regular.woff2
   └── JetBrainsMono-Regular.woff2
   ```

## 2. Set Permissions

The web server needs explicit permission to read these files. Run these commands from your BookStack root directory:

```
# Set ownership to the web server user (standard is www-data)
sudo chown -R www-data:www-data public/fonts

# Set directory permissions to 755 (read/execute)
sudo find public/fonts -type d -exec chmod 755 {} \;

# Set file permissions to 644 (read)
sudo find public/fonts -type f -exec chmod 644 {} \;
```

## 3. Configure Content Security Policy (CSP)

BookStack's security headers will block local fonts unless they are whitelisted in the `.env` file.

1. Open your `.env` file:

   ```
   sudo nano .env
   ```
3. Add or update the following line (replace `http://novastack` with your actual domain):

   ```
   ALLOWED_EXTERNAL_RESOURCES="font-src http://novastack; style-src 'unsafe-inline' http://novastack"
   ```
   
5. Clear the BookStack configuration cache to apply the changes:

   ```
   sudo php artisan config:cache
   ```

## 4. Apply Custom HTML Head Content

1. Log in to BookStack as an **Admin**.
2. Go to **Settings** > **Customization**.
3. Locate the **Custom HTML Head Content** text area and paste the following block:

```
<style>
    /* 1. Define Figtree (Primary Body Font) */
    @font-face {
      font-family: 'Figtree';
      src: url('http://novastack/fonts/figtree-v9-latin-regular.woff2') format('woff2');
      font-weight: normal;
      font-style: normal;
    }

    /* 2. Define Google Sans (Secondary Fallback) */
    @font-face {
      font-family: 'Google Sans';
      src: url('http://novastack/fonts/google-sans-v67-latin-regular.woff2') format('woff2');
      font-weight: normal;
      font-style: normal;
    }

    /* 3. Define JetBrains Mono (For Code) */
    @font-face {
      font-family: 'JetBrains Mono';
      src: url('http://novastack/fonts/JetBrainsMono-Regular.woff2') format('woff2');
      font-weight: normal;
      font-style: normal;
    }

    /* 4. Apply to BookStack Variables */
    :root {
        --font-body: 'Figtree', 'Google Sans', sans-serif;
        --font-code: 'JetBrains Mono', monospace;
    }

    /* Force the editor and code blocks to use JetBrains Mono */
    code, pre, .CodeMirror, .cm-editor {
        font-family: 'JetBrains Mono', monospace !important;
    }
</style>
```

4. Click **Save Changes** at the bottom of the page.

## 5. Verification
* Refresh your BookStack page.
* If the fonts do not appear, press `F12` to open Developer Tools.
* Check the **Console** for "Content-Security-Policy" errors.
* Check the **Network** tab to ensure the `.woff2` files are returning a `200 OK` status.
