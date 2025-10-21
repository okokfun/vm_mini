# 6位数字手持式电压-电阻-表
这是 JVO-2 的存储库 此自述文件仍然不完整。在此处查找更多信息
https://www.eevblog.com/forum/metrology/diy-6-digit-handheld-volohmmeter/
https://imgur.com/a/50MBxly


# 下文是上面链接的翻译
## 前言 
&ensp;&ensp; 我不确定这个主题是否应该属于“项目”部分，但 6 位电压表是边缘电压表，也许在 DIY 项目时更多，所以我在这里输入了它。当然，Mod 有权将其转移到其他地方。

&ensp;&ensp; 这是一个更少完成的项目。我将接受更多的吐槽和润色，但计划中没有进行重大返工。我决定让任何感兴趣的人都可以使用这个项目
## 动机
It all started with my previous project [1]. It turned out to be quite chunky box, with a fair bit of circuitry for what it is doing - measuring single voltage input. Nonetheless, despite being one of a few finished long-scale voltmeter project, it served as testbed for further development and experiments. The enclosure is really closely packed with circuitry and I wondered if it can get any smaller. On a first glance it looked quite hard, but shortly I realised it can be done, with some added benefits. Except of the obvious form factor shrink, more efficient power supply could bring less self heating, with associated faster power-up stabilization. While shrink-down voltmeter could be worthy project on its own, I decided to make it harder by adding two features - switchable input ranges and resistance measurement.

While tabletop multimeters with 6 and more digits are quite common from multiple of manufacturers, vast majority of handheld multimeters do have resolution up to 60000 counts (4 and 3/4 digits), with some going to 500000 counts (5 and 3/4 digits), while 6 digits multimeters being rare as hen's teeth (Gossen Metrawatt METRAHIT 30M being exception). This imbalance exists for a reason - different use case for handheld multimeters is one factor, harder environmental conditions making stable references difficult to build is definitely another one, not to mention somehow higher consumption of precision circuitry. My device isn't here to change this in any way. I'm aware of limitations of my device and I approached it as practical example of impractical design, since chase is better than the catch.

## 实施、设计目标
I set a few goals in previous paragraph - 6 digit measurement (with reasonable linearity and noise), battery operation, switchable input ranges. This is one boundary condition, still leaving a lot of degrees of freedom. In order to move forward I needed to define mechanical factors. Despite having 3D printer in my home lab, I opted for off-the-shelf plastic enclosure, after a bit of searching I settled down on a quite cheap enclosure [2] with 6xAA battery holder and dimensions still acceptable to be called handheld. 

Considering low cost, acceptable quality and good availability, another boundary condition was set. Designing handheld devices usually involves pingponging between mechanical and electrical design, this one was not an exception, so I had to return to electronics again.

I set up those design goals:
- Three voltage ranges: 1V, 10V and 100V full range, bipolar. 1V and 10V ranges should have switchable input resistance - 10M and >1GΩ. (in fact I was able to acheive 1M, 10M and >1GΩ)
- Selectable integration time, at least 1, 10 and 100PLC
- Five resistance ranges - 1k, 10k, 100k, 1M and 10M full range
- Four wire resistance measurement

I set up rough block diagram, see attachment 01 below. For ADC core I opted for design I already had [1], with some adjustments. EPM240 is CPLD that can host and ADC controller, but not much else. Since I wanted employ integration time switching as well as experimenting with other multislope modulation schemes, I decided to leave EPM240 for something more powerful (here it means jumping from 240 logic blocks to 1k and more). Lattice seems to have portfolio that fits my needs - hand-solderable package (QFN being tolerable, but QFP preferred), relatively low power operation and 1k+ LUTs. Mach XO2 fit the needs quite well, so the FPGA selection was done. In my previous project I used MSP430 to spice up my work, in this project I decided to use more familiar STM32L151. On user end of device is thin COG LCD with backlight, EA DOG132, since it had suitable dimensions and current consumption. For buttons I opted for Marquardt 6450 Series again, I just love those pushbuttons.

