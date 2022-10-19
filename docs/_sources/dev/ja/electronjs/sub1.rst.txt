===========================================================
First sample from official site
===========================================================

https://www.electronjs.org/docs/latest/tutorial/quick-start

#. Project生成
    .. code-block:: shell

        npm init

#. package.json
    .. code-block:: json

        {
            "name": "electronjs_example1",
            "version": "1.0.0",
            "description": "first pratice",
            "main": "main.js",
            "scripts": {
                "start": "electron ."
            },
            "author": "soyoyoo",
            "license": "ISC",
            "devDependencies": {
                "electron": "^19.0.1"
            }
        }

#. main.js
    .. code-block:: javascript

        const { app, BrowserWindow } = require('electron')
        const createWindow = () => {
            const win = new BrowserWindow({
            width: 800,
            height: 600
            })
        
            win.loadFile('index.html')
        }
        // npm startでKickされるEvent
        app.whenReady().then(() => {
            createWindow()
        })

#. index.html
    .. code-block:: html

        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
            <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
            <title>Hello World!</title>
        </head>
        <body>
            <h1>Hello World!</h1>
            We are using Node.js <span id="node-version"></span>,
            Chromium <span id="chrome-version"></span>,
            and Electron <span id="electron-version"></span>.
        </body>
        </html>

#. 実行
    .. code-block:: shell

        npm start
