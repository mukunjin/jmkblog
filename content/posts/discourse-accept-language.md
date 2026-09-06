---
title: "Accept-Language如何影响Discourse的语言显示"
date: 2026-08-27T19:59:23+08:00
categories: 计算机
description: "Discourse 如何根据 URL 参数、Cookie 和 Accept-Language 请求头决定页面语言？本文通过实际经历和源码分析，还原这一语言解析流程。"
summary: "Discourse 如何根据 URL 参数、Cookie 和 Accept-Language 请求头决定页面语言？本文通过实际经历和源码分析，还原这一语言解析流程。"
---
## 前言
我从25年末开始逛各种网络论坛和博客，有时还会在学校看。我平时只能用老年机，加上网络的原因我只能看NL，没法看L站、V站和NS。但是我的老年机上的浏览器完全无法执行JS，连CSS都加载不全，只能在NL看文字。

原本NL上几乎所有帖子都是中文，但是发现在我的老年机上大部分都变成了英文，明显看得出来是被翻译过的。很奇怪的是，部分内容还是保留着中文。这个问题困扰了我很久，一直不知道为什么会被翻译。

## 初见端倪
偶然间，我在正常的浏览器环境中手动把论坛语言选择为英文，发现中文帖子竟然被翻译成了英文，我原本以为他只会把论坛中的UI字符串转换成英文。

于是我在老年机上打开相同的内容对比，发现的帖子翻译情况（有的帖子变英文、有的帖子变中文）和正常的浏览器环境上看到的完全一样（见下图）。这也就说明，一定是我老年机上面的浏览器环境触发了Discourse的语言切换。

