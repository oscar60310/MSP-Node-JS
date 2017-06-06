###### tags:`msp`
# 在 Azure 上部署 Node.js Web app 網頁應用程式逐步教學[for Mac OS]
window 使用者可以參考[課程影片](https://channel9.msdn.com/Series/MSP-TW/Node-JS-Azure)
## ==1.安裝 Node.js==
首先，到[ Node.js 官方網站](https://nodejs.org/en/)安裝適合自己作業系統的版本，依照安裝指示的步驟即可
![](https://i.imgur.com/IGnLvBd.png)
![](https://i.imgur.com/tTRsSst.png)
![](https://i.imgur.com/hPrJO4D.png)
![](https://i.imgur.com/iFWKg9E.png)

完成安裝的最後一個步驟會告訴你 Node.js 安裝的位置在哪，接著我們可以打開終端機來看看，到它告訴我們的路徑裡，看看 npm 和 node 是否真的安裝成功
![](https://i.imgur.com/fQ5mtEb.png)
## ==2.建立 Azure web service==
1. 首先你要擁有一個 Microsoft 的帳號
2. 再來要有一個 Azure 的訂用帳戶，可以登入 Azure Portal，或者如果你是學生，可以選擇 [Microsoft Imagine](https://imagine.microsoft.com/zh-TW) 得到一個學生專屬訂用帳戶來進入 Azure 的服務
3. 進入 [Azure Portal](https://portal.azure.com)
4. 新增一個 Web應用程式做為我們的網站空間
![](https://i.imgur.com/94Y234a.png)
5. 輸入一個你喜歡的名稱及資源群組，按下建立後就會成立一個你專屬的網站，網址會是 *[你剛剛輸入的名稱].azurewebsites.net*
![](https://i.imgur.com/YVadJgq.png)
6. 你可以到 “資源群組” 或 “所有資源” 或 “應用程式服務” 中查看已建立好的應用程式，並看到這個 Web應用程式的網址
![](https://i.imgur.com/m8oZdxQ.png)
7. 在瀏覽器中輸入網址我們會看到一個空白頁面像這樣
![](https://i.imgur.com/4EtbXbh.png)

## ==3.用 Node.js 撰寫網頁上的內容並在本機測試==
你可以選擇你所習慣的開發環境像是 sublime text, nopad++ 等等，在這裡我選擇使用微軟的 Visual Studio Code 來做為開發工具。
1. 首先我們要建立一個資料夾，用來包含接下來所有檔案，我把它取名叫 **"helloMSP"**
2. 將以下程式碼的檔案取名為 **"server.js"** ，並存到helloMSP資料夾，它是用來建立 Node.js 的 server 端
```
 var http = require('http')
 var port = process.env.PORT || 1337;
 http.createServer(function(req, res) {
   res.writeHead(200, { 'Content-Type': 'text/plain' });
   res.end('hello MSP\n');
 }).listen(port);
```

3. 再來，在 **"helloMSP"** 資料夾中開啟 command line，輸數以下指令來啟動 node 的 server 在本機端先做測試
```
 node server.js
```
![](https://i.imgur.com/47zqYGI.png)
4. 開啟你的網頁瀏覽器網址輸入 "http://localhost:1337" 即可看到剛剛所建立的程式碼內容
![](https://i.imgur.com/IkrFcXd.png)


## ==4.建立 Azure 中的部署設定==
1. 在你的 Web應用程式中選擇部署選項然後選 “本機Git儲存機制” 並按確認
![](https://i.imgur.com/7M42YKi.png)
2. 再來點 “部署認證” 設定一組帳號密碼
![](https://i.imgur.com/sjudRuc.png)
3. 最後到 “屬性視窗“ 中，可以看到一些部署的相關URL，如FTP、Git等等
![](https://i.imgur.com/8vbpWtJ.png)

## ==5.將剛剛的程式碼使用 Git 部署到 Azure==
首先電腦中要可以使用 Git ，可以在 command line 中輸入以下指令檢查 git 版本
```
git --version
```
如果發現沒安裝可以到[官方網頁](https://git-scm.com/download)先去做安裝。
0. 在 command line 中回到 **"helloMSP"** 資料夾準備做部署到 Azure 的動作
1. 初始化一個空的 git repository 在本機端 **"helloMSP"** 資料夾
```
git init
```
2. 將檔案先新增到本機端剛剛創立的 repository，並加入一個註解說明是 initial commit
```
git add .
git commit -m "initial commit"
```
3. 接下來的指令是告訴電腦該將這個資料夾剛剛使用 git add 的檔案，加到 Git remote 遠端的何處。我們要加到的位置便是剛剛所建立的 Azure Web app 中，接著就是推到 Azure
```
git remote add azure [remote GIT URL(Azure web app 屬性中的 GIT URL)]
git push azure master
```
URL 會長的類似以下這樣：
*https:// ==你的名稱==@==你的名稱==.scm.azurewebsites.net:443/==你的部署帳號==.git*

4. 最後請輸入在 Azure Web app 中 "部署認證" 所設定的密碼，就可以完成部署了
注意密碼在你打鍵盤的時候是不會出現在銀幕上的
- 步驟及結果參考如下
![](https://i.imgur.com/mkyaaTA.png) 

## ==6.查看部署結果==
在瀏覽器中再次輸入 Azure Web app 的網址便可以看到剛剛部署的結果了
![](https://i.imgur.com/EFzu3fU.png)

## ==7.若有修改檔案如何更新==
1. 使用Git
若有修改剛剛 **server.js** 檔案，可以在修改檔案後，在終端機輸入以下指令，再次部署
```
git add .
git commit -m "[你可以在此解釋修改動作的comment]"
git push azure master
```
2. 使用 ftp filezilla
有初始部署之後，也可以使用熟悉的 ftp 上傳，
- 需要注意的地方是 ftp 主機為 Azure portal 中 "ftp://" 後面所接續的那串網址
![](https://i.imgur.com/Po25mYT.png)
- 帳號是部署認證中所設定的帳號，前面加上此 Web app 的名稱加反斜線（\）
如：LilyNodejs\LilyFTPaccount
- 密碼同部署認證中所設定的密碼
![](https://i.imgur.com/KktK3qS.png)

- 成功連線後即可使用 ftp 做更新
![](https://i.imgur.com/dmoEabE.png)

# 使用 Node.js 開發的例子：
一般網站：https://fandorashop.com/tw/
電腦應用程式：spotify
手機應用程式：

# 參考教學
- [試用 Azure App Service一小時](https://azure.microsoft.com/zh-tw/try/app-service/web/?language=nodejs)
- [Node.js 開發人員中心](https://azure.microsoft.com/zh-tw/develop/nodejs/)
- [發布至Azure的三種方法](http://blog.ibbhub.info/2016/07/nodejs-web-application-azure-web-app.html)
- [Microsoft document](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-nodejs-develop-deploy-mac)
# 其他
- [How to node教學網站](https://howtonode.org)
- [ 使用 NODE.JS 來達成電腦網頁與手機網頁即時互動](https://blog.patw.me/archives/566/node-js-using-node-js-to-achieve-interaction-between-web-and-mobile-web-pages-in-real-time/)