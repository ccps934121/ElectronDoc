# <font color="#4078c0">Quick Start</font>

###### tags: `Electron`

原文：[Electron - Quick Start](https://www.electronjs.org/docs/tutorial/quick-start#access-nodejs-from-the-renderer-with-a-preload-script)

Hackmd:[連結](https://hackmd.io/@RdUg6kDRTcKHmb2WGCucdA/SyR45_Gqd)

這個章節將會按照步驟用很直白的方式在Electron內去開發一個Hello World App。

在章節的最後，若有按照步驟完成的話，將可以執行一個應用程式版的瀏覽器，上面顯示Chromium、Node.js及Electron版本。

---
### <font color="#4078c0">首先 安裝Node.js</font>
為了要可以執行Electron，我們需要先去安裝Node.js，這邊是建議安裝最新的LTS版本。

>請用平台預先提供的安裝程序去安裝Node.js，不然在不同的開發工具上可能會有相容性的問題。

可以檢查看看Node.js是否有安裝成功，請在終端機輸入以下語法。

```
node -v
npm -v
```

![](https://i.imgur.com/JWPgRtJ.png)

這時候應該就可以看到node及npm的版本了！
##### <font color="#4078c0">**Note：**</font>Electron會將node.js嵌進它二進位編制中，所以它實際執行的版本有可能會與我們電腦所安裝的版本不一樣。
---
### <font color="#4078c0">產生一個應用程式</font>
#### <font color="#4078c0">建立一個專案</font>
Electron專案與node.js的專案結構一樣，所以要先新增一個專案並初始化npm的套件。
```
mkdir my-electron-app 
cd my-electron-app
npm init
```
執行npm init時，會跳出選項填寫，完成後可以看到資料夾底下多了package.json檔案，這邊有一些規則是必須遵從的：

- entry point 要改為 main.js. (原本是預設index.js)
- author和description可以是任意填寫，但是打包的時候會用到

那完成之後，package.json檔應該會是這個樣子
```json=
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "Hello World!",
  "main": "main.js",
  "author": "Jane Doe",
  "license": "MIT"
}
```
那接下來要在專案底下安裝Electron的套件。
```
npm install --save-dev electron
```

再來如果要能執行Electron，請在package.json的script物件內加入start這個指令，如下所示：
```json=
{
  "scripts": {
    "start": "electron ."
  }
}
```
這時候的package.json應該會類似這個樣子
![](https://i.imgur.com/tdAkG3K.png)

完成之後，就可以輸入start指令嘗試打開Electron
```
npm start
```
> 這個指令會通知Electron要在這個專案啟動，不過在目前這個步驟中，應該是會得到錯誤訊息且app還無法運行。

#### <font color="#4078c0">執行主程序</font>

Electrin在執行時，主要是要透過main script。這腳本主要是控制在node.js的環境中執行的過程、專案中的生命週期及操作的介面、渲染的過程等。

在執行的過程中，Electron會去package.json的main指令去找所對應到的檔案，當初在init所設定的main.js

**<font color="＃AE0000">為了要讓main指令可以運作，必須要在專案的根目錄底下新增一個檔案命名為main.js。</font>**

![](https://i.imgur.com/9a8Krhe.png)


> 這時候再去執行一次npm start的指令，應該就不會有錯誤訊息出現，不過也不會有任何反應，因為還沒在main.js內加入任何程式。

#### <font color="#4078c0">新增一個網頁</font>
在應用程式產生一個視窗之前，我們需要提供內容讓它能夠載入。在Electron中，視窗呈現的網頁內容可以是專案內的Html檔案或是一個遠端的網址。

在這章節中會使用前者，在專案底下新增一個index.html檔案，內容就如下：
```htmlembedded=
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.
  </body>
</html>
```
> Note:可以注意到在這個HTML文件中，版本號的位置是沒有值的，這部分接下來的步驟會用javascript去產生。

#### <font color="#4078c0">在瀏覽器視窗開啟網頁</font>
現在已經有一個網頁了，要將它載進應用程式的視窗內，所以需要用到兩個Electron的套件：
- app: 控制應用程式內事件的生命週期。
- BrowserWindow: 產生並運作一個應用程式的視窗。


```javascript=
// main.js
const { app, BrowserWindow } = require('electron')
```

接下來為了要載入index.html，所以需要新增一個createWindow的function，然後新增一個BrowserWindow的實例。

```javascript=
// main.js
function createWindow () {
  const win = new BrowserWindow({
    width: 800,
    height: 600
  })

  win.loadFile('index.html')
}
```

接下來，呼叫createWindow()來開啟視窗。
在Electron中，瀏覽器視窗只會在app套件中的ready事件被觸發之後才會產生。所以可以使用app.whenReady()事件來等待事件的觸發。在whenReady()的callback function中 呼叫createWindow()就可以解決app還沒準備好的問題。

```javascript=
// main.js
app.whenReady().then(() => {
  createWindow()
})
```

> 這時候再去執行一次npm start，Electron的應用程式應該就可以成功run起來，並顯示index.html的內容。

目前main.js就會是這個樣子
![](https://i.imgur.com/tLlYHOj.png)

及Electron run起來的樣子
![](https://i.imgur.com/F45wkj1.png)


#### <font color="#4078c0">管理視窗的生命週期</font>

雖然現在已經可以成功執行瀏覽器視窗，但因在每個平台的操作行為不同，所以可以透過額外的樣板程式碼(boilerplate code)來讓你的視窗更加地適應在每個平台上。Electron將這個責任賦予給開發者去屢行這些設定。

通常可以使用process內platform這個全域的屬性來操作系統內的設定。

> 什麼是boilerplate code？
> 很類似一種樣版，在使用不同程式語言時，常常會有一些基本設定，或是常常需要重複寫的code，每次開新專案時就要輸入的話會相當耗時，所以boilerplate就把東西包好好，直接執行就可以產出你所要開發的樣本了
> Ex.Vue-Cli 就是自動產生Vue開發的樣本

##### <font color="#4078c0">當所有的視窗都被關閉結束這個app(Windows & Linux)</font>

在Window及Linux中，當關閉所有視窗時，通常該應用程式也會被完整的結束運行。
為了要實現這個做法，可以去監聽app模組中的window-all-closed事件，然後當使用者的作業系統不是MacOS(darwin)的話就去呼叫app.quit()這個function。

```javascript=
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})
```

##### <font color="#4078c0">當視窗都被關閉時開啟一個新視窗(MacOS)</font>

macOS與Linux及Windows作業系統關閉視窗後就會結束該應用程式的規則不一樣，當沒有視窗時通常它還是會持續運行，而且會去觸發該應用程式去開啟一個新的視窗。

為了要去實行這個特點，可以去監聽app模組中的activate事件，當沒有瀏覽器視窗被開啟時，再次呼叫createWindow()這個function。

由於上面有提到說視窗不能在ready事件之前被創造，所以應該要在當初始化app時去監聽activate事件，將該事件放在已經存在的whenReady()的callback function中。

```javascript=
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})
```

> 這時候，我們這視窗的控制就相當的齊全了！

#### <font color="#4078c0">透過preload腳本從renderer來訪問Node.js</font>

現在，最後一步要做的事情是要把Electron及相依的工具版本號印出來並顯示在網頁上。

為了要得知版本的資訊，我們可以使用Node的全域物件process來取得，但是！我們無法直接在主程序中去控制DOM元件，因為它沒有權限去渲染document的內文。它們是完全不同的程序。

> 如果需要更深入的研究關於Electron執行的程序，可以去看看[Process Model](https://www.electronjs.org/docs/tutorial/process-model)的文件

preload會在渲染的程序之前被載入，並將進入window、document及Node.js環境之間，這邊preload很像是這兩者之間的橋樑，將不同程序的東係搭載在一起。

所以現在需要在專案底下新增一個preload.js的檔案，內容如下：

```javascript=
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

上面的程式碼是透過process.version物件去取得Node.js的版本，並執行replaceText()將取得的版本號依序加到HTML中。

為了要將preload的程式渲染到現有的BrowserWindow，可以將preload.js的路徑傳入webPreferences.preload。

```javascript=
// include the Node.js 'path' module at the top of your file
const path = require('path')

// modify your existing createWindow() function
function createWindow () {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  win.loadFile('index.html')
}
// ...
```

這邊有使用到兩個Node.js的概念

- __dirname: 正在執行的專案的路徑(該範例來看就是專案的根目錄)
- path.join: 連接多個路徑，結合之後產生一個新的路徑字串，並可以在該平台的各地方使用。

如果使用相對路徑的話，就可以在開端及打包的時候都可以使用。


#### <font color="#4078c0">額外：在網頁中增加更多的功能</font>
在這個段落，可以學到如何在應用程式中增加更多的功能
因為渲染器會在原生的網頁中運行，所以在index.html中body標籤結尾的地方加入script標籤，將可以把想要執行的script都包含進這個index.html。

```htmlembedded=
<script src="./renderer.js"></script>
```
在路徑下新增一個名為render.js的檔案，內的程式皆可以使用原生javascript的api及工作來做前端的開發。像是使用webpack來壓縮程式碼，或是使用React來管理使用者介面等。


### <font color="#4078c0">回顧</font>

如果都有照上述的步驟來做的話，應該已經可以成功執行一個完整的Electron應用程式，像是以下範例

![](https://i.imgur.com/KyyIc12.png)

以下為完整的程式碼：
```javascript=
// main.js

// Modules to control application life and create native browser window
const { app, BrowserWindow } = require('electron')
const path = require('path')

function createWindow () {
  // Create the browser window.
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // and load the index.html of the app.
  mainWindow.loadFile('index.html')

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.
```

```javascript=
// preload.js

// All of the Node.js APIs are available in the preload process.
// It has the same sandbox as a Chrome extension.
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

```htmlembedded=
<!--index.html-->

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using Node.js <span id="node-version"></span>,
    Chromium <span id="chrome-version"></span>,
    and Electron <span id="electron-version"></span>.

    <!-- You can also require other files to run in this process -->
    <script src="./renderer.js"></script>
  </body>
</html>
```

總結上述所完成的步驟：

- 使用了Node.js的應用程式，並加入Electron
- 新增main.js的檔案作為執行的主程序，並可以控制該應用程式及在Node.js的環境中運行，此外在另一個程序中，使用了Electron內的app及BrowserWindow的套件來產生一個瀏覽器視窗來顯示網頁內容。
- 為了要使用Node.js內的一些功能，我們將preload的腳本放入BrowserWindow內。

### <font color="#4078c0">打包並發布應用程式</font>

要快速發佈最新的產生的app，可以使用Electron Forge這個套件。
1.在專案底下安裝Electron Forge套件
```
npm install --save-dev @electron-forge/cli
```

2.透過import指令去設定架設Forge

```
npx electron-forge import
```
這時候畫面結果會長這樣，代表有成功執行
![](https://i.imgur.com/7nb9UsZ.png)

3.透過Forge指令來發佈應用程式
```
npm run make
```
看到以下畫面代表你的應用程式已經成功被打包了
![](https://i.imgur.com/h3xaWHc.png)

打包完成的檔案會被放在專案路徑下的out的資料夾底下

```
// Example for macOS
out/
├── out/make/zip/darwin/x64/my-electron-app-darwin-x64-1.0.0.zip
├── ...
└── out/my-electron-app-darwin-x64/my-electron-app.app/Contents/MacOS/my-electron-app
```

![](https://i.imgur.com/4SEY1qs.png)

在electron-app-darwin-x64這個資料夾裡面，就可以看到可執行的檔案

Electrom Quick Start 這邊教學就到這邊，期待大家都可以做出自己第一個Elecrtron Application，就這樣 ㄅㄅ






