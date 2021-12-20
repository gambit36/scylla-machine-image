# ScyllaDB 宁夏区安装指南
主要由以下步骤组成
- 创建一个已经预装ScyllaDB的AMI
- 允许在第一次启动实例时配置数据库 
- 轻松创建集群 

## OS Package
在镜像中预装RPM/DEB包。由于CentOS即将终止支持，建议使用Ubuntu安装DEB包。
在实例第一次启动期间配置 ScyllaDB。 

## Create an image
### AWS
```shell script
aws/ami/build_ami.sh
```

## Scylla AMI user-data Format v2

我们会使用EC2的 user-data 功能向 ScyllaDB AMI 传递一些参数, 如下面文档描述：

如何向EC2实例传递参数:
[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-add-user-data.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-add-user-data.html)
---
### EC2 User-Data
创建EC2时，以下参数将被传递

* **Object Properties**
    * **scylla_yaml** ([`Scylla YAML`](#scylla_yaml)) – 映射到 scylla.yaml 配置文件的所有字段 
    * **scylla_startup_args** (*list*) – embedded information about the user that created the issue (NOT YET IMPLEMENTED) (*default=’[]’*)
    * **developer_mode** ([*boolean*](https://docs.python.org/library/stdtypes.html#boolean-values)) – 是否启动开发者模式 (*default=’false’*)
    * **post_configuration_script** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – A script to run once AMI first configuration is finished, can be a string encoded in base64. (*default=’’*)
    * **post_configuration_script_timeout** ([*int*](https://docs.python.org/library/stdtypes.html#int)) – Time in seconds to limit the post_configuration_script (*default=’600’*)
    * **start_scylla_on_first_boot** ([*boolean*](https://docs.python.org/library/stdtypes.html#boolean-values)) – If true, scylla-server would boot at AMI boot (*default=’true’*)

### <a href="scylla_yaml"></a>Scylla YAML
All fields that would pass down to scylla.yaml configuration file

see [https://docs.scylladb.com/operating-scylla/scylla-yaml/](https://docs.scylladb.com/operating-scylla/scylla-yaml/) for all the possible configuration availble
listed here only the one get defaults scylla AMI

* **Object Properties**    
    * **cluster_name** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – Name of the cluster (*default=`generated name that would work for only one node cluster`*)
    * **experimental** ([*boolean*](https://docs.python.org/library/stdtypes.html#boolean-values)) – To enable all experimental features add to the scylla.yaml (*default=’false’*)
    * **auto_bootstrap** ([*boolean*](https://docs.python.org/library/stdtypes.html#boolean-values)) – Enable auto bootstrap (*default=’true’*)
    * **listen_address** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – Defaults to ec2 instance private ip
    * **broadcast_rpc_address** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – Defaults to ec2 instance private ip
    * **endpoint_snitch** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – Defaults to ‘org.apache.cassandra.locator.Ec2Snitch’
    * **rpc_address** ([*string*](https://docs.python.org/library/stdtypes.html#str)) – Defaults to ‘0.0.0.0’
    * **seed_provider** (*mapping*) – Defaults to ec2 instance private ip

### Example usage of user-data

Spinning a new node connecting to “10.0.219.209” as a seed, and installing cloud-init-cfn package at first boot.

```json
{
     "scylla_yaml": {
         "cluster_name": "test-cluster",
         "experimental": true,
         "seed_provider": [{"class_name": "org.apache.cassandra.locator.SimpleSeedProvider",
                            "parameters": [{"seeds": "10.0.219.209"}]}],
     },
     "post_configuration_script": "#! /bin/bash\nyum install cloud-init-cfn",
     "start_scylla_on_first_boot": true
}
```

## Creating a Scylla cluster using the Machine Image
### AWS - CloudFormation
使用模版文件 `aws/cloudformation/scylla.yaml`.
当前模版支持创建最大10个节点的集群.

## 构建 scyllaDB 镜像

### Ubuntu - DEB

```
dist/debian/build_deb.sh
```

Build using Docker

```
docker run -it -v $PWD:/scylla-machine-image -w /scylla-machine-image  --rm ubuntu:20.04 bash -c './dist/debian/build_deb.sh'
```

## Building docs

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install sphinx sphinx-jsondomain sphinx-markdown-builder
make html
make markdown
```

