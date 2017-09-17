#!/bin/sh
echo Content-type: text/plain
echo
if [ "$REQUEST_METHOD" = "GET" ] ; then		
	echo GET is not expected without https
	exit                     
fi
#/bin/date
now_date=$(date "+%Y-%m-%d %H:%M:%S")
file=systemlog.tar
file_back=/www/systemlog.tar
pwd=/tmp/log/
#check_backlog
check_backlog()
{
if [ -f "$file_back" ]; then
	rm -rf "$file_back"
fi
}
#check path
check_path()
{
if [ ! -d /tmp/log/ ]; then     
    mkdir -p /tmp/log;
fi
}
#cpu info
cpu_check()
{
	top -b -n 1 > /tmp/log/cpu.txt
	sed -i "1 s/^/$now_date\n/" /tmp/log/cpu.txt 
}
check_process()
{
	ps > /tmp/log/ps.txt
}
#mem_info
mem_info()
{
	free > /tmp/log/mem_info.txt
	sed -i "1 s/^/$now_date\n/" /tmp/log/mem_info.txt
}
check_ipset()
{
	ipset list > /tmp/log/ipset.txt
}
check_conf
{
	ls -l /etc/shata/ > /tmp/log/conf.txt
}
#get the number of box connection hosts
Terminal_quantity()
{
	cat /proc/net/arp | grep -v IP | grep -v br-lan | wc -l > /tmp/log/Terminal_quantity.txt
	cat /proc/net/arp > /tmp/log/ip.txt
	sed -i "1 s/^/$now_date\n/" /tmp/log/ip.txt
}

system_load()
{
	cat /proc/loadavg > /tmp/log/loadavg.txt
	sed -i "1 s/^/$now_date\n/" /tmp/log/loadavg.txt
}
#获取已连接终端实时流量
Host_traffic_total()
{
	cat /proc/net/arp | grep : | grep ^192 | grep -v 00:00:00:00:00:00| awk '{print $1}'> mac-arp 
	iptables -N UPLOAD 
	iptables -N DOWNLOAD
	while read line;do iptables -I FORWARD 1 -s $line -j UPLOAD;done < mac-arp
	while read line;do iptables -I FORWARD 1 -d $line -j DOWNLOAD;done < mac-arp
	
	sleep 1

	iptables -nvx -L FORWARD | grep DOWNLOAD | awk '{print $2/1024/1" KB/s ",$1/10" packets/s", $9}' | sort -n -r > /tmp/log/Download.txt
	iptables -nvx -L FORWARD | grep UPLOAD | awk '{print $2/1024/1" KB/s ",$1/10" packets/s", $8}' | sort -n -r  > /tmp/log/Upload.txt
	while read line;do iptables -D FORWARD -s $line -j UPLOAD;done < mac-arp 
	while read line;do iptables -D FORWARD -d $line -j DOWNLOAD;done < mac-arp  
	iptables -X UPLOAD  
	iptables -X DOWNLOAD 
}
#get wan ip
wan_ip()
{
	ubus call network.interface.wan status | grep \"address\" | grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' > /tmp/log/wan_ip.txt
}
#get Gateway_information info
Gateway_information()
{
	ubus call network.interface.wan status | grep nexthop | grep -oE '([0-9]{1,3}.){3}.[0-9]{1,3}' > /tmp/log/gateway.txt
}
#get uptime
uptime()
{
	cat /proc/uptime| awk -F. '{run_days=$1 / 86400;run_hour=($1 % 86400)/3600;run_minute=($1 % 3600)/60;run_second=$1 % 60;printf("系统已运行：%d天%d时%d分%d秒",run_days,run_hour,run_minute,run_second)}' > /tmp/log/uptime.txt
}
Connectivity_test()
{
	timeout=5
	target=www.baidu.com
	ret_code=`curl -I -s --connect-timeout $timeout $target -w %{http_code} | tail -n1`
	if [ "x$ret_code" = "x200" ]; then
		echo "dns正常" > /tmp/log/dns_test.txt
	else
		echo "dns错误"  > /tmp/log/dns_test.txt
	fi
	sed -i "1 s/^/$now_date\n/" /tmp/log/dns_test.txt
}
Logread()
{
	logread > /tmp/log/logread.txt
	dmesg > /tmp/log/dmesg.txt
}
Component_status()
{
	#opkg update > /dev/null
	opkg list-installed > /tmp/log/list_installed.txt
	#opkg list-upgradable > /tmp/log/list_upgrade.txt
	#opkg list_installed | sed 's/ - .*//' | sed 's/^/opkg upgrade /' > /dev/null #自动升级
}
wifi_pass()
{
	
	cat /etc/config/wireless  > /scripts/wificonfig.txt
}
Connection_number()
{
	sed -n 's%.* src=\(192.168.[0-9.]*\).*%\1%p' /proc/net/nf_conntrack | sort | uniq -c > /tmp/log/ip_connection_number.txt
}
Get_Route()
{
	route > /tmp/log/route.txt
}
Get_Public_Network_IP()
{
	result_ping=`ping 114.114.114.114 -c 3`
	if [ $? -eq 0 ]; then
   		public_ip=`wget http://ipecho.net/plain -O - -q ; echo`
		echo $public_ip > /tmp/log/public_network_ip.txt 
	else
    	echo "notwork error,no public_network_ip" > /tmp/log/public_network_ip.txt
	fi
	
}
ping_test()
{
	 result=`ping 114.114.114.114 -c 3 ` 
	 if [ $? -eq 0 ]; then
	 	echo "ping test is ok" > /tmp/log/ping.txt
	 else
	 	echo "ping test is fail" > /tmp/log/ping.txt
	 fi
	echo $result >> /tmp/log/ping.txt
}
tar_log()
{
zip -P passwd1234 /www/systemlog.tar /tmp/log/* > /dev/null 
rm -rf /tmp/log
echo  "$file"
}
check_path #检查目录
check_backlog #检查是否有log
cpu_check #检查cpu
mem_info  #检查内存
check_ipset #检查ipset列表
check_conf #检查配置文件
check_process #检查进程
Terminal_quantity #检查连接终端数
system_load #检查系统平均负载
Host_traffic_total #检查当前下载和上传速率
wan_ip #获取wan ip
Gateway_information #获取网关
uptime #获取运行时间
Connectivity_test #查看网络是否正常
Logread #日志收集
Component_status #系统环境检查
Connection_number #查看已连接终端建立连接数
Get_Route #获取盒子路由状态
Get_Public_Network_IP #获取公网ip
ping_test #ping 测试
tar_log #加密打包




