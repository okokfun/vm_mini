# 6位数字手持式电压-电阻-表
这是 JVO-2 的存储库 此自述文件仍然不完整。在此处查找更多信息
https://www.eevblog.com/forum/metrology/diy-6-digit-handheld-volohmmeter/
https://imgur.com/a/50MBxly


# 下文是上面链接的翻译

## Foreword
I'm not sure whether this topic should belong to 'Projects' section, but 6 digit voltmeters is borderline voltnuttery thing, perhaps more when it's DIY project, so I entered it here. Mods have all the power to move it elsewhere, of course.

This is more-less finished project. I will receive some more spit and polish, but major rework is not in the plans. I decided to make the project available for anybody intersted.

## Motivation
It all started with my previous project [1]. It turned out to be quite chunky box, with a fair bit of circuitry for what it is doing - measuring single voltage input. Nonetheless, despite being one of a few finished long-scale voltmeter project, it served as testbed for further development and experiments. The enclosure is really closely packed with circuitry and I wondered if it can get any smaller. On a first glance it looked quite hard, but shortly I realised it can be done, with some added benefits. Except of the obvious form factor shrink, more efficient power supply could bring less self heating, with associated faster power-up stabilization. While shrink-down voltmeter could be worthy project on its own, I decided to make it harder by adding two features - switchable input ranges and resistance measurement.

While tabletop multimeters with 6 and more digits are quite common from multiple of manufacturers, vast majority of handheld multimeters do have resolution up to 60000 counts (4 and 3/4 digits), with some going to 500000 counts (5 and 3/4 digits), while 6 digits multimeters being rare as hen's teeth (Gossen Metrawatt METRAHIT 30M being exception). This imbalance exists for a reason - different use case for handheld multimeters is one factor, harder environmental conditions making stable references difficult to build is definitely another one, not to mention somehow higher consumption of precision circuitry. My device isn't here to change this in any way. I'm aware of limitations of my device and I approached it as practical example of impractical design, since chase is better than the catch.

## Implementation, design goals
I set a few goals in previous paragraph - 6 digit measurement (with reasonable linearity and noise), battery operation, switchable input ranges. This is one boundary condition, still leaving a lot of degrees of freedom. In order to move forward I needed to define mechanical factors. Despite having 3D printer in my home lab, I opted for off-the-shelf plastic enclosure, after a bit of searching I settled down on a quite cheap enclosure [2] with 6xAA battery holder and dimensions still acceptable to be called handheld. 
Considering low cost, acceptable quality and good availability, another boundary condition was set. Designing handheld devices usually involves pingponging between mechanical and electrical design, this one was not an exception, so I had to return to electronics again.
I set up those design goals:
- Three voltage ranges: 1V, 10V and 100V full range, bipolar. 1V and 10V ranges should have switchable input resistance - 10M and >1Gohm. (in fact I was able to acheive 1M, 10M and >1Gohm)
- Selectable integration time, at least 1, 10 and 100PLC
- Five resistance ranges - 1k, 10k, 100k, 1M and 10M full range
- Four wire resistance measurement