# sso
单点登录<br>
之所以拿出来分享，是因为在此之前面试官问道，也有其他同事或不认识的人问道。故研究并分享出来。<br>
联系方式：https://github.com/taozywu/JD


## 原理
1）假设有多个站点（a.com;b.com;c.com;d.com...），还有个passport站（u.com）<br>
2）登录：当你从a.com登录，提交会跳到u.com登录操作，u.com会去验证用户和密码是否匹配对，如果匹配不正确则返回code码回调给a.com，a.com拿到code码进行相应处理；如果匹配正确则回调给a.com中会带上sid（凭证），a.com拿到sid后进行解密得出可能有的参数（uid，uname，uexpiretime，uip。。。），如果说a.com有自己的用户表，则可以拿uid去验证下。验证正常后则在a.com保存下会话即可。<br>
3）登出：当你从a.com登出，先会清除掉a.com上的会话，然后跳到u.com进行登出操作，操作完后回调到a.com。<br>
4）自动登录：当访问b.com，则会跳到u.com去通过cookie拿到sid（凭证），然后回调给b.com中带上该参数，后续验证2）逻辑中厚半部分。<br>

### 使用说明
// 登录入口页面。<br>
public function testSSOAction()<br>
{<br>

    session_start();
    if (isset($_SESSION['UID']) && $_SESSION['UID'] > 0) {

        die("欢迎你，" . $_SESSION['UNAME'] . "<a href='/index/index/testSSOLogout'>退出</a>");
    }
}

// 登出页面。<br>
public function testSSOLogoutAction()<br>
{<br>
    session_start();<br>

    unset($_SESSION['UID']);
    unset($_SESSION['UNAME']);

    session_destroy();

    header("Location:http://ucenter.dev/?c=SSO&a=logout&back_url=http://hardyaf.dev/index/index/testSSO");
    exit;
}<br>

// 登录。<br>
public function testDoLoginAction()<br>
{<br>

    session_start();
    $uucode = (int) $_GET['uucode'];

    $uid = 0;
    $uname = "";

    if ($uucode == 1) {
        $sid = trim($_GET['sid']);
        if (!$sid) {
            die("sid is null");
        }

        // 加载路径
        \YYcenterSDK\Ucenter::load();
        // 验证auth
        $result = \YYcenterSDK\Ucenter::getCasAuthRs();

        $uid = isset($result[1]) ? $result[1] : 0;
        $uname = isset($result[0]) ? $result[0] : "";
    } else {
        header("Location: /index/index/testSSO");
    }


    if ($uid > 0) {

        $_SESSION['UID'] = $uid;
        $_SESSION['UNAME'] = $uname;

        header("Location: /index/index/testSSO");
    } 

    exit;

}<br>
