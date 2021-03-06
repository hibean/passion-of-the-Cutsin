# IE interpolation(图片插值模式)管窥

有同事email提议：

产品的用户头像在IE下看起来很糙，可以利用“双三次插值”方法对图片已经处理，让它看起来柔和一些。IE7本身就有这个私有属性，我也火星一下，才了解到，不过对我们产品的细节有帮助，可以看看给需要优化的图片加个样式，改善在IE7下因缩放造成的失真。

我拿新班级首页代码试了下，挺明显的，为了测试效果直接作用于所有img标签了：
```
img {-ms-interpolation-mode:bicubic}
```
效果对比见下图

隐约记得年初时看到过相关介绍，使劲回忆了下为什么当时没有应用的原因：还是效率问题

这个interpolation（插值模式）没有默认值，只有nearest-neighbor（近邻取样/仿色）和bicubic （双三次插值）两个值，后者效率较低。

虽然说是没有默认值，但msdn上却有一句话：__If the zoom level of the page is 100%, the default interpolation is nearest-neighbor, otherwise bicubic mode is used.__

也就是说，当你缩放网页时（Ctrl+滚轮或查看->缩放），只要不是100%，图像的算法就会变成双三次插值，所以应该可以理解为IE7默认的插值算法就是临近取样；

然后，IE6不支持缩放，所以和它无关；

IE7呢？请参考附件截图（nearest-neighbor.png, bicubic.png, none.png），F5刷新页面瞬间，默认和指定临近取样时cpu 占用都在70%左右，而双三次插值则到了97%甚至100%；

而IE8默认就是双三次插值，cpu 占用始终在60%左右，这时再指定临近取样也是生效的，而且能看到cpu占用的降低；

所以从IE7默认不打开双三次插值这一点来看，也许微软觉得IE7默认执行不太靠谱，尤其是元素比较多的情况，所以让我们自己权衡……

于是除了附件中的截图，又做了个小试验，初衷仍然是：“好在它是个style属性，可以自行指定想在哪些元素上应用，效率应该还是可以控制的”
然而结果很无奈，cpu的占用会随着你应用双三次插值元素数量的增加而增加……截图中那个页，一半数量的成员头像开启双三次插值就让cpu飙到了94%

以上cpu占用数据都是指刷新/打开页面的瞬间，并且在虚拟机中测试。

虽然个人还是倾向于对某些需要着重显示的图片如此处理（比如相册大图、幻灯效果之类），但仍可以多做测试，看是否能正式在产品中应用以改善在IE7的显示效果。
