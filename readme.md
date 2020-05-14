# Hacker-Friendly Message Board
## 概述
- 台大電機所，網路攻防實習課程ㄉ作業
- 作業內容需要設計一款留言板。留言板內部需要有註冊、登入功能，並可供使用者上傳圖片。
- 作業繳交後會將Server開放給班上其他同學進行各種網路的攻擊。
- 身為一個駭客(X 玩資安的人，當然要在網站上面埋很多好玩的東西，讓想來玩我網站的人好好的被我玩囉XDDDDD

## 正常版功能
- 網站連結(我也不知道啥時會撤掉):http://meow.meow-meow.tk
### 首頁
- 網頁主要ㄉ風格採用大概20年前ㄉ舊風格，下載ㄌ很多古代ㄉ網頁素材
![](https://i.imgur.com/txBTaAg.png)
### 留言板
![](https://i.imgur.com/LG7G9L0.jpg)

## 好玩ㄉ功能
這些好玩ㄉ功能都讓各種來攻擊ㄉ同學玩ㄌ很久XDD

### 網頁伺服器
- 我ㄉ網頁副檔名都使用.php
![](https://i.imgur.com/gA2Qhr5.png)

- Header 裡面標注的也是PHP + Apache
![](https://i.imgur.com/jp8OqOz.png)

- 身為一個攻擊者，會根據不同的網頁伺服器採取不同的攻擊手段。但是事實上我採用的是Python的Flask，而不是PHP。透過這種手段來誤導駭客，做出一些浪費時間ㄉ事XDDD
```python=
class localFlask(Flask):
    def process_response(self, response):
        response.headers['server'] = 'Apache/2.4.38'
        response.headers['x-powered-by'] = 'PHP/5.5.13'
        super(localFlask, self).process_response(response)
        return(response)
application = localFlask(__name__)
```

### Fake SQL injection
- 我ㄉ網站透過ORM來防SQL injection。但是單純的防禦也太無聊了吧？！所以我在註冊跟登入的地方塞了一些好玩ㄉ東西。
- 攻擊者如果在登入視窗輸入SQLi最常見的單引號，會回傳MySQL的Error
![](https://i.imgur.com/DJV8hbP.png)
- 但這個功能事實上的實現方式是...
```python=
if '\'' in name:
    rt = '1064 - You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near \'{}\' at line 1.'.format(name)
    return render_template_string('{{meow}}',meow=rt)
```
- 依照Log看起來，非常多人在這邊玩ㄌ很久XDDD


### 彩蛋網頁
- 在首頁的原始碼中，我有放了三個彩蛋網頁的註解
```html=
<!-- <br><a href="./phpmyadmin.php">phpmyadmin</a>
<br><a href="./phpinfo.php">phpinfo</a>
<br><a href="./admin">admin_page</a> -->
```
- phpMyAdmin
進入網頁會呈現一個常見的phpMyAdmin
但是他只是假ㄉ空殼
![](https://i.imgur.com/ZgpaEh1.png)

- PHP info
就是...很普通的PHP info 網頁
![](https://i.imgur.com/M9my75l.png)

- Admin page
會到一個我們國小很常出現的嚇人網頁
如果在上述的phpMyAdmin送出帳密也會到這ㄍ網頁
嘻嘻嘻嘻
![](https://i.imgur.com/xkYzBaR.png)

### Error Handling
無論是對付掃路徑，或是用各種方法放入非預期結果想讓伺服器壞掉的話。我ㄉ網站都不會回應常見的Error Code如400、403、404、500。

回應的部分我ㄉ伺服器永遠都只會回應200 OK
![](https://i.imgur.com/vJybAPo.png)

## 正常ㄉ防禦
- 圖片上傳奇怪ㄉ檔案：用Pillow強制讀進來改尺寸
    - Pillow讀到壞掉就壞掉ㄌ
- XSS
    - 用render_template內建ㄉ防禦
- CSRF
    - 增加CSRF Token
- SSTI
    - 小心ㄉ使用render_template_string
