在Android 28中，通过使用onKeyDown()拦截KeyEvent.KEYCODE_BACK事件返回上一页，触发以下两个警告：
1. W/ViewRootImpl[DepartmentStatisticsActivity]: Cancelling event due to no window focus: KeyEvent { action=ACTION_UP, keyCode=KEYCODE_BACK, scanCode=0, metaState=0, flags=0x68, repeatCount=0, eventTime=106018768, downTime=106018719, deviceId=-1, source=0x101 }
1. W/InputEventReceiver: Attempted to finish an input event but the input event receiver has already been disposed.

通过debug跟踪，发现当点击返回键时，每一次执行完ACTION_DOWN事件，抬起时会执行ACTION_UP，但是在这个时候原Activity已经退出，没有了窗口焦点，因此会报第一个警告。 第二个警告的原因是在执行input event时，receiver已经被释放，无法完成输入事件。

解决方法：
1. 弃用onKeyDown()监听返回键，使用onKeyUp()或者onBackPress(),其中onBackPress()方法在Android2.0以后都是在onKeyUp()方法中调用，在执行onKeyUp()方法之前，这时window仍有focus，而且receiver也没有被释放。
1. 使用dispathKeyEvent()方法
```java
        @Override
        public boolean dispatchKeyEvent(KeyEvent e) {
            //判断当前按键是否返回键以及行为是否ACTION_UP。如果是，则拦截并进行处理，return true则表示事件不会继续向下派发。
            if (e.getKeyCode() == KeyEvent.KEYCODE_BACK && e.getAction() == KeyEvent.ACTION_UP) {
                SubAppManager.getInstance().finishActivity();
                return true;
            }
            return super.dispatchKeyEvent(e);
        }
```
