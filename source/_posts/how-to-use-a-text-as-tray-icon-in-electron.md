title: 如何在 electron 中使用文本作为 Tray 的 Icon
date: 2016-6-25 17:39:40
tags: electron tray text
---

我们的目标是在 macOS 的任务栏上显示当时的日期和时间。

首先，我们来看一下 `electron` 的 `Tray` [API](http://electron.atom.io/docs/api/tray/):

```js

const {app, Menu, Tray} = require('electron')

let tray = null
app.on('ready', () => {
  tray = new Tray('/path/to/my/icon')
  const contextMenu = Menu.buildFromTemplate([
    {label: 'Item1', type: 'radio'},
    {label: 'Item2', type: 'radio'},
    {label: 'Item3', type: 'radio', checked: true},
    {label: 'Item4', type: 'radio'}
  ]);
  tray.setToolTip('This is my application.')
  tray.setContextMenu(contextMenu)
})

```

然后，我们就发现 `Tray` 并不能使用一个文本作，只能使用图片。这是因为只有 macOS 才上才能把文本显示到任务栏上，例如系统时间。而 Windows 上是不行的。而我们的需要正是在 macOS 上使用一个文本来显示日期。那么，我们怎么来实现呢？

基本思路是使用 `canvas` 把文本渲染成图片！具体的步骤是这样的：

1. 在 web page 中创建一个 canvas，在 canvas 中把日期渲染出来: fillText
2. 把 `Canvas` 的内容渲染到 `dataURL`: canvas.toDataURL
3. 把 `dataURL` 从 `ipcRenderer` 发送到 `ipcMain`；这是因为 Tray 对象是在 main 进程中的，web page 中没办法拿到这个实例。
4. 把 `dataURL` 渲染成 `Image`: electron.nativeImage.createFromDataURL
5. 设置 Tray 的 icon 成新的 image: tray.setImage(image);
6. tada，大功告成！
