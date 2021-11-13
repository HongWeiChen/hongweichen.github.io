# iOS 14.x SDWebImage BUG

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
