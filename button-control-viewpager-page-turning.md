# [Android 实战技巧之七：按钮控制 ViewPager 的左右翻页](http://blog.csdn.net/lincyang/article/details/20797143)

为了实现左右翻页的效果，使用了 ViewPager，它会很方便的实现左右滑动后翻页。

这时需要自己也加上两个 button 来实现同样的操作，如何实现呢？网上一篇 blog 帮了忙啦。

[【Android】按钮控制 ViewPager 的左右翻页，保留原有的动画效果](http://blog.sina.com.cn/s/blog_4b20ae2e0101fkpz.html)

ViewPager 的一个公共方法 arrowScroll，查看代码我们可以有两个重要的发现：

```
    public boolean executeKeyEvent(KeyEvent event) {  
        boolean handled = false;  
        if (event.getAction() == KeyEvent.ACTION_DOWN) {  
            switch (event.getKeyCode()) {  
                case KeyEvent.KEYCODE_DPAD_LEFT://键盘方向键左键的控制，向左翻页  
                    handled = arrowScroll(FOCUS_LEFT);//FOCUS_LEFT:17  
                    break;  
                case KeyEvent.KEYCODE_DPAD_RIGHT://右键  
                    handled = arrowScroll(FOCUS_RIGHT);//FOCUS_RIGHT:66  
                    break;  
                case KeyEvent.KEYCODE_TAB:  
                    if (Build.VERSION.SDK_INT >= 11) {  
                        // The focus finder had a bug handling FOCUS_FORWARD and FOCUS_BACKWARD  
                        // before Android 3.0. Ignore the tab key on those devices.  
                        if (KeyEventCompat.hasNoModifiers(event)) {  
                            handled = arrowScroll(FOCUS_FORWARD);//FOCUS_FORWARD:2  
                        } else if (KeyEventCompat.hasModifiers(event, KeyEvent.META_SHIFT_ON)) {  
                        handled = arrowScroll(FOCUS_BACKWARD);// FOCUS_BACKWARD:1  
                        }  
                    }  
                    break;  
            }  
        }  
        return handled;  
    }
```

```
    public boolean arrowScroll(int direction) {  
        View currentFocused = findFocus();  
        if (currentFocused == this) currentFocused = null;  
  
        boolean handled = false;  
  
        View nextFocused = FocusFinder.getInstance().findNextFocus(this, currentFocused,  
                direction);  
        if (nextFocused != null && nextFocused != currentFocused) {  
            if (direction == View.FOCUS_LEFT) {  
                // If there is nothing to the left, or this is causing us to  
                // jump to the right, then what we really want to do is page left.  
                final int nextLeft = getChildRectInPagerCoordinates(mTempRect, nextFocused).left;  
                final int currLeft = getChildRectInPagerCoordinates(mTempRect, currentFocused).left;  
                if (currentFocused != null && nextLeft >= currLeft) {  
                    handled = pageLeft();  
                } else {  
                    handled = nextFocused.requestFocus();  
                }  
            } else if (direction == View.FOCUS_RIGHT) {  
                // If there is nothing to the right, or this is causing us to  
                // jump to the left, then what we really want to do is page right.  
                final int nextLeft = getChildRectInPagerCoordinates(mTempRect, nextFocused).left;  
                final int currLeft = getChildRectInPagerCoordinates(mTempRect, currentFocused).left;  
                if (currentFocused != null && nextLeft <= currLeft) {  
                    handled = pageRight();  
                } else {  
                    handled = nextFocused.requestFocus();  
                }  
            }  
        } else if (direction == FOCUS_LEFT || direction == FOCUS_BACKWARD) {//17 or 1  
            // Trying to move left and nothing there; try to page.  
            handled = pageLeft();  
        } else if (direction == FOCUS_RIGHT || direction == FOCUS_FORWARD) {//66 or 2  
            // Trying to move right and nothing there; try to page.  
            handled = pageRight();  
        }  
        if (handled) {  
            playSoundEffect(SoundEffectConstants.getContantForFocusDirection(direction));  
        }  
        return handled;  
    }  
```

也就是说，我们调用 arrowScroll 方法用参数1或者17就可以实现向左翻页；参数2或66就可以实现向右翻页。

后记：  

当你的 UI 中有 EditText 这种获得 focus 的 widget 时，则必须用17和66，否则要报错。