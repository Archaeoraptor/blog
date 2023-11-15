---
title: 原神，启动！
date: 2023-11-15 19:45:08
abbrlink: 'genshin-start'
hide: true
tags:
- 原神
- Genshin
- wine
---
Linux下终于不需要禁用反作弊才能用Wine跑原神了
<!-- more -->

万恶的反作弊会直接将Wine运行的游戏判定为开挂，所以在Linux想用Wine跑需要一些魔法关闭反作弊。否则使用Wine或者Proton、CrossOver启不动。（尤其是国服）现在原神在Wine下启动已经不需要禁用反作弊了。

国际服，直接：

```bash
Wine GenshinImpact.exe
```

国服:

```bash
wine YuanShen.exe
```

然后会提示“您的机器环境异常，请重启机器”，这是因为您没有大喊“原神，启动！”。系统检测到您的逆天程度不足99.9%。[^1]

此时，您需要字正腔圆的大吼一声：“原神，启动”，~~扑向~~，点击确认并重新登录。

[^1]: <https://github.com/USTC-Hackergame/hackergame2023-writeups/blob/master/official/Hackergame%20启动/README.md>