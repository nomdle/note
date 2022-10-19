===========================================================
Javascript
===========================================================

.. toctree::
    :maxdepth: 1

    electronjs/sub1
    electronjs/sub2

Keywords
=======================================

electronjs
---------------------------------------
javascript, html, cssでDesktop Applicationを作成するためのフレームワーク
Chromiumとnodejsをバイナリで組み込んでいてjavascriptコードベースを維持しながらCrossplatformアプリケーションを作成可能とする。

react
---------------------------------------

jsx
---------------------------------------

component
---------------------------------------
functionを利用してHTMLで書かれたDOMを返却してタグのように使用できるようにする。
Reactの機能？



環境構築
=======================================

開発環境
---------------------------------------

Windows 10 Home

#. nodejsからWindowsのInstallerをダウンロードして設置
#. npmでyarnを設置
    .. code-block:: shell

        C:\project\electronjs_react2>npm install -g yarn

        added 1 package, and audited 2 packages in 692ms

        found 0 vulnerabilities
        npm notice
        npm notice New minor version of npm available! 8.18.0 -> 8.19.1
        npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.19.1
        npm notice Run npm install -g npm@8.19.1 to update!
        npm notice

        C:\project\electronjs_react2>

#. npmでelectronを設置
    .. code-block:: shell

        npm install --save-dev electron

#. projectディレクトリを作成して、初期化
    .. code-block:: shell

        npm init

----


主題：
簡単なUI要素
データの分離
UI更新

reference
---------------------------------------

| `electronjs <https://www.electronjs.org/ja/docs/latest/>`_
| `nodejs <https://nodejs.org/en/download/>`_