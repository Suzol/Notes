[参考](https://blog.csdn.net/ApeLife/article/details/92386968)

镜像ubuntu-12.04.5-alternate-amd64.iso

## 创建虚拟机
1. 虚拟机物理内存3G（2G在跑testpmd时报了内存不够）
2. 4个cpu
3. 5个网卡（桥接，4个网卡dpdk，一个与宿主机通信）
4. ifconfig ethx(x=1,2,3,4) down (为了后面绑定dpdk)

## 编译安装
```
RTE_SDK        dpdk安装目录
RTE_TARGET     编译器（根据机器类型选择）
EXTRA_CFLAGS   打开调试选项
```
### 1.环境变量
```
cd /home/dpdk-1.8.0/

vim dpdkenv
# write into dpdkenv
export RTE_SDK=`pwd` 
export RTE_TARGET=x86_64-native-linuxapp-gcc
export EXTRA_CFLAGS="-O0 -g"
# end of write

source dkdpenv

```
建议在此处保存一下快照，后面可能要回退



### 2.编译dpdk源码
`make install T=x86_64-native-linuxapp-gcc`

### 3.设置大页内存
```
# 默认每个大页2M 这里设置256M 大页内存一共512M
echo 256 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_huge

mkdir /mnt/huge
#挂载大页系统
mount -t hugetlbfs nodev /mnt/huge

设置完了可以查看下
cat /proc/meminfo | grep Huge
cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_huge
如果内存不够，实际会小于256，这时就要考虑修改虚拟机内存了
```
### 4.安装igb_uio驱动进行绑定
```
dpdk编译完成会生成如下驱动
x86_64-native-linuxapp-gcc/kmod/igb_uio.ko
加载驱动到内核
modprobe uio
insmod x86_64-native-linuxapp-gcc/kmod/igb_uio.ko

```

### 5.将物理网卡和igb_uio绑定
```
查看网卡信息
./tools/dpdk_nic_bind.py --status

绑定
./tools/dpdk_nic_bind.py -b igb_uio 0000:02:02.0
./tools/dpdk_nic_bind.py -b igb_uio 0000:02:03.0
./tools/dpdk_nic_bind.py -b igb_uio 0000:02:04.0
./tools/dpdk_nic_bind.py -b igb_uio 0000:02:05.0

再次查看
./tools/dpdk_nic_bind.py --status
被绑走的网卡通过ip a是找不到的
如果想解除并重新与e1000绑定
./tools/dpdk_nic_bind.py -u  0000:02:00.0
./tools/dpdk_nic_bind.py -b e1000  0000:02:00.0 

```

## 测试
testpmd
```

4个cpu， 因此对应的掩码为0xf;  -n 2表示内存通道为2
/home/dpdk-1.8.0/x86_64-native-linuxapp-gcc/app# ./testpmd -c 0xf -n 2 -- -i

虚拟机需要跳过pci检查，修改
lib/librte_eal/linuxapp/igb_uio/igb_uio.c里的
if (pci_intx_mask_supported(dev))为
if (pci_intx_mask_supported(dev) || true)
重新编译，卸掉原有驱动，加载新的驱动，或者回退到原来的快照，先修改代码，按步骤执行

```

helloworld
```
cd /home/dpdk-1.8.0/examples/helloworld/
make
cd build
./helloworld -c 0x0f -n 2 -- -i

hello from core 1
hello from core 2
hello from core 3
hello from core 0

```
l2fwd
```
cd /home/dpdk-1.8.0/examples/l2fwd/
make
cd build
./l2fwd -c 0xf -n 2 -- -q 2 -p 0xf

参数说明
-c 0xf 4个cpu
-n 2   内存通道为2
--前为EAL环境参数 --后为程序参数
-q 2   每个cpu有两组收发队列
-p 0xf 指定网卡，前面绑定了4个网卡为0xf

结果：
Port statistics ====================================
Statistics for port 0 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Statistics for port 1 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Statistics for port 2 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Statistics for port 3 ------------------------------
Packets sent:                        0
Packets received:                    0
Packets dropped:                     0
Aggregate statistics ===============================
Total packets sent:                  0
Total packets received:              0
Total packets dropped:               0
====================================================
```

## GDB调试DPDK源码
以上面的helloworld为例
```
gdb ./helloworld
r -c 0xf -n 2 -- -i

```
或者到x86_64-native-linuxapp-gcc下make


总结下：编译（生成驱动）、Load驱动、设置大页内存、网卡与驱动绑定

接下来对dpdk源码分析
