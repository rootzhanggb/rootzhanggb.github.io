# kafka监控
**基于获取K8S中的borkers判断kafka运行情况，采用钉钉接口报警**
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- 
import os
import sys
#路径
base_dir = os.path.dirname(os.path.realpath(__file__))
# 钉钉报警脚本
ding = '''#!/bin/bash
curl 'https://oapi.dingtalk.com/robot/send?access_token=d7b486a4cc42b8b1d00bb0d6db58a077aeab6fd8c4763ef8590fa8287dc
d4735' \\
   -H 'Content-Type: application/json' \\
   -d '{
                "msgtype": "text", 
                "text": {
                "content": "云基kafka-'${1}'没有正常运行，请登陆检查"
                                }
        }'
'''
with open(base_dir+'/ding.sh','w') as f:
                f.write(ding)
os.system('chmod a+x ' +base_dir+'/ding.sh')
# 获取kafka的ids
message = os.popen("kubectl exec -it zookeeper-cluster-0 -n fs-public -- ./bin/zkCli.sh -server zookeeper-in:2181 l
s /brokers/ids|sed -n '$p'")
result = message.read().replace("[","").replace("]","").replace(",","").replace("\r","").replace("\n","").replace("
 ","")
result_list = list(result)
# 目前是三个集群
if len(result_list) < 3:
        for num in range(3):
                if str(num) not in result_list:
                        os.system('./ding.sh'+' '+str(num))


else:
        print "nothing to do"
```
