<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Embedder - Your Web Library</title>
    <style>
        :root {
            --bg-primary: #1e1e2e;
            --bg-sidebar: #252538;
            --accent: #89b4fa;
            --text: #cdd6f4;
            --text-muted: #7f849c;
            --border: #313244;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            display: flex;
            height: 100vh;
            background-color: var(--bg-primary);
            color: var(--text);
            overflow: hidden;
        }

        /* Sidebar Styling */
        #sidebar {
            width: 320px;
            background-color: var(--bg-sidebar);
            border-right: 1px solid var(--border);
            display: flex;
            flex-direction: column;
            padding: 20px;
            flex-shrink: 0;
        }

        h1 {
            font-size: 24px;
            margin-bottom: 20px;
            color: var(--accent);
            display: flex;
            align-items: center;
            gap: 10px;
        }

        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 25px;
        }

        input {
            background: var(--bg-primary);
            border: 1px solid var(--border);
            padding: 10px;
            border-radius: 6px;
            color: var(--text);
            font-size: 14px;
        }

        input:focus {
            outline: 1px solid var(--accent);
        }

        button {
            background: var(--accent);
            color: var(--bg-primary);
            border: none;
            padding: 10px;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            transition: opacity 0.2s;
        }

        button:hover {
            opacity: 0.9;
        }

        .library-title {
            font-size: 14px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: var(--text-muted);
            margin-bottom: 10px;
        }

        #library-list {
            list-style: none;
            overflow-y: auto;
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .library-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: var(--bg-primary);
            padding: 10px;
            border-radius: 6px;
            border: 1px solid var(--border);
            cursor: pointer;
            transition: border-color 0.2s;
        }

        .library-item:hover {
            border-color: var(--accent);
        }

        .library-item span {
            font-size: 14px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            max-width: 200px;
        }

        .delete-btn {
            background: transparent;
            color: #f38ba8;
            border: none;
            cursor: pointer;
            font-size: 12px;
            padding: 2px 6px;
        }

        /* Main Viewport Styling */
        #main-view {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            background: #11111b;
            position: relative;
        }

        #viewer-placeholder {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: var(--text-muted);
            text-align: center;
        }

        iframe {
            width: 100%;
            height: 100%;
            border: none;
            background: white;
            display: none;
        }
    </style>
</head>
<body>

    <!-- Sidebar Layout -->
    <div id="#sidebar" style="width: 320px; background-color: var(--bg-sidebar); border-right: 1px solid var(--border); display: flex; flex-direction: column; padding: 20px; flex-shrink: 0;">
        <h1>🔗 Embedder</h1>
        
        <!-- Input Form -->
        <form id="embed-form">
            <input type="text" id="site-name" placeholder="Website Name (e.g., Wikipedia)" required>
            <input type="url" id="site-url" placeholder="URL (https://...)" required>
            <button type="submit">Embed & Save</button>
        </form>

        <!-- Library Section -->
        <div class="library-title">The Library</div>
        <ul id="library-list"></ul>
    </div>

    <!-- Live Preview Display -->
    <div id="main-view">
        <div id="viewer-placeholder">
            <h2>No Website Loaded</h2>
            <p>Add a URL or select an item from your library to embed it here.</p>
        </div>
        <iframe id="web-viewer" sandbox="allow-scripts allow-same-origin allow-forms"></iframe>
    </div>

    <script>
        const form = document.getElementById('embed-form');
        const nameInput = document.getElementById('site-name');
        const urlInput = document.getElementById('site-url');
        const libraryList = document.getElementById('library-list');
        const iframe = document.getElementById('web-viewer');
        const placeholder = document.getElementById('viewer-placeholder');

        // Load library items from local storage
        let library = JSON.parse(localStorage.getItem('embedder_library')) || [];

        // App Initialization
        function init() {
            renderLibrary();
            form.addEventListener('submit', handleFormSubmit);
        }

        // Render library UI list
        function renderLibrary() {
            libraryList.innerHTML = '';
            library.forEach((item, index) => {
                const li = document.createElement('li');
                li.className = 'library-item';
                li.innerHTML = `
                    <span onclick="loadFrame('${item.url}')">${item.name}</span>
                    <button class="delete-btn" onclick="deleteItem(event, ${index})">Delete</button>
                `;
                libraryList.appendChild(li);
            });
        }

        // Handle URL submissions
        function handleFormSubmit(e) {
            e.preventDefault();
            let url = urlInput.value.trim();
            
            // Format URL if protocol is missing
            if (!/^https?:\/\//i.test(url)) {
                url = 'https://' + url;
            }

            const newItem = {
                name: nameInput.value.trim(),
                url: url
            };

            library.push(newItem);
            localStorage.setItem('embedder_library', JSON.stringify(library));
            
            renderLibrary();
            loadFrame(url);

            // Reset inputs
            nameInput.value = '';
            urlInput.value = '';
        }

        // Load selected URL inside the Iframe
        function loadFrame(url) {
            placeholder.style.display = 'none';
            iframe.style.display = 'block';
            iframe.src = url;
        }

        // Delete item from library
        window.deleteItem = function(e, index) {
            e.stopPropagation(); // Stop click from triggering item load
            library.splice(index, 1);
            localStorage.setItem('embedder_library', JSON.stringify(library));
            renderLibrary();
            
            // Clear view if no items left
            if (library.length === 0) {
                iframe.src = '';
                iframe.style.display = 'none';
                placeholder.style.display = 'block';
            }
        };

        // Initialize App
        init();
    </script>
</body>
</html>

