---
layout:     post
title:      iOS14下SDWebImage所引发的一个BUG
subtitle:   iOS14下SDWebImage所引发的一个BUG
date:       2020/12/31
author:     HongWeiChen
header-img: img/blog-banner-dark.jpg
catalog: true
tags:
    - Objective-C
---

# iOS14下SDWebImage所引发的一个BUG

https://github.com/SDWebImage/SDWebImage/pull/3092

- (void)displayLayer:(CALayer *)layer
{
    if (_currentFrame) {
        layer.contentsScale = self.animatedImageScale;
        layer.contents = (__bridge id)_currentFrame.CGImage;
    } else {
        if ([UIImageView instancesRespondToSelector:@selector(displayLayer:)]) {
            [super displayLayer:layer];
        }
    }
}
