# Custom Font Setup for BookStack

This guide explains how to implement **Figtree** (Primary Body), **Google Sans** (Fallback), and **JetBrains Mono** (Code) into a self-hosted BookStack instance for offline/local use.

Included dark mode for code blocks!

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
<script>
    window.addEventListener('library-cm6::configure-theme', (event) => {
        const {EditorView} = event.detail;

        event.detail.registerViewTheme(() => ({
            "&": { 
                backgroundColor: "#2e3440", 
                color: "#eceff4",
                borderRadius: "10px !important",
                overflow: "hidden !important",
                boxShadow: "0 2px 8px rgba(0, 0, 0, 0.12)",
                /* Sets font size for the editor interface */
                fontSize: "14px"
            },
            
            ".cm-scroller": {
                fontFamily: "'JetBrains Mono', monospace !important",
                lineHeight: "1.6 !important",
                borderRadius: "10px !important",
                overflow: "hidden !important"
            },

            ".cm-gutters": { 
                backgroundColor: "#2e3440", 
                color: "#d8dee9",           
                borderRight: "1px solid #4c566a", 
                minWidth: "45px", /* Slightly wider for larger numbers */
                borderTopLeftRadius: "10px",
                borderBottomLeftRadius: "10px"
            },
            
            ".cm-activeLine": { backgroundColor: "#3b4252" },
            ".cm-cursor": { borderLeftColor: "#ffffff" }
        }));

        event.detail.registerHighlightStyle((t) => [
            {tag: t.keyword, color: "#81a1c1", fontWeight: "bold"},
            {tag: t.string, color: "#a3be8c"},
            {tag: t.comment, color: "#9ca3af", fontStyle: "italic"},
            {tag: t.number, color: "#b48ead"},
            {tag: [t.atom, t.bool, t.url], color: "#88c0d0"},
            {tag: [t.variableName, t.definition(t.variableName)], color: "#88c0d0"},
            {tag: [t.propertyName, t.definition(t.propertyName)], color: "#ebcb8b"},
            {tag: t.function(t.variableName), color: "#8fbcbb"},
            {tag: t.className, color: "#8fbcbb"},
            {tag: t.meta, color: "#d08770"},
            {tag: t.operator, color: "#81a1c1"}
        ]);
    });
</script>

<style>
    /* 1. CUSTOM FONTS */
    @font-face { font-family: 'Figtree'; src: url('http://novastack/fonts/figtree-v9-latin-regular.woff2') format('woff2'); }
    @font-face { font-family: 'JetBrains Mono'; src: url('http://novastack/fonts/JetBrainsMono-Regular.woff2') format('woff2'); }

    :root {
        --font-body: 'Figtree', sans-serif;
        --font-code: 'JetBrains Mono', monospace;
    }

    /* 2. Static Block Styling */
    body .page-content pre, 
    body .page-content .cm-editor {
        font-family: 'JetBrains Mono', monospace !important;
        font-size: 14px !important; /* Larger font size */
        background-color: #2e3440 !important;
        border-radius: 10px !important;
        overflow: hidden !important; 
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12) !important;
        border: 1px solid #434c5e !important;
        padding: 0 !important;
    }

    /* Padding fix for the text inside static blocks */
    body .page-content pre code {
        display: block;
        padding: 20px !important; /* Increased padding slightly for larger font */
        border-radius: 10px !important;
        font-family: 'JetBrains Mono', monospace !important;
        line-height: 1.6 !important;
    }

    /* 3. Inline Code Styling */
    body .page-content p > code, body .page-content li > code {
        background-color: #e5e9f0 !important;
        color: #bf616a !important;
        padding: 2px 5px !important;
        border-radius: 4px !important;
        border: 1px solid #d8dee9 !important;
        font-family: 'JetBrains Mono', monospace !important;
        font-size: 0.95em !important; /* Scaled slightly for inline text flow */
        font-weight: 600 !important;
    }

    /* Global Body Font */
    body {
        font-family: var(--font-body) !important;
    }
</style>
```

4. Click **Save Changes** at the bottom of the page.

## 5. Verification
* Refresh your BookStack page.
* If the fonts do not appear, press `F12` to open Developer Tools.
* Check the **Console** for "Content-Security-Policy" errors.
* Check the **Network** tab to ensure the `.woff2` files are returning a `200 OK` status.
