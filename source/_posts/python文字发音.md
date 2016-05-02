title: python文字发音
date: 2015-08-12 20:46:05
tags: pyton
---

* [python-2.7.9.msi](https://www.python.org/downloads/release/python-279/)
* [pywin32-219.win32-py2.7.exe](http://sourceforge.net/projects/pywin32/files/pywin32/Build%20219/)(要翻墙)
 
安装python 配置环境变量，使用pip安装pyttsx

	
		# coding=utf-8
		
		#@see http://pyttsx.readthedocs.org/en/latest/engine.html#examples
		import pyttsx
		engine = pyttsx.init()
		
		#语速
		rate = engine.getProperty('rate')
		#print rate
		
		#音量
		volume = engine.getProperty('volume')
		#print volume
		
		engine.setProperty('rate', 100)
		engine.setProperty('volume',  2000)
		
		#语音类别
		voices = engine.getProperty('voices')
		engine.say(u'李洋是逗比')
		
		'''
		for voice in voices:
		   engine.setProperty('voice', voice.id)
		   print voice.id
		'''
		
		engine.runAndWait()