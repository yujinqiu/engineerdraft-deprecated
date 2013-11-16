## Ubuntu 类似 mac的 say 命令
### 背景

最近迷上了家庭网络组建,想要搞一个类似家庭控制中心,然后在希望能够控制远程的电脑"说话"

### 安装

默认情况下ubuntu 好像没有安装该命令, 需要手动安装. 

`
sudo apt-get install espeak
`

[espeak](http://espeak.sourceforge.net) 本质上是一个text to speech engin. 
>eSpeak is a compact open source software speech synthesizer for English and other languages, for Linux and Windows.

### 主要特点: 

- 优点: 体积小且清晰适合高速网络传输.
- 缺点: 语言听起来比较像机器,和人的自然语言差距比较大.

>eSpeak uses a "formant synthesis" method. This allows many languages to be provided in a small size. The speech is clear, and can be used at high speeds, but is not as natural or smooth as larger synthesizers which are based on human speech recordings.

### 例子


`
	echo 'hello,world' | espeak
`

神奇的体验

### 中文

如何让espeak 讲普通话呢? 

`
	espeak -vzh  '爸爸,去哪里? '
`

	-v <voice name>
              Use voice file of this name from espeak-data/voices


### 中文其它语言

[粤语](http://espeak.sourceforge.net/data/)



