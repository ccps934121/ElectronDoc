# <font color="#4078c0">程序模型</font>
###### tags: `Electron`

原文：[Electron - Process Model](https://www.electronjs.org/docs/tutorial/process-model#the-multi-process-model)

Hackmd:[連結](https://hackmd.io/@RdUg6kDRTcKHmb2WGCucdA/r1KB-Qvcd)

Electron從Chromium繼承了多程序的構造，讓框架可以與現代網頁瀏覽器端非常的相似。在這章節中，會解釋一些在上一篇Quick Start Electron所用到的概念。

---

### <font color="#4078c0">為什麼不是單一程序？</font>
網頁瀏覽器是一個相當複雜的應用程式，除了顯示網頁的主要功能外，還有其他的職責例如管理多個視窗(頁籤)、管理第三方套件的擴充。
在之前，瀏覽器是使用單一程序來執行它的功能，雖然這樣代表在開啟其他新頁籤時不會有過載的問題，但這同時也代表了當一個網頁卡在那邊時將會影響到整個瀏覽器。

### <font color="#4078c0">多程序模型</font>
為了要解決這個問題，Chrome團隊決定將每個頁籤都有一個獨立的程序來做渲染的動作，減少網頁錯誤或惡意的攻擊對應用程式的損害，然後瀏覽器以單程序模式控制這些程序以及該生命週期。
以下圖表為Chrome Comic所提供的視覺化模型：

![](https://cdn.rawgit.com/electron/electron/v13.1.0//docs/images/chrome-processes.png)

Electron的結構與此十分相似。身為一個開發者，你可以控制兩個型態的程序：main 及 renderer，這些與上方所述Chrome的瀏覽器及渲染進程類似。

### <font color="#4078c0">主程序</font>
每個Electron程式都有一個單一的主程序作為該應用程式的入口點，該主程序主要是在Node.js中執行，這代表該程序可以去取用Node.js內所有的模組。

#### <font color="#4078c0">視窗管理</font>
主程序基本的目的就是要透過<font color="#AE0000">**BrowserWindow**</font>模組來新增及管理一個應用程式的視窗。

BrowserWindow的實例會產生一個獨立的應用程式視窗，在該視窗渲染器進程中載入網頁，你可以從主程序中透過使用視窗內的webContents物件來與網頁的內容互動。

```javascript=
const { BrowserWindow } = require('electron')

const win = new BrowserWindow({ width: 800, height: 1500 })
win.loadURL('https://github.com')

const contents = win.webContents
console.log(contents)
```

> Note: 渲染器的程序也可以透過BrowserView來產生一個嵌入式的網頁，那這邊一樣也可以透過webContents來嵌入網頁的內容。☞ [BrowserView用法](https://www.electronjs.org/docs/api/browser-view)

因為BrowserWindow模組是一個自定義的事件，所以你可以增加各式各樣的處理程序在該事件中(ex.定義視窗的大小)

#### <font color="#4078c0">應用程式的生命週期</font>
這個主程序主要也透過了Electron內的app模組控制整個應用程式的生命週期，這個模組提供了許多的事件以及方法讓使用者可以加入自定義的行為(例如：已編程方式來離開應用程式、修改應用程式的圖示及顯示About的功能)。

以下為一個實際的案例，在quick start文章有提到可以使用app內的api來創建更多原生應用程式的視窗體驗。

```javascript=
// quitting the app when no windows are open on macOS
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})
```

#### <font color="#4078c0">原生API</font>
為了要擴展Electron的功能，讓它不只是只為了顯示網頁內容的視窗瀏覽器，這個主程序增加了許多自定義的API來與使用者操作的系統互動。Electron釋出各式各樣的模組來操控原生的桌面功能，像是功能列、對話框等功能。

想了解更多Electron主程序的模組，可以查看官網API的文件。

### <font color="#4078c0">渲染程序</font>
每個Electron應用程序在開啟BrowserWindow(或嵌入網頁)時會產生一個獨立的渲染程序，就如它的名字所示，渲染器將會負責渲染的網頁內容，它主要的目的就是基於網頁標準的開發模式上運行渲染的程序(至少Chrome瀏覽器是如此)
因此，所有應用程式上的使用者界面或是功能都可以跟在開發網頁所使用的工具或是範例寫法一致。

雖然解釋Web的規範超過本章的範圍，不過有些基礎還是需要了解一下：

-  html的檔案是渲染器渲染的入口點
-  界面的樣式主要是來自CSS
-  要執行的js可以透過<script>頁籤包著

然而，這也代表著渲染器無法直接使用require或Node.js內的API，所以為了要在渲染器中引用NPM的模組，必須與網頁上所使用的打包工具(例如：webpack、parcel)一致。


> Note:在以前的預設是渲染器的程序可以使用Node.js去生成該環境，以便於開發，但是現在基於安全因素的考量所以已經禁用了此功能。

在這，你可能會想知道渲染的使用介面要如何與Node.js及Electron原生的桌面功能互動，當這些功能只能透過主程序來訪問時。事實上，這邊沒有直接的方法去匯入Electron的腳本。

### <font color="#4078c0">Preload Script</font>
預載腳本內所包含的程式主要是在網頁內容載入之前去做渲染的程序，這個腳本在渲染的環境中運行，但可以透過訪問Node.js的API去獲得更多的權限。

預載的腳本可以放在主程序中的BrowserWindow function內的webPreferences選項中，範例如下：

```javascript=
const { BrowserWindow } = require('electron')
//...
const win = new BrowserWindow({
  preload: 'path/to/preload.js'
})
//...
```

因為這個預載腳本可以共享使用者介面，並可以去訪問Node.js的API，它可以使用各種window內全域的API來增強渲染器的功能，並讓網頁的內容可以去使用這些API。

雖然預載的腳本與渲染器共享一個全域的視窗，但因contextIsolation預設值的關係，你依然不行直接將預載腳本內的變數直接用在該渲染器的程式內。

```javascript=
// preload.js
window.myAPI = {
  desktop: true
}
```

```javascript=
// renderer.js
console.log(window.myAPI)
// => undefined
```
context Isolation的意思是預載的腳本在渲染器內是被隔離的，原因是因為要避免在網頁的程式內洩漏任何的API。
所以如果真的要在渲染器內使用preload.js內所定義的變數，可以在preload.js內使用contextBridge這個模組來達到目的：
```javascript=
// preload.js
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('myAPI', {
  desktop: true
})
```
這時候可以看到console出來的結果，有成功取得剛剛所定義的物件了
```javascript=
// renderer.js
console.log(window.myAPI)
// => { desktop: true }
```
這個功能在這兩的主要的目的中非常的實用：

- 透過暴露ipcRender給渲染器，可以使用程序間通訊(IPC)從渲染器來觸發主程序的任務。
- 如果你正在用Electron開發既有的Web或遠端URl的應用程式，可以在渲染器的視窗上使用自定義屬性，這將可以讓網頁的客戶端使用只限於桌面的邏輯。