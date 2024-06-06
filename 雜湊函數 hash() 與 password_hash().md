## hash()

```
echo hash('sha256','mypassword');
```
#### hash 有兩個參數，一個是我要使用哪種 Hash 演算法 ( 演算法介紹請參考這裡 )，後者是要被 Hash 的值。

#### 而這邊其實還有第三個參數，是一個 Boolean，若為 Ture 則輸出二進位的資料，若為 False，則輸出小寫十六進位資料。

***
## password_hash() 與 password_verify()

```
$passwordHash = password_hash('mypassword', PASSWORD_DEFAULT);
```
#### password_hash() 這個函數會回傳一個先被「隨機加鹽」再 Hash 後的值，所以其實這個函數內建加鹽。
#### 第一個參數放要被 Hash 的值，第二個參數是要使用哪種 Hash 設定。
#### PASSWORD_DEFAULT 則是使用預設的 Hash 法，但要注意既然是 php 預設，則不同版本的 php 預設也都會不同

***
### 要注意同樣的值在每一次的 password_hash() 的結果都會不同，所以需要 password_verify() 來作驗證

```
  $input = '123';

  $passwordHash = password_hash($input, PASSWORD_DEFAULT);

  $isVerify = verify_password($passwordHash,'123');
  ```
  #### verify_password 回傳一個 Boolean，從上圖很清楚知道，第一個參數放的是要比對的　Hash Value，第二個參數要放的是比對值
  #### 若比對Success回傳 Ture，否則為 False。
