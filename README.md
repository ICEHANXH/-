# -行文本编辑器ed

##  题目要求
编写一个命令行的行文本编辑处理程序。用户按照给定的格式输入，程序需要解析用户输入的命令，并完成相应的操作。

现有类为EDLineEditor，待测方法为public static void main(String[] args)方法。args不传递参数，所有输入通过命令行进行。

## 命令介绍

对ed命令中实现起来相对可行的部分进行了提取和修改，内容如下：

ed有两个模式，命令模式和输入模式。在命令模式下输入的是命令，这些命令用来指定对编辑文本的操作；输入模式下输入的都是文本（除了退出输入模式的命令），这些文本将依照命令模式下的命令，被直接添加或替换到文本中。

可以通过ed file或者ed进入ed编辑器，进入时不需要输出或打印任何内容。两者有所区别。前者读取file文件的内容到编辑器缓存中，后续操作基于缓存中的内容。后者直接操作空的编辑器缓存。所有输入影响的对象是编辑器缓存中的内容，在命令模式下可以通过保存命令将文本保存到指定文件中。

初始默认进入命令模式。命令模式通过"a"、"i"、"c"命令（后续详细介绍）进入到输入模式，输入模式通过"."退出到命令模式。

在命令模式中，ed命令是由0、1或2个地址，后接一个单字符命令，以及参数（可能没有）组成。ed命令的结构如下：

	[地址[,地址]]命令[参数]

地址部分指定了命令起作用的一行或多行文本。如果给出的地址少于命令接受的地址，则使用默认地址。

测试时只测试完整的ed命令，不会单独测试地址。

地址部分包含如下元素：

	.		当前行——初始时，当前行默认为给定文本的最后一行。
	$		文本最后一行
	n		文本的第n行，如果没有，则输出'?'
	-n		从文本当前行数起，向前的第n行，如果没有，则输出'?'
	+n		从文本当前行数起，向后的第n行，如果没有，则输出'?'
	m,n		文本的第m到n行，n不小于m，否则输出'?'
	,		文本的所有行
	;		文本当前行到最后一行
	/str/	从文本当前行起（不包含当前行），下一个匹配str的行。如果在当前行之后没找到，搜索将从文本的开始处继续至当前行。匹配不到则输出'?'
	?str?	从文本当前行起（不包含当前行），上一个匹配str的行。如果在当前行之前没找到，搜索将从文本的末尾处继续至当前行。匹配不到则输出'?'

注1：m、n均为非负整数。'.'、'$'和'n'能够与'-n'和'+n'组合，成为更复杂的地址，实际上'-n'可表示为为'.-n'，'+n'可表示为'.+n'。'm,n'中的m和n可由其他单行地址代替。/str/和?str?能够与'-n'或'+n'组合。

注2：每行有一个行号，行号从1开始。当内容为空时，当前行的行号为0。匹配的意思是行中包含这个str字符串。

命令为单个字符，其中一些命令有参数。需要实现解析的命令如下：（小括号里为默认地址）

基本命令：

	(.)a		切换到输入模式，将新输入的文本追加到指定行的后面，当前行被设为输入文本的最后一行。
	(.)i		切换到输入模式，将新输入的文本追加到指定行的前面，当前行被设为输入文本的最后一行。
	(.,.)c		切换到输入模式，将新输入的文本替换成指定行，当前行被设为输入文本的最后一行。
	(.,.)d		删除指定行，如果被删除的文本后还有文本行，则当前行被设为该行，否则设为被删除的文本的上一行。
	(.,.)p		打印指定行的内容，当前行被设为打印行的最后一行。
	($)=		打印指定行的行号，不改变当前行。
	(.+1)z[n]	从指定行，一次向后移动n行，打印包含指定行以及其后n行的内容。当前行被设为最后被打印的行。当指定行到末尾行不足n行时，打印当前行到末尾行。当参数n指定时，必为正整数；不指定时，打印当前行到末尾行。
	q			退出ed。当有更改的内容未保存到文件时，提示一次'?'。再次输入q，则放弃内容的更改，退出ed。
	Q			退出ed。不论有没有更改未保存，都不提示直接退出ed。
	f [file]	设置默认文件名，不打印任何内容。如果没有指定参数file，则打印默认文件名（没有默认文件名时，打印'?'）。通过ed file命令进入时，file被设置为默认文件名。通过ed命令进入时，没有默认文件名。
	(1,$)w [file]	保存指定行到指定文件，当前行不变。保存后不需要输出或打印任何内容。如果指定文件存在，则保存的内容覆盖文件内容。如果文件不存在，则创建文件并保存内容。如果参数file不指定，则使用默认文件名代替参数file。当没有默认文件名时，参数file必须指定，否则打印'?'提示。当没有默认文件名时，第一个w file命令会把默认文件名设置为file。当有默认文件名时，w的file参数不会更改默认文件名。
	(1,$)W [file]	保存指定行到指定文件，当前行不变。保存后不需要输出或打印任何内容。保存形式为追加保存，追加到文件末尾。如果文件不存在，则创建文件并保存内容。其他使用情况参照w命令中的描述。