我以前的电压表部分是以“把厨房水槽扔到问题上”的方式设计的。我试图让我的脚更靠近地面。昂贵的 LT1116 比较器被 LM311 取代，同时仍然适用于此应用。积分器中的运算放大器是OPA140，电路的其余部分采用OPA192或OPA2192;OPA177A用于 LM399A 周围的参考电路。在参考源中使用 LM399A 是一个艰难的选择。对于电池供电的应用，像 LT1236 或 LTC6655 这样的东西会是我更明智的选择，但我决定在这里花哨一点，并使用加热参考，主要是因为它是设计挑战。

要实现 +-10V 输入范围（实际上是 +-12V 的超量程）和高输入阻抗（GOhm 或更高）并不是一件容易的事。在我以前的电压表中，我对LTC2057寄予厚望，但它的显着电流噪声在较高的输入阻抗下给我带来了很多麻烦。还有其他自动归零运算放大器，例如参数更好的LTC1052，但电源范围低迫使我为它制作自举电源。这开始决定电源要求。为了在自举输出上实现+-12V输出摆幅，需要+-18V范围。36V 是 OPA192 和其他运算放大器的最大电源电压，因此我决定派生另一个电源轨 -14V 来为运算放大器供电。这将电源电压设置为 32V，对于电路的模拟部分和 ADC 仍然很好。由于两个 +-18V 电源轨的预计电流消耗约为 20mA，这需要模拟部件大约 1W 的输出功率。经过一番搜索，我选择了ADP5070，然后选择了基本的、更少的数据表配置中的ADP7142和ADP7182。FPGA 的电源通过 3,3V LDO 在关断的情况下提供，MCU 始终通过低功耗 MCP1703 LDO 通电。最后一个电源轨为 5V，用于为继电器和显示器背光供电。当器件关断时，MCU处于休眠状态，电流消耗最小，但可以通过按下按钮唤醒并启动电路的FPGA和模拟部分;以及在需要时关闭所有电源。电池电压通过高电阻分压器 （6,2/2,4M） 监控，然后使用电压跟随器监控MCP6441为 MCU 内部的 ADC 获得良好的低阻抗输出。板载隔离式USB接口，围绕CP2102接口IC和ADUM1202隔离器制成。

我希望至少将数字域与模拟电隔离，但不幸的是我无法使电源足够小且足够好，所以我选择了上述电源方案。这也迫使我在接地、去耦和电源平面放置方面更加小心。回到机械设计，我意识到我无法将所有东西都安装在单个 PCB 上，尺寸由外壳尺寸给出，所以我选择了两个 PCB 的夹层结构。一些电路部件是固定的，例如用户界面（显示器、按键）必须位于上部（主）PCB上。最合乎逻辑的选择是在两个 PCB 上分离数字和模拟部件，但我无法在单个 PCB 上同时安装模拟前端和 ADC，因此我决定在主 PCB 上安装 ADC，而将模拟前端留在较小的扩展 PCB 上。

外壳尺寸和电路块划分给了我另一个限制，所以我终于可以进行原理图绘制和 PCB 设计了。

由于我很清楚该做什么，所以这是相对容易的部分，大部分工作都在两个晚上完成。我选择了一些有趣和快乐(业余爱好项目是为了有趣和快乐而做的，对吗？)，并在两块电路板上使用了Minimelf 0207电阻，甚至在一些电容上也是如此。

不幸的是，去耦电容采用了更“无聊”的0805尺寸格式。如果有人试图复制我的设计，最小基底面可以用普通且易于焊接的1206尺寸电阻填充，这样你就不必获得不完全普通的最小尺寸电阻。
## 让它发挥作用
当PCB被大量填充后，我发现了一些令人讨厌的布局错误，迫使我进行了相当多的返工和调整。较大的主PCB基本上还可以，但模拟板即使在过度调整后也只能发挥一半作用。电压表部分没问题。 但欧姆电流源少了两个量程，4线测量显然无法正常工作。我决定重新使用电路板来修复缺陷，之后我有了更少功能的设备，可以进行4线电阻测量，并且可以工作到10兆欧。更多图片和评论见相册[3]。

