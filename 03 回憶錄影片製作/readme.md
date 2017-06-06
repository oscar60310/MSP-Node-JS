Node js 回憶影片課程討論
===
這次的專案使用到了 Facebook Graph API，還不熟悉的讀者，可以參考我們先前的[文章](https://blogs.msdn.microsoft.com/microsoft_student_partners_in_taiwan/2017/04/06/nodejs/)，或是到 [Facebook 開發人員中心](https://developers.facebook.com/docs/graph-api)查看說明。
## **簡介**
這次的課程我們會沿用上一次[文字雲](https://blogs.msdn.microsoft.com/microsoft_student_partners_in_taiwan/2017/04/06/nodejs/)課程的 Facebook Graph API，來繼續實作回憶錄的功能，可以讓大家選擇自己在 Facebook 的照片變成一部回憶錄的影片並在網頁顯示出來，那除了上次課程使用到的 [VS Code](https://code.visualstudio.com/)，還需要另外安裝 [NodeJs (7.6.0)]( https://nodejs.org/en/download/current/) 以上的版本，原因會在後面程式碼為大家解說。

這次課程除了讓大家複習 Facebook Graph API 認證程序，也會帶大家使用套件來進行撰寫，主要在 NodeJS 的撰寫上，所以前端的部份我們不會解釋，如果有任何疑問，都可以在下方留言或進入社團跟我們交流。

這次完成的作品
![](https://i.imgur.com/lGabHn4.jpg)

## **使用 NodeJs 實作授權及取得資料**
一樣，在開始之前，請前往[下載](https://github.com/alice0329/fbvedio-student)課程檔案，裡面包還含了 npm 設定、VS code 設定、ffmpeg執行檔及前端資源。

### **申請應用程式**
這個部分請參考上一篇 Node js 文字雲課程討論，步驟都是一樣的，不過因為這次範例我們是使用**粉絲專頁存取權杖**，用來取得使用者粉絲專業的資料，所以在**導向授權頁面**，我們傳回三個參數中的 scope，除了 user_posts 這個權限以外，我們還需要加入 **manage_pages**、**user_photos**，之後再選擇同意授權，就可以取得我們想要的資料了。

讓我們看一下程式碼修改之後的樣子吧。
```javascript
app.get('/api/user', (req, res) => {
    var re;
    if (req.session.name)
        re = { statu: 'ok', name: req.session.name };
    else
        re = { statu: 'not login', url: 'https://www.facebook.com/v2.8/dialog/oauth?client_id=' + process.env.appID + '&redirect_uri=' + process.env.redirect + '/api/code&scope=user_posts&scope=manage_pages&scope=user_photos' };
    res.setHeader('Content-Type', 'application/json');
    res.end(JSON.stringify(re));
});
```
## **使用者相簿資料**
我們先使用[測試工具](https://developers.facebook.com/tools/explorer/
)來測試一下結果。

請按下**取得權杖 → 取得用戶存取權杖**，勾選 user_posts、manage_pages、user_photos。
![](https://i.imgur.com/MQkosSj.png)

接著試試看 **me/albums** 端點，可以看到有好幾個 id，一個 id 就是用戶的一個相簿，隨便點選一個 id，就是去搜尋那個相簿裡面的資料，然後再到左手邊的**搜尋欄位**輸入 photos 以及 images，按下**提交**。
![](https://i.imgur.com/t1KKowT.png)

就可以看到相簿裡面每一張不同大小相片的網址。
我們要取得相片的網址並將它下載至本機端。

在看 code 之前，我們先來介紹今天 code 會用到的一些套件，並於程式中require這些套件。
| 引用的套件| 功用 |
| ------ | ----------- |
| express | Express 是最小又靈活的 Node.js Web 應用程式架構，為 Web 與行動式應用程式提供一組健全的特性 |
| express-session | 將資料儲存在伺服器上 |
| request | 向 http 發出請求 |
| fs | fs 是 filesystem 的縮寫，提供文件的讀寫能力 |
| videoshow | 簡單的實用工具，使用 ffmpeg 來製作圖像的視頻幻燈片，例如音頻，字幕和幻燈片之間的淡入/淡出轉換等附加功能 |
| sharp | 用來處理圖片 |



說這麼多先來看看 code 怎麼打。
```javascript
// GET 取得相簿資料
app.get('/api/albums', (req, res) => {
    getAlbums(req.session.key).then((data) => {
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify(data));
    });
});
// 讀相簿列表 
function getAlbums(key) {
    return new Promise((resolve, reject) => {
        var qs = {
            access_token: key
        };
        request({
            url: 'https://graph.facebook.com/v2.8/me/albums',
            qs
        }, (error, response, body) => {
            resolve(JSON.parse(body));
        });
    });
}
```
將資料傳送到前端，就可以選擇其中一個相簿。

那我們就接著繼續完成 Code 吧
```javascript
// GET 取得相片資料傳給 getImage
app.get('/api/create', (req, res) => {
    getPhotos(req.session.key, req.query.id).then((data) => {
        getImage(data, res);
    });
});
// 讀取特定相簿的相片列表
function getPhotos(key, id) {
    return new Promise((resolve, reject) => {
        var qs = {
            fields: 'images',
            access_token: key
        };
        request({
            url: 'https://graph.facebook.com/v2.8/' + id + '/photos',
            qs
        }, (error, response, body) => {
            var photo_data = JSON.parse(body);
            var photos = [];
            for (var i in photo_data.data) {
                photos.push(photo_data.data[i].images[0].source);
            }
            resolve(photos);
        });
    });
}
```
做到這裡就可以取得相片的網址囉。
## **合成影片**
### 步驟
取得相片列表後，我們使用前面介紹的 videoshow 模組把這些相片接成一個回憶影片。但是這個模組輸入的圖片一定要是相同大小，所以合成影片我們分成三個步驟：

![](https://i.imgur.com/2y9RrVr.png)

在執行這三個步驟之前，我們要先清空存放下載圖片的 image 資料夾、存放調整過大小的 image_resized 資料夾，並確保兩個資料夾存在，以避免合成影片時發生錯誤。

接著我們就來寫 code 囉！首先先來看我們整個合成影片的步驟的 function。剩下的所有步驟都包含在這個 function 裡面了。但是我們每個步驟都需要執行完才能執行下一個步驟，結果才能完整呈現、不會出錯。所以我們要處理 javascript 的非同步問題。

記得上次課程用來處理非同步問題的 Promise 嗎? 這次我們介紹 async/await 的寫法搭配 Promise 使用，可以讓你的程式碼更好讀易懂！

```javascript
async function getImage(key, res) {
    try {
        await clearFolder(); // 清空image資料夾和image_resized資料夾
        await urlImage(key); // 下載照片
        for (let i = 0; i < key.length; i++) {
            await imageResize("./image/" + i + ".jpg", "./image_resized/" + i + ".jpg", 960, 720); // 重設圖片大小
        }
        videoGen(res); // 合成影片
    } catch (e) {
        console.log(e);
    }
}
```
在function前面加"async"，function裡面就可以使用await。await可以接Promise物件。await要接到這個非同步的結果回傳後，才會執行下一行。

以下分別介紹每個步驟的function。
## 1.清空資料夾
```javascript
function clearFolder() {  
    return new Promise((resolve, reject) => {
        var fileUrl = "./image";
        var fileUrl2 = "./image_resized";
        if(fs.existsSync(fileUrl)){
            var files = fs.readdirSync(fileUrl);
            files.forEach(function (file) {
                fs.unlinkSync(fileUrl + '/' + file);
                console.log("删除文件" + fileUrl + '/' + file + "成功");
            });
        }
        else{
            fs.mkdirSync(fileUrl);
        }     
        if(fs.existsSync(fileUrl2)){
            var files = fs.readdirSync(fileUrl2);
            files.forEach(function (file) {
                fs.unlinkSync(fileUrl2 + '/' + file);
                console.log("删除文件" + fileUrl2 + '/' + file + "成功");
            });
        }
        else{
            fs.mkdirSync(fileUrl2);
        } 
        resolve();
    })
}
```

在刪除資料夾內的檔案之前，要先確認資料夾存在 ( existsSync() )，否則程式會出錯。如果檢查到資料夾不存在，就要建一個資料夾 ( mkdirSync() )，等等才有地方存放圖片。
因為我們有兩個資料夾 (./image、./image_resized) 都要清空，所以大家會看到我們幾乎相同的code重複了兩次。
## 2.下載圖片
```javascript
function urlImage(url) {   
    return new Promise((resolve, reject) => {
        url.forEach(function (url_data, index, array) {
            let stream = request(url_data).pipe(fs.createWriteStream("./image/" + index + ".jpg"))
            if (index === array.length - 1)
                stream.on('finish', () => resolve())
        })
    })
}
```
到這裏我們就成功把選擇的資料夾裡面的照片全部下載到 image 資料夾了。接下來我們要將圖片重設大小來配合 videoshow 套件的要求。
## 3.重設圖片大小
```javascript
function imageResize(imgPath, imgPath_resized, width, height) {
    return new Promise((resolve, reject) => {
        sharp(imgPath)
        .resize(width, height)
        .ignoreAspectRatio()
        .toFile(imgPath_resized, (err, info) =>{
            resolve()
        });
    })
}
```
這裡我們用到 sharp 這個套件。如果你的 node_modules 資料夾裡面沒有 sharp這個資料夾，記得要先執行 npm install sharp 來安裝套件，並引入 sharp 模組。
```javascript
var sharp = require('sharp');
```
## 4.使用 videoshow 合成影片
在使用 videoshow 之前，也要先安裝套件：
```javascript
var videoshow = require('videoshow');
```
然後到 [FFmpeg 官網](http://ffmpeg.org/download.html) 去下載 FFmpeg 的binary檔案 (ffmpeg.exe 和 ffprobe.exe)。
並在程式中加入下面這兩行，並注意檔案路徑：
```javascript
videoshow.ffmpeg.setFfmpegPath(__dirname + '/ffmpeg/ffmpeg.exe')
videoshow.ffmpeg.setFfprobePath(__dirname + '/ffmpeg/ffprobe.exe')
```
我們在合成影片的時候需要設定影片和音樂的的一些參數：
```javascript
var videoOptions = {
    fps: 25,
    loop: 5, // seconds
    transition: true,
    transitionDuration: 1, // seconds
    videoBitrate: 1024,
    videoCodec: 'libx264',
    size: '640x?',
    audioBitrate: '128k',
    audioChannels: 2,
    format: 'mp4',
    pixelFormat: 'yuv420p'
}

var audioParams = {
    fade: true,
    delay: 2 // seconds
}
```
然後就是重要的影片合成啦！
```javascript
function videoGen(res) {
    var images = [];
    fs.readdir(__dirname + '/image_resized', function (err, files) {
        if (err) return;
        files.forEach(function (f) {
            images.push(__dirname + '/image_resized/' + f );
            //console.log('Files: ' + f);
            return images;
        });
        console.log(images);
        var audio = __dirname + '/music/About_That_Oldie.mp3'
        videoshow(images, videoOptions)
            .audio(audio, audioParams)
            .save('./static/output/video.mp4')
            .on('start', function (command) {
                console.log('ffmpeg process started:', command)
            })
            .on('error', function (err) {
                console.error('Error:', err)
            })
            .on('end', function (output) {
                console.log('Video created in:', output)
                var re;
                re = {
                    statu: 'ok',
                };
                res.setHeader('Content-Type', 'application/json');
                res.end(JSON.stringify(re));
            })
    });
}
```
我們先去 image_resized 把照片一張一張讀出來 ( readdir() )，然後塞進 image 陣列中。再指定音樂的路徑後，就可以用 videoshow 把影片做出來了。在影片合成完之後，就傳送成功的訊息給前端，在前端顯示合成完的影片。

在執行之前，不要忘記把前面取得相簿資料那步驟的傳送資料動作註解掉喔！
```javascript
app.get('/api/albums', (req, res) => {
    getAlbums(req.session.key).then((data) => {
        //res.setHeader('Content-Type', 'application/json');
        //res.end(JSON.stringify(data));
    });
});
```


這樣就完成我們整個回憶錄的製作了！

# 總結
看過這篇文章後，你應該對 Facebook Graph API 的影片應用有更進一步的瞭解了，可以前往 [範例 Github](https://github.com/alice0329/fbvideo) 看看完整程式，我們還有其他課程，若有興趣，可以加入[課程社團](https://www.facebook.com/groups/126283061232907/) 一起討論！

第十一屆微軟學生大使　技術組　王采楓、何天與　撰寫

## Node JS 技術小聚課程
1. [Node js 實作課程：文字雲](https://blogs.msdn.microsoft.com/microsoft_student_partners_in_taiwan/2017/04/06/nodejs/)
2. [Node js 實作課程：回憶影片](http://blogs.msdn.microsoft.com/microsoft_student_partners_in_taiwan/2017/05/09/nodejs-2)
3. [Node js 實作課程：小遊戲製作]( http://blogs.msdn.microsoft.com/microsoft_student_partners_in_taiwan/2017/05/12/nodejs-2)
