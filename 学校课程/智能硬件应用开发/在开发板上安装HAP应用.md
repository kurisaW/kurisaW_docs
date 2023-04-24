## 前提准备：[ 编写通过JS应用控制LED灯](https://gitee.com/bearpi/bearpi-hm_micro_small/blob/master/applications/BearPi/BearPi-HM_Micro/docs/device-dev/%E9%80%9A%E8%BF%87JS%E5%BA%94%E7%94%A8%E6%8E%A7%E5%88%B6LED%E7%81%AF.md)

---

## 参考：[ 如何在开发板上安装HAP应用](https://gitee.com/bearpi/bearpi-hm_micro_small/blob/master/applications/BearPi/BearPi-HM_Micro/docs/device-dev/%E5%A6%82%E4%BD%95%E5%9C%A8%E5%BC%80%E5%8F%91%E6%9D%BF%E4%B8%8A%E5%AE%89%E8%A3%85HAP%E5%BA%94%E7%94%A8.md)

注意：此处我们不使用sd卡

完成准备工作后，按下RESET键重启开发板并进入uboot模式

```c
cd /vendorhap_tools/hap_example

cp bm /
cp LED_1.0.0.hap /
```

重启开发板不需要进入uboot模式，并进入hap_example

```c
cd /vendorhap_tools/hap_example
```

输入以下命令，打开调试模式

```
./bm set -s disable
./bm set -d enable安装应用
```

```
./bm install -p LED_1.0.0.hap
```

注: LED_1.0.0.hap为安装包名称，安装其他应用需要修改为对应的安装包名称。