注1：命令a、i、=、z的地址只会指定具体的一行，命令c、d、p、w、W的地址可以指定单行或多行。

注2：0a，0i会进入输入模式，输入文本加在文本最前面。


进阶命令：

	(.,.)m(.)	移动左边源指定行到右边目的指定行后，当前行被设为移动行的最后一行。当目的地址在源地址之间但不为源地址最后一行时，输出'?'的错误提示。
	(.,.)t(.)	复制左边源指定行到右边目的指定行后，当前行被设为复制行的最后一行。目的地址可以在源地址之间。
	(.,.+1)j	合并指定行内容，当前行被设为合并行。
	(.,.)s[/str1/str2/count]
	(.,.)s[/str1/str2/g]
	(.,.)s		在指定行用str2替换str1。当最后为count（正整数）时，对指定的每一行，替换第count个匹配的str1（若该行没有，则不替换），若不指定，即(.,.)s/str1/str2/，默认每一行替换第一个。当最后为g时，替换所有。当指定行没有符合要求的或没有第count个时，输出'?'。当前行被设为最后一个被改变的行。(.,.)s重复上一次替换命令。

	注1：命令m、t左地址可以指定单行或多行，右地址只能指定具体的一行。命令j的地址只会指定多行（可以是单字符指定的多行），如果指定了一行，则对单行的合并相当于不做任何操作，也不算对文本做任何改动，同时，当前行也不变。
	注2：这些命令不打印或输出任何内容。

高级命令：

	(.)k[x]		用一个小写字符'x'标记指定行，标记成功后，可以通过"'x"这个地址（英文单引号'加上字符x）来访问该行。当被标记行被删除或修改后，该标记失效。在被标记行前后插入内容，或者移动被标记行时，该标记不会失效。可以对一行加多个标记，一个标记只能标记一行。标记一行为x后，"'x"视为一个地址，可以代入地址的运算中，如'x-2和'x,$等。被标记的地址也不会与命令冲突，如"'ii"表示在标记为i的行前插入内容。
	u			撤销上一次改动文本的命令，当前行（即'.'地址）还原为改动之前的当前行的地址。注意，撤销的是对编辑器缓存中文本的改动。可撤销多次。u本身不算作对文本的一次改动。

所有文件保存在项目根目录下，不需要删除文件。

##  示例

	示例1：
	test1内容为：
	cn
	test
	software
	命令：
	ed test1
	p
	q
	输出：
	software

	示例2：
	命令：
	ed
	a
	cn
	test
	software
	.
	w test009
	q
	输出为空
	test009内容为：
	cn
	test
	software
	

##  命令示例2

	2,4p			打印2到4行内容
	/a/p			打印向后匹配到'a'的第一行的内容
	-1a				从当前行的前一行的之后插入内容
	+2i				从当前行的后两行的之前插入内容
	1,2c			将1到2行内容替换
	$-2d			删除倒数第3行
	1,2W			将1到2行内容追加写到文件
	?favourite?ki	标记向前匹配到"favourite"的那行为'i'
	'iw favour		将标记为'i'的那行保存到favour文件中

## u命令示例

	文件test_u内容为：
	test_u
	001
	dd
	ff
	对其进行如下操作：
	ed test_u		当前地址设为4
	2p				当前地址设为2
	3d				当前地址设为3
	2a
	aa
	ww
	.				当前地址设为4
	,p				当前地址设为5
	u				当前地址设为3，即"2a"命令执行之前的当前地址
	,p				当前地址设为3
	u				当前地址设为2，即"3d"命令执行之前的当前地址
	,p				当前地址设为4
	输出：
	001				第一次输出
	test_u			第二次输出
	001
	aa
	ww
	ff
	test_u			第三次输出
	001
	ff
	test_u			第四次输出
	001
	dd
	ff
