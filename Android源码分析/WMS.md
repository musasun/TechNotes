# WMS窗口管理服务源码分析

## Surface的DrawState状态变化

### DrawState的5种状态

``` Java
/** This is set when there is no Surface */
// surface未创建
static final int NO_SURFACE = 0;
/** This is set after the Surface has been created but before the window has been drawn. During
    * this time the surface is hidden. */
// surface已创建，但窗口还没有绘制。这个阶段surface被隐藏。
static final int DRAW_PENDING = 1;
/** This is set after the window has finished drawing for the first time but before its surface
    * is shown.  The surface will be displayed when the next layout is run. */
// 完成窗口首次绘制，但它的surface还没有显示。surface将在下一次layout执行时显示。
static final int COMMIT_DRAW_PENDING = 2;
/** This is set during the time after the window's drawing has been committed, and before its
    * surface is actually shown.  It is used to delay showing the surface until all windows in a
    * token are ready to be shown. */
// 窗口绘制已提交，但还没有真正显示出来。通常是延迟等待同一个token的所有窗口都准备好显示。
static final int READY_TO_SHOW = 3;
/** Set when the window has been shown in the screen the first time. */
// 窗口首次显示到屏幕上
static final int HAS_DRAWN = 4;
```

### NO_SURFACE

#### createSurfaceLocked

窗口首次relayout时，先创建surface
```
WindowManagerService.relayoutWindow
WindowManagerService.createSurfaceControl
WindowStateAnimator.createSurfaceLocked
```

#### destroySurfaceLocked

要relayout可见窗口的话，要将老surface销毁
```
WindowManagerService.relayoutWindow
WindowState.relayoutVisibleWindow
WindowStateAnimator.preserveSurfaceLocked
WindowStateAnimator.destroySurfaceLocked
```

### DRAW_PENDING

```
DisplayPowerController.setScreenState
PhoneWindowManager.screenTurningOn
```

```
PhoneWindowManager$PolicyHandler.handleMessage
    case MSG_KEYGUARD_DRAWN_COMPLETE:
        finishKeyguardDrawn();

    case MSG_KEYGUARD_DRAWN_TIMEOUT:
        finishKeyguardDrawn();
```

```
PhoneWindowManager.finishKeyguardDrawn
WindowManagerInternal.waitForAllWindowsDrawn
DisplayContent.waitForAllWindowsDrawn

WindowState.updateResizingWindowIfNeeded

WindowStateAnimator.resetDrawState
```

### COMMIT_DRAW_PENDING

当应用完成窗口绘制后，通过binder接口通知系统。
```
ViewRootImpl.pendingDrawFinished
ViewRootImpl.reportDrawFinished or ViewRootImpl.doDie
Session.finishDrawing
WindowManagerService.finishDrawingWindow
WindowStateAnimator.finishDrawingLocked
```

### READY_TO_SHOW

```
DisplayContent$mApplySurfaceChangesTransaction
WindowStateAnimator.commitFinishDrawingLocked
```

### HAS_DRAWN

```
AppWindowToken.showAllWindowsLocked

WindowState.commitFinishDrawingLocked

DisplayContent$mUpdateWindowsForAnimator
```

```
WindowState.performShowLocked
```
