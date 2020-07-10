



## Ubuntu18

### 配置静态IP
```shell
mv /etc/netplan/{50-cloud-init.yaml,50-cloud-init.yaml.backup}
# cat << EOF > /etc/netplan/50-cloud-init.yaml
network:
    ethernets:
        enp1s0:
            dhcp4: no
            addresses: [192.168.0.101/24]
            optional: true
            gateway4: 192.168.0.1
            nameservers:
                    addresses: [192.168.1.1,114.114.114.114]
        enp2s0:
            dhcp4: true
    version: 2
EOF
# 应用配置
netplan apply
```
