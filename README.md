const cookieKey = 'my_tb_coin_cookie';
const $ = new Loon(); // 假设你使用的是基础工具类

if (typeof $request !== 'undefined') {
    // --- 获取 Cookie 逻辑 ---
    const cookie = $request.headers['Cookie'] || $request.headers['cookie'];
    if (cookie) {
        $.setdata(cookie, cookieKey);
        $.notification("淘宝金币", "Cookie 获取成功", "现在可以关闭重写脚本了");
    }
    $.done({});
} else {
    // --- 自动领取逻辑 ---
    const savedCookie = $.getdata(cookieKey);
    if (!savedCookie) {
        $.notification("淘宝金币", "领取失败", "未检测到有效 Cookie，请先登录并进入页面");
        $.done();
    }

    const request = {
        url: 'https://guide-acs.m.taobao.com/gw/mtop.taobao.ms.coins.entry/1.0/',
        headers: {
            'Cookie': savedCookie,
            'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 15_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148'
        }
    };

    $.post(request, (error, response, data) => {
        if (error) {
            $.notification("淘宝金币", "请求错误", error);
        } else {
            const res = JSON.parse(data);
            if (res.ret[0].includes("SUCCESS")) {
                $.notification("淘宝金币", "领取成功", "今日金币已到账");
            } else {
                $.notification("淘宝金币", "领取失败", res.ret[0]);
            }
        }
        $.done();
    });
}

// 简易 Loon 环境适配器
function Loon() {
    this.getdata = (key) => $persistentStore.read(key);
    this.setdata = (val, key) => $persistentStore.write(val, key);
    this.notification = (title, subtitle, content) => $notification.post(title, subtitle, content);
    this.post = (opts, cb) => $httpClient.post(opts, cb);
    this.done = (obj = {}) => $done(obj);
}
