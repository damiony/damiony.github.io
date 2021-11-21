# 【Elasticsearch】使用docker安装ElasticSearch


## 前言

每次搭建编程环境的时候，都会忘记`ElasticSearch`的安装方法，然后就得查资料研究好一会儿，因此现在将安装过程记录下来，方便以后查阅。

本文使用的是`Docker`安装方式。

## Docker安装单节点Elasticsearch

### 1. 下载镜像

首先下载`ElasticSearch`的`Docker`镜像，具体版本可以根据自己的需要来指定，此处下载的是`7.8.0`版本。

```shell
docker pull elasticsearch:7.8.0
```

### 2. 创建目录

创建三个目录，用于存放数据、插件和配置。

```shell
# 目录位置可以根据需要来指定
mkdir /data/elasticsearch/{data, config, plugins}
```

在`config`目录下创建`elasticsearch.yml`文件，用于存放配置，并在配置文件中加入基本配置。

```shell
# 创建配置文件
touch elasticsearch.yml
# 写入配置
echo 'cluster.name: "elasticsearch"\nnetwork.host: 0.0.0.0' >> elasticsearch.yml
```

### 3. 运行容器

运行`Docker`容器。

```shell
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
    -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms128m -Xmx128m" \
    -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
    -v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
    -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
    -d elasticsearch:7.8.0
```

`discovery.type=single-node`表示安装的`es`为单节点。

`ES_JAVA_OPTS`用于指定`ElasticSearch`能使用的内存，如果内存过大会导致无法使用`Docker`容器。`-Xms128m`表示最少能用`128MB`，`-Xmx128m`表示最多能用`128MB`。

`-v`用于将宿主机的目录映射到容器内，注意得先分配目录的权限。

### 4. 检验是否成功

浏览器打开`localhost:9200`，如果出现如下内容，则表示安装成功。

```shell
{
    name: "e6810bbefb2b",
    cluster_name: "elasticsearch",
    cluster_uuid: "SFTOoP2eSTKPEjl-L4325A",
    version: {
        number: "7.8.0",
        build_flavor: "default",
        build_type: "docker",
        build_hash: "757314695644ea9a1dc2fecd26d1a43856725e65",
        build_date: "2020-06-14T19:35:50.234439Z",
        build_snapshot: false,
        lucene_version: "8.5.1",
        minimum_wire_compatibility_version: "6.8.0",
        minimum_index_compatibility_version: "6.0.0-beta1"
    },
    tagline: "You Know, for Search"
}
```

`name`是节点名称，`cluster_name`是集群名称，可以通过修改`elasticsearch.yml`来修改它们，如：

```yml
cluster.name: myes
node.name: master
```

这样就完成了修改，重启`elasticsearch`即可看到变化。

## Docker安装HEAD插件

`HEAD`插件提供了一个可视化的界面，供我们查看集群信息。安装命令为:

```shell
docker run --name eshead -d -p 9100:9100 mobz/elasticsearch-head:5
```

接着浏览器访问`http://localhost:9100`，就能看到相应界面，但是此时还存在跨域问题，修改`elasticsearch.yml`即可解决。

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## Docker安装kibana

`kibana`提供了一个操作`ElasticSearch`的图形化界面。当然，`kibana`不是必须得安装。

```shell
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://{ip地址}:9200 -p 5601:5601 -d kibana:7.8.0
```

`docker pull`操作并不是必需的，如果`docker run`找不到镜像，`docker`内部也会先去执行`pull`操作。

本文所有的`Elasticsearch`请求命令，都默认在`kibana`工具中操作。

## 安装分词器

### 1. 说明

进行搜索的时候，`ElasticSearch`内部往往会对输入语句进行分词操作，分成一个个单词短语，然后进行检索。但是`ElasticSearch`本身没有支持中文的分词器，所以需要自行安装。

### 2. 安装

安装步骤为：

- 在`elasticsearch/plugins/`目录下创建`ik`目录。
- 进入下载页面[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)。
- 下载所需版本，然后解压到`ik`，并重启`elasticsearch`。

### 3. 测试

为了测试分词器是否安装成功，先创建一个索引：

```shell
PUT /test
```

然后发送请求：

```shell
POST /test/_analyze
{
    "analyzer": "ik_max_word",
    "text": "美国留给伊拉克的是个烂摊子吗"
}
```

如果请求处理成功，则分词器安装成功。

### 4. 配置自定义词库

有时候，分词器的结果并不能符合我们的期望，这时候可以自定义词库。

自定义词库是一个配置文件，文件内容为自定义的词，每个词独占一行，如：

```shell
朝辞白帝彩云间
千里江陵一日还
```

可以将配置文件命名为`ext.dic`，然后放置在`elasticsearch/plugins/ik/config`中。当然，文件名和位置可以根据自己的需要进行调整。

打开`elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

修改第6行然后重启`elasticsearch`即可，注意`ext.dic`的路径必须正确。

```xml
<entry key="ext_dict">ext.dic</entry>
```

上述步骤为配置本地自定义词库，还可以配置网络自定义词库，网络词库支持热更新。

比如可以使用`nginx`或者自定义`http server`，来提供网络词库。

## 结语

至此，`ElasticSearch`的环境就安装结束了。
