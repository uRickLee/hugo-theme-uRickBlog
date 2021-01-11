---
 title: "Theme Guide"
 description: "uRickBlog is a template based  Hugo,This is builted with Fluse Blog (https://github.com/kelvinconrad/fuse-Blog) Style."
 date: "2020-11-20"
 authors: 
  - uRick
 feature: true
 image: ""
 categories:
  - Theme
 tags:
  - Config
  - uRickBlog Theme
---



## Theme Style

`assets/sass`

```
//Default transition
$transition : 0.2s ease-in-out;
//Default colors
$colors     : (
    normalGray: #f5f5f5,
    title: #141414,
    subTitle: #595959,
    white: #FFFFFF,
    border: #E1E1E1,
    card: #FFFFFF,
    footer: #818394,
    btnBg: rgba(67, 112, 245, .1),
    btnBgHover: rgba(67, 112, 245, .2),
    highlight: #1890FF,//theme highlight color
    iconfont: #1890FF,//icon color
    
    // markdown base style
    blockquoteBg: #f8f8f8,
    blockquoteColor:#666,
    highlightStyle:  #1890FF, // some highlight  #42b983
    fontColor:#34495e,
    code:#e96900,
    thColor:#f2f2f2,
    trColor:#fafafa,
    preBg:#282a36,
    preColor:#f8f8f2
    );

//Cross-browser
$prefixes    : (webkit, moz, ms, o);
//Rembaseline
$rem-baseline: 16;
//Breakpoint
$breakpoint    : (sm: "576px",
    md: "768px",
    lg: "992px",
    xl: "1200px",
    xxl: "1400px",
    xxxl: "1770px", );
```

## Hugo Config

```
baseurl = "http://example.org"
defaultContentLanguage = "zh"
defaultContentLanguageInSubdir = true
# disableLanguages = ["en"] 
enableEmoji = true
enableGitInfo = false
googleAnalytics = ""
hasCJKLanguage = true
ignoreFiles = [""] 
languageCode = "zh"
paginate = 10
rssLimit = 100
summaryLength = 70
theme = "hugo-theme-uRickBlog"
timeout = 10000
title = "uRick"

[permalinks]
page = "/:slug/"
post = "/blog/:year/:month/:filename/"

[markup]
[markup.goldmark]
[markup.goldmark.renderer]
hardWraps = true
unsafe = true
xHTML = true
[markup.highlight]
# anchorLineNos = false
codeFences = true
guessSyntax = true
noClasses = false
[markup.tableOfContents]
endLevel = 4
ordered = true
startLevel = 0

[outputs]
home = ["HTML", "RSS", "json"]
page = ["HTML"]
section = ["HTML"]
taxonomy = ["HTML"]
taxonomyTerm = ["HTML"]
```

## Param Config

```
author = "uRick" # default first Author
description = "Hello,I am uRick😄."
logo="https://xxxx/blog-favicon.png" 
#logo="/images/logo.png" # images/avatar.png
imageField = "image" 
mainSections = ["post"] 
loading="/images/loading.svg"
enableTableContents=true 
googleTagManager = "" # GTM-XXXXXX
baiduAnalytics = ""
disableGoogleAdsense=true

[dateFormat]
lastUpdated = "2006-01-02"
published = "2006-01-02"
[sidebar]
featureLimit = 5  
featureFeild = 'feature' 
categories = 8 
tags = 16

[search]
    enable = '' # only algolia
    appId=''
    searchKey=''

# social start
[social]
[social.github]
icon = "iconfont icon-github"
present="hugo-theme-uRickBlog."
url = "https://github.com/uRickLee/hugo-theme-uRickBlog"
[social.gitee]
icon = "iconfont icon-gitee"
url = "https://gitee.com/uRick/hugo-theme-uRickBlog"
present="hugo-theme-uRickBlog."
# social end

# footer start
[footer]
copyright = "@2019-2020, uRick,"
licence=true #CC By 4.0
enableShare=false 
[footer.about]
summary = "Hello I am uRick.😄"
coordinate = "Chengdu · China" 
email = "urickwork@gmail.com" 
phone = "+86 1234 5486 1125"

[footer.links]
[footer.links.cocolian]
icon = "/images/vnote.png"
name = "Vnotex"
present="A note-taking application that knows programmers and Markdown"
url = "https://vnotex.github.io/vnote/en_us/"
[footer.links.coolshell]
present="The world’s fastest framework for building websites"
name = "Hugo"
icon = "/images/hugo-32x32.png"
url = "https://gohugo.io/"
[footer.sub]
# footer end

#  comments start
[comments]
enable = false 
type = "gitalk" # Available: disqus, utterances,gitalk   
[comments.utterances] # https://utteranc.es/
issueTerm = "" 
label = "" # Optional
repo = "" 
theme = "" 
[comments.gitalk]
clientId = "" 
clientSecret = ""
repo = "" 
owner = "" 
admin = "" 
# comments end 
```

## Categories Color
![Set Cat Color](/images/catColor.png)

## Summary
Hugo Config Guide: [https://gohugo.io/getting-started/configuration/](https://gohugo.io/getting-started/configuration/)

Algolia Config Guide: [https://www.algolia.com/](https://www.algolia.com/)

Gitalk Config Guide: [https://github.com/gitalk/gitalk](https://github.com/gitalk/gitalk)

Utteranc Configt Guide: [https://utteranc.es/](https://utteranc.es/)

