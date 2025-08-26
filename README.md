#!/bin/sh
set -e

install_tool() {
    which "$1" >/dev/null 2>&1 && return 0
    if which apt >/dev/null 2>&1; then
        apt update
        apt -y install "$1"
        return 0
    fi
    if which yum >/dev/null 2>&1; then
        yum -y install "$1"
        return 0
    fi
    echo "没找到包管理器"
    return 1
}

install_docker() {
    install_tool curl
    which docker >/dev/null 2>&1 && return 0
    curl -fsSL https://get.docker.com | sh
    systemctl start docker
    systemctl enable docker
}

while true; do
    clear
    cat <<EOF
1:轻语
EOF
    read -p "请选择产品:" product
    if [ "$product" != "1" ] ; then
        read -p "输入错误，请重新输入!" msg
    else
        break;
    fi
done

read -p "请输入域名:" domain

read -p "按回车开始安装" msg

echo "安装docker"
install_docker
echo "安装wget"
install_tool wget
echo "安装unzip"
install_tool unzip
echo "安装产品"
mkdir -p /docker
cd /docker
case "$product" in
#1:轻语
	"1" )
		cd /root/
		wget -O docker_qy.zip 'http://192.227.231.163:8080/s/Ms27qLgAtp8woWZ/download/docker_qy.zip'
		unzip docker.zip
		mv /root/docker /docker/qy
		cd /docker/qy
		sh server.sh "$domain"
		chmod -R 777 rocketmq
		docker compose up -d
		sleep 10
		docker compose exec mqbroker sh mqadmin updateTopic -c DefaultCluster -t pushMessage
		docker compose exec mqbroker sh mqadmin updateTopic -c DefaultCluster -t xmppMessage
		docker compose exec mqbroker sh mqadmin updateTopic -c DefaultCluster -t userStatusMessage
		echo "安装完成，请打开后台设置页面设置"
		echo "如果有XMPP的，domain设置成xmpp.local"
		
		rm /root/docker.zip
		rm -rf /root/__MACOSX
		rm /docker/qy/docker-compose.yaml
		rm /docker/qy/docker-compose暴漏端口.yaml
		;;
esac

