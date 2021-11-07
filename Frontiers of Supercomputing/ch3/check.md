##### check.sh

```
 #!/bin/sh
 #电脑概览
 #电脑型号
 ComputerModel=`/usr/bin/sudo /usr/sbin/dmidecode | grep -A2 "System Information" | awk -F ':' '{print $2}' | grep -v '^$' |sed -e 's/Inc//g' -e 's/ //g' -e ':a;N;s/\n/ /g;ta' -e 's/.//g'`
 x86_64=`getconf LONG_BIT`
 #系统版本
 SystemVersion=`cat /etc/redhat-release`
 #内核版本
 KernelVersion=`uname -r`
 #CPU信息,1物理CPU个数2,查看每个物理CPU中core的个数(即核数),查看逻辑CPU的个数(即线程).CPU型号
 CPUNum=`cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l`
 CPUNucleusNum=`cat /proc/cpuinfo| grep "cpu cores"| uniq | awk -F ':' '{print $2}' | sed 's/ //g'`
 CPUThreadNum=`cat /proc/cpuinfo| grep "processor"| wc -l`
 CPUmodel=`cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq | sed 's/ //g'`
 CPUTotalnuclearNum=$[$CPUNum*$CPUNucleusNum]
 #主板型号,.主板厂商与型号,.主板版本
 MotherBoardModel=`/usr/bin/sudo /usr/sbin/dmidecode | grep -A2 "Base Board Information$" | awk -F ':' '{print $2}' | grep -v '^$' |sed -e 's/Inc//g' -e 's/ //g' -e ':a;N;s/\n/ /g;ta' -e 's/.//g'`
 MotherBoardVersion=`/usr/bin/sudo /usr/sbin/dmidecode | grep -A3 "Base Board Information$" | awk -F ':' '{print $2}' | grep -v '^$' |sed -e 's/Inc//g' -e 's/ //g' -e ':a;N;s/\n/ /g;ta' -e 's/.//g' | awk '{print $3}'`
 #内存,.内存总数2,内存插槽编号,3内存规格,4内存数组,.循环出所有的内存条参数,6内存显示,7内存数量,8总插槽数量,9支持最大内存
 MemoryNum=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep Size | awk '{print $2}' | grep -v 'No' | awk '{sum +=$1};END{print sum/1024}'`
 MemoryName=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep '^Handle' | awk '{print $2}' | sed 's/,//g'`
 MemoryNameDDR=""
 MemoryArray=
 for i in ${MemoryName[@]}
 do
     Memorytest=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep -A19 '^Handle '$i'' | grep -E 'Configured Clock Speed|Speed' | grep 'Unknown'`
     if [ $? -eq  ];then
         continue
     fi
     MemoryNameDDR[$MemoryArray]=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep -A19 '^Handle '$i'' | grep -E 'Type|Configured Clock Speed|Speed' | grep -Ev 'Error|Unknown' | awk -F ':' '{print $2}' | sed -e '2{h;d};3G' | sed -e ':a;N;s/\n/ /g;ta' | awk '{for(i=4;i<=NF;++i) printf $i " ";print $1,$2,$3,"\n"}'`
     let MemoryArray=MemoryArray+
 done
 MemoryEcho=`echo ${MemoryNameDDR[*]} | sed -e 's/MHz/MHz\n/g' | sed -e 's#^ ##g' | sort -r | uniq |grep -v '^$'`
 Memory=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep -E Size | grep -Ev  'Installed Size|Maximum Memory Module Size|Maximum Total Memory Size:|Enabled Size:|No' | wc -l`
 MemoryTotalSlotNum=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep "Number Of Devices:" | awk -F':' '{print $2}' | sed 's/ //g'`
 MemoryMaximumCapacity=`/usr/bin/sudo /usr/sbin/dmidecode -t memory | grep "Maximum Capacity:" | awk -F':' '{print $2}' | sed 's/ //g'`
 #硬盘1,获取所有硬盘盘符,,只获取第一块硬盘信息,.储存判断信息,4获取服务器厂商名,5根据服务器厂商名获取磁盘厂商名
 diskNum=`ls /dev/sd* | grep -v '[0-9]$' | awk -F '/' '{print $3}'`
 diskVersion=`/usr/bin/sudo /usr/sbin/smartctl --all /dev/${diskNum[]} | grep -E 'Vendor|Product|User Capacity|Rotation Rate' | grep -v 'cache' | awk -F ':' '{print $2}' | sed -e 's# ##g' -e 's/\[/[\n/g' -e 's#]##g' | sed -e '/\[/d' -e 's/rpm//g' | sed ':a;N;s/\n/ /g;ta' | awk '{print $1,$2,"("$3"/"$4"/分)"}'`
 diskYes=""
 diskComputerModel=`/usr/bin/sudo /usr/sbin/dmidecode | grep -A1 "System Information" | awk -F ':' '{print $2}' | grep -v '^$' |sed -e 's/Inc//g' -e 's/ //g' -e 's/,//g' | awk -F '.' '{print $1}' `
 diskRAID=`cat /proc/scsi/scsi | grep Vendor | sed -e 's/Model/\nModel/g' | sed -e '/Model/d' | grep -Eo $diskComputerModel'|VMware'`
 #diskSize=`fdisk -l | grep "Disk" | awk '{print $3}' | awk '{sum +=$1};END{print sum}'`
 diskSize=`df -P | grep -v 'Filesystem' | awk '{sum +=$2};END{print sum/1024/1024}'`
 if [ "$diskRAID" = "VMware" ];
 then
     diskYes='unknown(Because the hard drive is VMware)'
 elif [ "$diskComputerModel" = "$diskRAID" ];
 then
     diskYes='Yes'
 else
     diskYes='No'
 fi
 #显卡
 VGA=`/usr/bin/sudo /sbin/lspci |grep VGA | awk -F ':' '{print $3}' | awk -F '.' '{print $1$2}'| sed -e 's#^ ##g'`
 #网卡
 network=`/usr/bin/sudo /sbin/lspci | grep Ethernet | awk -F ':' '{print $3}' | uniq | sed 's/^ //g'`
 #系统序列号
 SystemSerialNum=`/usr/bin/sudo /usr/sbin/dmidecode -s system-serial-number`
 #打印
 echo -e "Server model"'\t'$ComputerModel
 echo -e "serial number"'\t'$SystemSerialNum
 echo -e "system version"'\t'$SystemVersion"X"$x86_64
 echo -e "Kernel version"'\t'$KernelVersion
 echo -e '\n'
 echo -e "processor  "'\t'$CPUmodel"(*"$CPUNum") "$CPUTotalnuclearNum"核"
 echo -e "Motherboard"'\t'$MotherBoardModel"("$MotherBoardVersion")"
 echo -e "RAM    "'\t\t'$MemoryNum" GB"
 echo -e "Total Slots"'\t'""$MemoryTotalSlotNum
 echo -e "Used Slots"'\t'$Memory " Maximum memory support:"$MemoryMaximumCapacity
 echo -e "Hard Details"'\t'"RAID:"$diskYes "capacity:"$diskSize"G"
 echo -e "The first disk"'\t'$diskVersion
 #echo -e "显卡    "'\t'$VGA
 echo -e "NIC    "'\t\t'$network | sed 's/) [A-Z a-z 1-9]/)\n &/g' | sed -e 's/^ /\t\t/g' -e 's/\t) /\t/g' | grep -v '^$'
```

