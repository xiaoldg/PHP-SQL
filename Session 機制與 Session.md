### SESSION 機制並非一開始就被內建在程式語言之中，這個機制的概念可以比擬「通行證」的設計，假設你是一位訪客，你造訪一棟大廈，警衛室發給你一張「通行證」，之後你只要出示這張通行證，我就會知道你是誰誰誰，並且我就認為你有通行的資格。

### 並且當你確定結束這趟造訪的時候，你需要把通行證繳交回來，斷開你與通行證的關係

***

### 這邊有三個重點：
#### 1.當使用者出示通行證的時候，我就會認為你是「誰」

#### 2.當使用者出示通行證的時候，我知道你有「通行的資格」

#### 3.當你確定「結束」這趟造訪的時候，「斷開」你與通行證的關係。

##### 第一點 : 也就是認證不認人，如果別人拿著你的通行證來驗證，我也會把那個人當作你，讓他通過。

##### 第二點 : 承上，只要你有證，你就有通行的資格。

##### 第三點 : 如同你造訪大樓一樣，離開時要換證，要把通行證還給警衛室，如此你就與這張通行證沒有關係了。

***
***

## 要加入我們留言板網站的會員驗證，該怎麼做呢？

#### 核心關鍵「通行證」該如何實現？
#### 將這個「通行證」用 Cookie 機制來實現，只要每次連線都可以帶上可以驗證我們身分的 Cookie，那我就不必每次都要輸入帳號密碼登入了。

#### 而由於「通行證」是大樓本身－也就是後端要發給我的，所以當我第一次登入Success的時候，後端若要發給我這張通行證 Cookie，需要使用以下語法：
```html
if ( verify === true ) {
 echo "登入Success ! ";
 setcookie("member_id", "001", time()+3600*24); //setcookit(name,value,有效時間)
}
```
#### 使用 setcookie 這個語法，我發送了一個名為 member_id 的 Cookie 給使用者，而這個 Cookie 過了 24 小時之後便會自動銷毀。

#### 而在驗證上，我做了一個最簡單的驗證，只要你名為 member_id 的 Cookie 值不為空，我就當作你擁有通行證，有通行資格
```html
// 使用者連線到網站

require_once('./conn.php');

if(!isset($_COOKIE["member_id"])) { 
 echo "目前是登出狀態";
} else {
 echo "目前是登入狀態";
}
```
### $_COOKIE["member_id"] 可讓後端接收名為 member_id 的 Cookie 的值，這邊可以看到我的驗證條件設定為「裡面有值」就能通過

### 這樣當然是非常非常非常非常不安全的，前面有提過，Cookie 是可以由使用者自己創造與修改的，所以我只要在瀏覽的時候自行創造一個名為 member_id 的 Cookie，然後在 Value 裡面打幾個字，我就可以登入Success了

***
***

## 「登出」功能
#### 當你按下登出時，發送一個同樣名為 member_id 但值為空的 Cookie，基於 Cookie 同名會覆蓋的特性，這份值為空的 Cookie 會覆蓋會員原先的那份 Cookie，如此等同於你擁有的 Cookie 無效
```html
// 按下登出之後

setcookie("member_id", "", time()+3600*24); // 雖然是賦予""，但其實是空值，而非空字串
header("Location: ./index.php");
```

***
***
***
***
***

## 更進一步的 Session 機制
#### 上面方法我們給予使用者 Cookie，讓使用者下次瀏覽網頁時自動帶上這個 Cookie 讓後端作驗證，但驗證的方法太糟糕，非常不安全，因為當時我們採用的是只要該 Cookie 內的 Value 不為空就能算是登入狀態
#### 所以這一週我們要發送一組 Cookie 驗證碼給使用者，讓使用者帶上這份驗證碼讓後端進行驗證，若驗證Success，則為登入狀態
### 建議可以使用 uniqid() 這個函數生成一個唯一 ID，夾在 Cookie 內給使用者，並同時將這個唯一 ID 存入一個我們新創的資料表中

```
if ( verify === true ) {
    echo "登入Success ! ";
    $uniqid = uniqid();
    setcookit("member_id", $uniqid, time()+3600*24);
}
```
#### 然後使用者登入時帶上這個 Cookie Value 時，則將該值與我們資料表中該會員擁有的唯一 ID 作比對
***
***
***
***
***
### php 引擎有內建語法可以幫我們實現這個機制，所以其實我們不用建立資料庫，也可以存放通行證資料

### 這個語法就是 session_start()，若要使用，必須放在網頁最上方，或者還沒有任何輸出之前
```html
if ( verify === true ) {
    session_start(); // 啟動 Session
    $_SESSION['member'] = true; // 註冊登入者為會員並且使其為 true
    $_SESSION['id_number'] = $row['id_number']; // 註冊登入成功的會員帳號與暱稱
    $_SESSION['nickname'] = $row['nickname'];
    echo "<script>alert('登入成功 !');parent.location.href='../index.php';</script>";
}
```
#### 當 session_start() 被執行時，會正式啟動內建 Session 機制，並且自動發送一個名為 PHPSESSION 的 Cookie 給使用者

#### 這個名為 PHPSESSION 的 Cookie，其 Value 是一組隨機亂數產生的亂碼，這一點就與我們先前使用 uniqid() 的方法不謀而合

#### PHPSESSION 是該 Cookie 的預設名稱，可以於伺服器這邊更改

#### 另外，我可以使用 $_SESSION 建立屬於這組 Session 的相關資料，如該使用者的帳號與暱稱等等，都可以自定義儲存在後端數據中
### 那要如何驗證呢 ? 這邊同樣要先使用 session_start()
```html
isLogin = false; // 先宣告是好習慣，避免全局變數汙染

session_start();
  if (isset($_SESSION['member']) && $_SESSION['member'] === true) {
    isLogin = true;　// 成功登入狀態
  } else {
    isLogin = false; // 非登入狀態
  }
echo json_encode($arr);
```
### 那要如何登出呢 ?

### 只要在後端這邊銷毀這個通行證即可，也就是銷毀這個 Session，下列兩種方法都可以
```html
// 刪除 Session 變量
unset($_SESSION['member'])

// 刪除整份的 Session
session_destroy();
```
#### 這邊就是銷毀通行證的方法，下次用戶再輸入帳密登入時再發新的給他就好

#### 另外就是 Session 也可以設置存在時間，這樣可以更完善通行機制，這邊就不多做說明了