![discourse-post-translation-comparison.jpg](https://img.mukunjin.com/2026/discourse-post-translation-comparison.jpg)

## 头绪
论坛界面文字变成了英文而帖子内容没有全译，而我的老年机又跑不了JavaScript，说明语言切换只能是服务器端在返回页面前就决定的，服务器能提前获知的用户语言偏好来源有URL参数、Cookie和Accept-Language请求头。在我这个场景下，URL中没有语言参数，老年机浏览器也没有保存过语言Cookie，所以最可能的就是Accept-Language请求头。

为了验证我的Accept-Language请求头，我想直接在老年机上访问`https://httpbin.org/headers`，但是莫名其妙给我返回了一个.err文件，并且页面无法显示任何内容。

经过思考后，我决定部署一个Cloudflare Workers脚本。Workers运行在服务器端，可以直接读取请求头并生成纯HTML页面返回给浏览器；客户端浏览器收到的只是纯HTML内容，完全不涉及任何JavaScript的执行，因此即使我的老年机用不了JS，页面也能正常显示所有信息。

```javascript
export default {
  async fetch(request) {
    const acceptLanguage = request.headers.get('Accept-Language') || '未获取到';
    
    const html = `<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>请求头信息</title>
</head>
<body>
Accept-Language: ${acceptLanguage}
</body>
</html>`;
    
    return new Response(html, {
      headers: {
        'Content-Type': 'text/html; charset=utf-8',
      },
    });
  },
};
```

通过我的老年机访问，最终获得了如下图的结果。

![accept-language-query.webp](https://img.mukunjin.com/2026/accept-language-query.webp)

可以发现，Accept-Language的确是en。但这还不够，我需要去官方仓库里查阅源码才能确定处理Accept-Language的逻辑。

## 探索
经过一番苦苦搜寻后，终于在[官方仓库](https://github.com/discourse/discourse)中找到了答案。具体逻辑一共分成四步。
>下面部分描述可能因为新的提交而过时。本文写作时仓库的最新提交为https://github.com/discourse/discourse/commit/ed9f07d9f5cebcd9160c149cd179de98256e5dea

### 第一步：请求入口 `with_resolved_locale`

**文件**：`app/controllers/application_controller.rb`。

第42行注册为 `around_action`：
```ruby
around_action :with_resolved_locale
```
意味着每个请求前后都会执行它，它就是整个语言解析链的入口。

第424-445行是方法本体：

```ruby
def with_resolved_locale(check_current_user: true)
  if check_current_user &&
       (
         user =
           begin
             current_user
           rescue StandardError
             nil
           end
       )
    locale = user.effective_locale          # 第434行：已登录 → 用用户偏好
  else
    locale = Discourse.anonymous_locale(request)  # 第436行：未登录 → 关键调用
    locale ||= SiteSetting.default_locale         # 第437行：兜底默认语言
    persist_locale_param_to_cookie                # 第438行：把URL的locale参数存cookie
  end

  locale = SiteSettings::DefaultsProvider::DEFAULT_LOCALE if !I18n.locale_available?(locale)  # 第441行

  I18n.ensure_all_loaded!                        # 第443行
  I18n.with_locale(locale) { yield }             # 第444行：设置本次请求的语言
end
```
未登录用户走的是第 436 行`Discourse.anonymous_locale(request)`

### 第二步：`Discourse.anonymous_locale`

这个方法在`app/controllers/application_controller.rb`被调用，定义在 `lib/discourse.rb` 中的第1360-1366行。
```ruby
def self.anonymous_locale(request)
    locale = request.params[LOCALE_PARAM] if SiteSetting.set_locale_from_param
    locale ||= locale_from_cookie(request)
    locale ||=
      request.env["HTTP_ACCEPT_LANGUAGE"] if SiteSetting.set_locale_from_accept_language_header
    HttpLanguageParser.parse(locale)
  end
```

按优先级依次尝试从三个来源获取匿名用户的语言：
>实际上每个来源都有独立的SiteSetting开关，但NL应该是全部启用了。

1.URL参数：?tl=xx

2.Cookie

3.浏览器Accept-Language请求头

然后丢给`HttpLanguageParser.parse`做匹配解析。

### 第三步：Accept-Language头的实际解析

文件：`lib/http_language_parser.rb`。完整代码只有 14 行。

```ruby
# frozen_string_literal: true

module HttpLanguageParser
  def self.parse(header)
    # Rails I18n uses underscores between the locale and the region; the request
    # headers use hyphens.
    require "http_accept_language" unless defined?(HttpAcceptLanguage)
    available_locales = I18n.available_locales.map { |locale| locale.to_s.tr("_", "-") }
    parser = HttpAcceptLanguage::Parser.new(header&.tr("_", "-"))
    matched = parser.language_region_compatible_from(available_locales)&.tr("-", "_")
    matched || SiteSetting.default_locale
  end
end
```

这段代码负责把论坛所有可用语言从Rails格式（zh_CN）转成HTTP头格式（zh-CN），再将传入的Accept-Language头中可能存在的下划线统一替换为连字符，以符合 HTTP 语言标签的标准格式丢给http_accept_language这个gem的Parser去按浏览器优先级在可用语言里找最佳匹配，匹配到后再把连字符转回下划线（zh-CN →zh_CN）以符合Rails I18n的格式要求，找不到就返回站点默认语言。

### 第四步：匹配结果传回`with_resolved_locale`
最终，HttpLanguageParser.parse返回匹配到的语言"en"，控制回到`with_resolved_locale`，经过第441行的locale可用性校验后执行第444行：
```ruby
I18n.with_locale(locale) { yield }
```
I18n.with_locale是Rails i18n框架的方法，它把当前线程的locale 设为 "en"，然后执行 yield（也就是后续的页面渲染）。渲染过程中所有调用 I18n.t 翻译 UI 文字的地方，都会去读英文的翻译文件。页面返回给浏览器，就是英文的了。

## 真相大白
至此，困扰我几个月的问题终于有了答案。整个语言解析流程可以拆成四个环节：

1.请求进入`with_resolved_locale`，判断用户是否登录

2.未登录时调用Discourse.anonymous_locale，依次从URL 参数、Cookie、Accept-Language头中获取语言

3.HttpLanguageParser.parse将获取到的语言与论坛支持的语言列表做匹配

4.匹配结果传回`with_resolved_locale`，经过第 441 行的校验后通过I18n.with_locale应用语言并渲染页面。

~~至于为什么语言选择为英语后部分帖子仍为中文就不知道了。~~

一个看似诡异的问题，追溯到底不过是HTTP协议中一个再普通不过的请求头。
