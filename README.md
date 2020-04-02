该版本改编自DesertsP 的 [Valine-Admin](https://github.com/DesertsP/Valine-Admin) ，增加了Server酱、短信和telegram推送功能。

## 特殊变量说明

| 变量名 | 备注 |
| ------------ | ------------ |
| SCKEY | [可选] Server酱的SCKEY |
| TG_TOKEN | [可选] Telegram bot token,比如 `123456789:abc` |
| TG_CHATID | [可选] Telegram chat id，比如 `12345678` ，如TG_TOKEN不为空，则必填|
| REDIR_URL | [可选] 用于代理发送telegram请求的地址如为空则不使用代理地址发送，如https://xxx.xxx/post.php|
| NICK_LEN | [可选] Telegram通知中昵称摘要长度，默认是 7 |
| COMMENT_LEN | [可选] Telegram通知中评论摘要长度，默认是 30 |
| SMS_URL | [可选] 用于发送短信通知的地址，如https://xxx.xxx/sms.php|

## 相关php文件
### post.php
```
<?php
//不是post直接返回
header('Content-Type: text/html;charset=utf-8');
if ($_SERVER["REQUEST_METHOD"] != "POST"){
    die( "{\"code\":500,\"errmsg\":\"bad request\"}");
}

$_POST =$_POST?$_POST:json_decode(file_get_contents('php://input'), true);
$url=$_POST['url'];
unset($_POST['url']);
$return=http_post_data($url,json_encode($_POST));
http_response_code($return[0]);
die($return[1]);
function http_post_data($url, $data_string) {
  $ch = curl_init();
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_URL, $url);
  curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
  curl_setopt($ch, CURLOPT_HTTPHEADER, array(
    "Content-Type: application/json; charset=utf-8",
    "Content-Length: " . strlen($data_string))
  );
  ob_start();
  curl_exec($ch);
  $return_content = ob_get_contents();
  ob_end_clean();
  $return_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
  return array($return_code, $return_content);
}
```

### sms.php
基于腾讯云自定义监控,[教程](https://ifking.cn/p/312.html)
```
<?php
//不是post直接返回
header('Content-Type: text/html;charset=utf-8');
if ($_SERVER["REQUEST_METHOD"] != "POST") {
    die('<script>alert("别闹（。");location.href="./";</script>');
}
define('SecretId','你的SecretId');
define('SecrSetKey','你的secretKey');
define('PolicyId','告警策略');
function sms_send($text, $desp = '')
{
    $param = array(
        'Action' => 'SendCustomAlarmMsg',
        'Region' => 'ap-shanghai',
        'Version' => '2018-07-24',
        'Module' => 'monitor',
        'PolicyId' => PolicyId,
        'Timestamp' => time(),
        'Nonce' => rand(0, 65535),
        'SecretId' => SecretId,
        "Msg"=>$text."\n".$desp,
    );
    ksort($param);
    $signStr = "GETmonitor.tencentcloudapi.com/?";
    foreach ($param as $key => $value) {
        $signStr = $signStr . $key . "=" . $value . "&";
    }
    $signStr = substr($signStr, 0, -1);
    $signature = base64_encode(hash_hmac("sha1", $signStr,SecretKey, true));
    return $result = file_get_contents('https://monitor.tencentcloudapi.com/?' . http_build_query($param).'&Signature='.urlencode($signature));
}
header('Content-Type: application/json;charset=utf-8');
$text = "";
$desp = "";
$_POST = $_POST ? $_POST : json_decode(file_get_contents('php://input'), true);
//获取post参数
if (isset($_POST["text"])) {
    $text = $_POST["text"];
}
if (empty($text)) {
    $json['errno'] = 1;
    $json['errmsg'] = "消息标题不能为空啦";
    die(json_encode($json));
}
if (isset($_POST["contact"])) {
    $contact = $_POST["contact"];
}
if (isset($_POST["desp"])) {
    $desp = $_POST["desp"];
}

echo sms_send($text, $desp);
```
