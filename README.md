# Mina-SNARK暂停器 (mina-snark-stopper)
Mina协议的工具

Tool for Mina Protocol

## 描叙 (Description)
这个工具对于在同一个节点上同时运行区块生产者(block producer)和Snark工人(snark worker)的Mina验证者(validators)有一定的帮助。
Snark工人通常会占据了大部分的处理器时间，这对区块生产有负面影响。使用此工具，可以设置`STOP_WORKER_BEFORE_MIN`， 从而设定在区块产生的多少分钟前，脚本会暂停Snark工人一段时间，直到区块成功产生。而这个时间可以通过`STOP_WORKER_FOR_MIN`设置。

This tool can be useful for Mina validators who run node at same time as block producer and snark worker. 
Worker can take up all processor time, which negatively affects block producer. When less than `STOP_WORKER_BEFORE_MIN` minutes remain before the next proposal, the script disconnects the worker and starts it after `STOP_WORKER_FOR_MIN` minutes.  

## 要求 (Requirements)
Python ver. 3.6+

## 安装 (Install)
*你的Snark工人必须在运行状态中，不然脚本会直接用区块生产者的地址来作为Snark工人。*  
*配置文件里面有其他选项可以重新配置。*  
**如果你使用docker,你需要添加一个flag `-p 127.0.0.1:3085:3085`。**

*Your snark worker must be RUNNED. Otherwise, the script will take the Block producer public key*  
*Check the configuration file. There are some options you might want to reassign*  
**If you run mina daemon through docker, then you need to add the flag `-p 127.0.0.1:3085:3085`**

### Tmux 

使用以下命令安装

Install 
```
sudo apt-get update \
&& sudo apt-get install python3-venv tmux git -y \
&& git clone https://github.com/c29r3/mina-snark-stopper.git \
&& cd mina-snark-stopper \
&& python3 -m venv venv \
&& source ./venv/bin/activate \
&& pip3 install -r requirements.txt
```  
使用以下
Run  
Run  
```
tmux new -s snark-stopper -d venv/bin/python3 snark-stopper.py
```

You can watch the snark-stopper work  
`tmux attach -t snark-stopper`  

Press to exit `ctrl + b` and then `d`

### Docker  
1. Download config file and change the parameters to suit you
```
curl -s https://raw.githubusercontent.com/c29r3/mina-snark-stopper/master/config.yml > config.yml
```

2. Run docker container  
```
docker run -d \
--volume $(pwd)/config.yml:/mina/config.yml \
--net=host \
--restart always \
--name snark-stopper \
c29r3/snark-stopper
```

3. Check logs  
`docker logs -f snark-stopper`  
If you want to change some parameteres - change it in config file and then restart docker container  
`docker restart snark-stopper` 

## Troubleshooting  
If the snark-stopper can't connect to port `3085`:  
1. Check port availability  
`nc -t -vv localhost 3085`  
Output should be something like this:  
`Connection to localhost 3085 port [tcp/*] succeeded!`

If the connection hangs, then the following options are possible:  
- Access to port `3085` is blocked via ufw\iptables  
- You did not add a docker container flag `-p 127.0.0.1:3085:3085`  
- Node is not synced yet. For this reason the stopper can't connect  

2. Port responds, but the stopper still can't connect  
`iptables -D OUTPUT -d 172.16.0.0/12 -j DROP`  
it's because of the blocking of private subnets that the docker uses  

#### Update docker image  
After running the command below, go to step 2
```
docker rm -f snark-stopper; \
curl -s https://raw.githubusercontent.com/c29r3/mina-snark-stopper/master/config.yml > config.yml; \
docker pull c29r3/snark-stopper
```

## Uninstall  
```
rm -rf mina-snark-stopper; \
docker rm -f snark-stopper; \
docker system prune -af
```
