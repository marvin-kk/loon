/**
 * [Script]
 * http-request ^https:\/\/guide-acs\.m\.taobao\.com\/gw\/mtop\.taobao\.ms\.coins\.entry script-path=taojinbi.js, tag=抓取金币Cookie
 * cron "0 9 * * *" script-path=taojinbi.js, tag=自动领金币
 */

const KEY_COOKIE = "tb_coin_cookie";
const KEY_HEADERS = "tb_coin_headers";

// 判断是 Loon 拦截请求还是定时任务触发
const isRequest = typeof $request !== 'undefined';

if (isRequest) {
    // --- 抓取模块 ---
    // 捕获请求头，里面包含了淘宝生成的签名和 Session
    const headers = JSON.stringify($request.headers);
    const cookie = $request.headers['Cookie'] || $request.headers['cookie'];

    if (cookie) {
        $persistentStore.write(cookie, KEY_COOKIE);
        $persistentStore.write(headers, KEY_HEADERS);
        $notification.post("淘宝金币", "凭据获取成功", "现在可以执行定时任务了");
    }
    $done({});
} else {
    // --- 执行模块 ---
    const savedCookie = $persistentStore.read(KEY_COOKIE);
    const savedHeaders = $persistentStore.read(KEY_HEADERS);

    if (!savedCookie || !savedHeaders) {
        $notification.post("淘宝金币", "运行失败", "请先手动进入一次金币页面获取凭据");
        $done();
    }

    const request = {
        url: 'https://guide-acs.m.taobao.com/gw/mtop.taobao.ms.coins.entry/1.0/',
        headers: JSON.parse(savedHeaders), // 复用抓到的全部请求头
        body: '' 
    };

    // 强行更新请求头中的 Cookie，确保是最新的
    request.headers['Cookie'] = savedCookie;

    $httpClient.post(request, (error, response, data) => {
        if (error) {
            $notification.post("淘宝金币", "网络错误", error);
        } else {
            try {
                const obj = JSON.parse(data);
                // 淘宝接口通常返回 ["SUCCESS::调用成功"]
                if (obj.ret[0].includes("SUCCESS")) {
                    $notification.post("淘宝金币", "领取成功", "今日金币已到账");
                } else {
                    $notification.post("淘宝金币", "领取失败", obj.ret[0]);
                }
            } catch (e) {
                $notification.post("淘宝金币", "解析失败", "返回内容异常");
            }
        }
        $done();
    });
}
