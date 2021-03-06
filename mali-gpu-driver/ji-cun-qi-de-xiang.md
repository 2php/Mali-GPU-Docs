# 寄存器的遐想

这一篇是写于香港赤腊角机场，作为一个快销技术文章写手，在机场赶文章也不是一个什么大事儿。我可能是独学而无友，故孤陋而寡闻，所以我认定我是首创带聊闲天的快销技术文章写手。说实话我也没想到自己能坚持下来这本书，实在是能力一般，水平有限，承蒙各位读者抬爱，给了我继续更下去的动力。年根将近，在这里给大家拜个早年，江山父老能容我，不使人间造孽钱。

其实小时候更加盼望着过年，现在分析其中一大原因便是过年便意味着寒假，而寒假小伙伴们可以有更多的时间玩耍，甚至还能玩时令玩具：鞭炮 （我选用中文写这本书最主要的原因就是我是native speaker，就算自己造新词你们也没法说我错，lol）。可能我too young而且记事儿太晚，从我的记忆里，吃香的喝辣的就不是过年的专属了，毕竟随着生产力的发展，物质已经过剩了。而且对于现在来讲，过年反倒是知己相离的时候，而且由于我平日回家的频率也高，过年回家已经仅仅作为一个仪式了。不过也好，远离工作能让我有更多的时间聊闲白，这样就不用给干货了。

我决定趁着过年期间，就算我全分析错了你们也不会责备我，我决定今天更一些我对于atom完成过程的猜想。

首先，我们得猜与之相关的寄存器分别代表什么意思，这里一共有六个寄存器，分别是JOB\_IRQ\_RAWSTAT， JOB\_IRQ\_CLEAR，JOB\_IRQ\_MASK，JOB\_IRQ\_STATUS，JOB\_IRQ\_JS\_STATE，JOB\_IRQ\_THROTTLE。

经过我 胆大包天 胆大心细的分析，第一个JOB\_IRQ\_RAWSTAT就是真正的代表所有状态的寄存器，而JOB\_IRQ\_STATUS就是经过JOB\_IRQ\_MASK之后的状态寄存器。由于driver可能不想被某些中断打断，所以会mask掉一些中断，但是根据源码分析，mask仅仅是阻止了CPU被这个中断而打断，而不是完全忽略他们。如果存在一个能中断CPU的中断，那么处理完这个中断中priority最高的之后，driver会去读区RAWSTAT这个寄存器，这意味着那些被mask掉的中断也会被处理。无论是JOB\_IRQ\_STATUS还是JOB\_IRQ\_RAWSTAT，他们的高16位都代表着相应job slot由于failure而产生的中断，而低16位都代表着由于完成而产生的中断。

对于JOB\_IRQ\_CLEAR，这应该是个write-only的寄存器，如果给CLEAR的某个bit写上1，那么对应的RAWSTAT就会把相应的bit变成0。但我猜测JOB\_IRQ\_STATUS也会相应的变化。

这里最坑爹的就是JOB\_IRQ\_JS\_STATE，这个寄存器一共有32位，分为上半部分和下半部分，上半部分对应着\_next的相应slot有没有active的job，而下半部分对应着current的相应地slot有没有active的job。也就是说，如果当前中断是因为某个job完成而产生的，那么这个job的slot相应的JOB\_IRQ\_JS\_STATE的低位应该是0，除非\_next上的job已经被push到了current上。即使是这样，那么高位上相应的job slot active状态也应该是0，这一点在之后对于判断竞争状态至关重要。

最后一个寄存器就没什么鸟用了，它代表了我们要delay多少个cycle去传达一个中断，并且JOB\_IRQ\_STATUS不会被它影响，只有中断的传递会被影响。

以上。

