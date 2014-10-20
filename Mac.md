#Mac OS X
##Yosemite 升级之后应该进行什么操作?  
1. 从 mavericks 升级到 mac os x 之后, 会遇到一些比较诡异的问题(比如找不到 clang), 下面的命令会帮助你提前避免一些问题. 

```bash
sudo xcode-select --install  
sudo xcode-select --switch /Applications/Xcode.app   
sudo xcodebuild -license  
sudo gem update --system  
sudo gem install xcodeproj   
sudo brew update   
```
2. 同时 Yosemite 会收集用户的隐私数据, 建议通过[fix-macosx](https://fix-macosx.com/)进行修复