我意识到我也可以“免费”进行二极管测试，因此也在这里扩展了功能。

## 验证
由于ADC的输入范围为+-10V（cca 2V超量程），我首先关注该特定范围内的JVO-2测试。
- 我使用Time Electroncis 2003S校准器和以线性度良好著称的HP-34401A测量了6,5位数DMM的INL。非线性度均未超过1ppm，见下文附件02。无论如何，这仍然有点问题，为了适当覆盖INL，我需要更高级别的仪器。至少我知道我的设备没有完全关闭。
- 我对短路的输入插孔进行了相当多的测量，根据 [4] 测试噪声，这是很棒的资源。幸运的是，确实有很多已经测量的商业设备，所以我有一些东西可以对抗。在 10V 范围内，对于 1PLC、10PLC 和 100PLC，我测得的 RMS 噪声为 0.69、0.21 和 0.16ppm。有关其他范围，请参阅下面的附件 03。为了进行比较，我设置了表格捕获其他一些台式万用表和我的设备的测量噪声数据，请参阅下面的附件 04 和 05。
- 由于这是电池供电的仪器，我对启动行为以及电池耗尽时的稳定性感到好奇。上电后，跳高约 15ppm，在大约 10 分钟内下降到一两 ppm 以内，典型的启动行为如下图 06 所示。

当电源电压从 10V 降低到 5.8V 时，我无法检测到任何超过 1ppm 的读数变化。要实现这一点并不像听起来那么容易，在我的第一次试验中，我发现 ADC 读数对电池电压的依赖性相当强（且非线性），尽管 +-18V 电源轨非常稳定。经过一番思考，我找到了主开关电源传导 EMI 影响 LM399A 参考的原因。在LM399引脚上用100nF电容器正确去耦基准电压源解决了这个问题。
- 欧姆范围没有得到电压范围那么多的处理。根据我的HP34401调整了设备，并用我手头的一些稳定电阻器进行了检查。从初步检查来看，读数似乎在几十 ppm 内相互对应，但我还不称其为适当的测试。

## 概述
现在，我在手持式外壳中得到了某种奇怪的长尺度电压欧表组合，电池由饥饿的 LM399A 参考供电。

与这个项目之前的知识相比，我学到了一些新东西。
- 低局部加热(以及一般的热设计)是精密电路的重要因素。
- 有工作的ADC(作为PCB上的电路)与有电压表(作为盒内器件)相去甚远，这与多量程万用表相比还有很长的路要走。
- 不管MELF代表什么(大部分都是躺在地板上)，我并没有失去一个MELF电阻。
- LM399真的不适合电池工作。

毕竟，这是一个非常有趣的项目，我并不后悔花在上面的时间和金钱。所有来源都可以在 github [5] 上找到。[3] 中的链接包含很多照片，并附有一些评论。

## 今后的工作
- 我应该验证欧姆量程
- 自动范围仍未实现。不过，我什至不确定我是否要实现它。
- 随着 MCU 源代码的增长，我意识到我选择了糟糕的固件结构。

重写它以省略重复的代码块是个好主意，但工作量相当大。
- 有两个输入（主输入和 4W 电阻检测）使我能够进行比率测量。这是稍后要研究的事情。
- 搭载更大的FPGA使我能够尝试其他比现在更多的调制方案。这很可能是我将要尝试的一件事。

## 链接
[1] - https://www.eevblog.com/forum/metrology/diy-6-5-digit-voltmeter/

[2] - https://www.tme.eu/sk/katalog/?search=KM-103&s_field=1000011&s_order=desc

[3] - https://imgur.com/a/50MBxly

[4] - https://xdevs.com/article/dmm_noise/

[5] - https://github.com/jaromir-sukuba/vm_mini