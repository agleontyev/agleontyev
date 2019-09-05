---
published: true
tags: other
---

For those who like to use Mac for code development in R, python (or any other language), XCode is a necessary dependancy. However, because of Apple's decision, Xcode tools ship with support packages for mobile systems (iOS) and Apple Watch. If you are, like me, data scientists, chances are you are not out there making applications for apple watch or IPhone. To significantly -- in my case, 8 GB -- reduce the size of Xcode, we can delete the support packages for mobile devices. To do this, open Terminal and paste the following commands(it takes some time, especially on the older machines, to execute them):
```shell
cd /Applications/Xcode.app/Contents/Developer/Platforms/
sudo rm -rf AppleTV* Watch* iPhone*
```
Optionally, you can also remove support for SWIFT:
```shell
sudo rm -rf /Applications/Xcode.app/Contents/Developer/Toolchains/Swift_2.3.xctoolchain
```
