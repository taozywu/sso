# sso
单点登录<br>
之所以拿出来分享，是因为在此之前面试官问道，也有其他同事或不认识的人问道。故研究并分享出来。

## 使用说明

// 登录入口页面。
public function testSSOAction()
{

    session_start();
    if (isset($_SESSION['UID']) && $_SESSION['UID'] > 0) {

        die("欢迎你，" . $_SESSION['UNAME'] . "<a href='/index/index/testSSOLogout'>退出</a>");
    }
}

// 登出页面。
public function testSSOLogoutAction()
{
    session_start();

    unset($_SESSION['UID']);
    unset($_SESSION['UNAME']);

    session_destroy();

    header("Location:http://ucenter.dev/?c=SSO&a=logout&back_url=http://hardyaf.dev/index/index/testSSO");
    exit;
}

// 登录。
public function testDoLoginAction()
{

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

}
