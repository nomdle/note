===========================================================
Electron + React
===========================================================

Reference
===========================================================
* https://medium.com/folkdevelopers/the-ultimate-guide-to-electron-with-react-8df8d73f4c97
* https://blog.codefactory.ai/electron/create-desktop-app-with-react-and-electron/1-project-setting/

環境構築から実行まで
===========================================================
#. Create Project
    .. code-block:: doscon

        c:\project>yarn create react-app --template typescript electronjs_react_study
        yarn create v1.22.19
        [1/4] Resolving packages...
        [2/4] Fetching packages...
        [3/4] Linking dependencies...
        [4/4] Building fresh packages...
        .
        .
        .
        Happy hacking!
        Done in 48.28s.

        c:\project>

#. Setup Electron
    .. code-block:: doscon

        C:\project\electronjs_react_study>yarn add -D electron electron-builder
        yarn add v1.22.19
        [1/4] Resolving packages...
        warning electron-builder > app-builder-lib > electron-osx-sign@0.6.0: Please use @electron/osx-sign moving forward. Be aware the API is slightly different
        [2/4] Fetching packages...
        [3/4] Linking dependencies...
        warning " > @testing-library/user-event@13.5.0" has unmet peer dependency "@testing-library/dom@>=7.21.4".
        warning "react-scripts > eslint-config-react-app > eslint-plugin-flowtype@8.0.3" has unmet peer dependency "@babel/plugin-syntax-flow@^7.14.5".
        warning "react-scripts > eslint-config-react-app > eslint-plugin-flowtype@8.0.3" has unmet peer dependency "@babel/plugin-transform-react-jsx@^7.14.9".
        [4/4] Building fresh packages...
        success Saved lockfile.
        success Saved 110 new dependencies.
        .
        .
        .
        └─ yauzl@2.10.0
        Done in 99.22s.

        C:\project\electronjs_react_study>

#. Execute
    #. プロジェクトrootのpackage.json修正
        .. code-block:: javascript
            :caption: package.json

            {
                "name": "electronjs_react_study",
                "version": "0.1.0",
                "private": true,
                "main": "public/Main.js",
                "homepage": "./",
                .
                .
                .
                "scripts": {
                    "start": "react-scripts start",
                    "build": "react-scripts build",
                    "test": "react-scripts test",
                    "eject": "react-scripts eject",
                    "react-start": "yarn start",
                    "electron-start": "set ELECTRON_START_URL=http://localhost:3000 && electron .",
                    "electron-pack": "yarn build && electron-builder build -c.extraMetadata.main=build/main.js"
                },
                .
                .
                .
            }

    #. publicにmain.js作成
        .. code-block:: javascript
            :caption: main.js

            const {app, BrowserWindow} = require('electron');
            const path = require('path');
            const url = require('url');
            function createWindow() {
                const win = new BrowserWindow({
                    width:1920,
                    height:1080
                });
                const startUrl = process.env.ELECTRON_START_URL || url.format({
                    pathname: path.join(__dirname, '/../build/index.html'),
                    protocol: 'file:',
                    slashes: true
                });
                win.loadURL(startUrl);
            }

            app.on('ready', createWindow);

    #. プロジェクトrootディレクトリに.envファイルを作成
        .. code-block:: none
            :caption: .env
        
            BROWSER=none

    #. `yarn react-start` でReactをバックグラウンドで起動
    #. `yarn electron-start` でElectronで起動
#. Packaging
    .. code-block:: doscon

        C:\project\electronjs_react_study>yarn electron-pack
        yarn run v1.22.19
        $ yarn build && electron-builder build -c.extraMetadata.main=build/main.js
        $ react-scripts build
        Creating an optimized production build...
        Compiled successfully.

        File sizes after gzip:

        46.61 kB  build\static\js\main.0869c737.js
        1.79 kB   build\static\js\787.5cd121c6.chunk.js
        541 B     build\static\css\main.073c9b0a.css

        The project was built assuming it is hosted at ./.
        You can control this with the homepage field in your package.json.

        The build folder is ready to be deployed.

        Find out more about deployment here:

        https://cra.link/deployment

        • electron-builder  version=23.3.3 os=10.0.19044
        • public/electron.js not found. Please see https://medium.com/@kitze/%EF%B8%8F-from-react-to-an-electron-app-ready-for-production-a0468ecb1da3
        • loaded parent configuration  preset=react-cra
        • description is missed in the package.json  appPackageFile=C:\project\electronjs_react_study\package.json
        • author is missed in the package.json  appPackageFile=C:\project\electronjs_react_study\package.json
        • writing effective config  file=dist\builder-effective-config.yaml
        • packaging       platform=win32 arch=x64 electron=20.1.1 appOutDir=dist\win-unpacked  • default Electron icon is used  reason=application icon is not set
        • building        target=nsis file=dist\electronjs_react_study Setup 0.1.0.exe archs=x64 oneClick=true perMachine=false
        • building block map  blockMapFile=dist\electronjs_react_study Setup 0.1.0.exe.blockmap
        Done in 40.64s.

        C:\project\electronjs_react_study>

    distの配下にexeファイルが生成

----

Note
===========================================================
.. code-block:: doscon

    D:\projects\electronjs_react>npx create-react-app .

    Creating a new React app in D:\projects\electronjs_react.

    Installing packages. This might take a couple of minutes.
    Installing react, react-dom, and react-scripts with cra-template...

    [#########.........] | idealTree: timing idealTree Completed in 4424ms

| sdカードでディレクトリを作成して実行するとここで止まってしまう。
| ->普通にSSDのディレクトリでは問題なし。