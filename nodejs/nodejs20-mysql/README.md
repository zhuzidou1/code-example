# nodejs20 runtime 的 mysql 示例
本示例为您展示了 Nodejs runtime 的 mysql 使用示例。
在本示例中，mysql数据库配置在函数的环境变量配置中（参考s.yaml)，initializer 回调函数从环境变量中获取数据库配置，创建 mysql 连接，preStop 回调函数负责关闭 mysql 连接。

> nodejs mysql 官方文档: [nodejs mysql](https://github.com/mysqljs/mysql)

## 准备开始
- 一个可用的mysql数据库，可以参考以下sql创建表并插入测试数据

```sql
CREATE TABLE `users` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `age` tinyint(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `users` (`id`, `name`, `age`) VALUES
(1, '张三', 18),
(2, '李四', 28);
```

- [可选] 安装并配置 Serverless Devs 工具，以及 docker daemon。（https://help.aliyun.com/document_detail/2513712.html）

## 快速开始
### 方式一、使用 Serverless Devs 工具编译部署

#### 1. 修改 s.yaml 配置
- 根据需要修改 access 配置
- 修改 environmentVariables 配置，填入 MYSQL_USER, MYSQL_PASSWORD, MYSQL_ENDPOINT, MYSQL_PORT 和 MYSQL_DBNAME
- 使用 initialize 和 prestop 回调，需要在 s.yaml 中配置

```yaml
        initializationTimeout: 20
        initializer: index.initialize
        instanceLifecycleConfig:
          preStop:
            handler: index.preStop
            timeout: 20
```

#### 2. 安装依赖并部署

安装依赖库 mysql
```shell
# 使用s工具安装依赖，需要使用 docker
s build --use-docker
```
> 注意: 使用 `s build --use-docker` 会将依赖安装到代码包的根目录下，可以在本地的 `.s/build/artifacts/{serviceName}/{functionName}/` 查看。

部署代码
```bash
# 部署代码
s deploy
```

#### 3. 调用测试

```shell
s invoke
```

调用函数时收到的响应如下所示：

```bash
2025-04-24 14:15:24FC Invoke Start RequestId: 1-6809d6fc-******
2025-04-24 14:15:242025-04-24 06:15:24.086 1-6809d6fc-****** [info] --------------------------SELECT----------------------------
2025-04-24 14:15:242025-04-24 06:15:24.088 1-6809d6fc-****** [info] [ RowDataPacket { id: 1, name: '张三', age: 18 } ]
2025-04-24 14:15:242025-04-24 06:15:24.089 1-6809d6fc-****** [info] ------------------------------------------------------------

2025-04-24 14:15:24FC Invoke End RequestId: 1-6809d6fc-******
2025-04-24 14:21:16FC PreStop Start RequestId: e63c7c4d-******
2025-04-24 14:21:162025-04-24 06:21:16.643 e63c7c4d-****** [info] pre_stop start
2025-04-24 14:21:16FC PreStop End RequestId: e63c7c4d-******
```

### 方式二、使用控制台创建

#### 1. 安装依赖和部署代码包

```shell
# 安装依赖到 /code 目录
cd code && npm install
# 打包文件
cd code && zip -r nodejs20-mysql.zip *
```

创建函数并上传代码包

#### 2. 设置initializer/preStop回调函数配置和环境变量配置

回调函数配置
![img_1.png](assets/20250424100830.jpg)

环境变量配置
![img_2.png](assets/20220331111048.jpg)

#### 3. 调用测试
![img_3.png](assets/20250424143042.jpg)

## 数据库访问限制
当使用云数据库时，一般都会有访问控制，比如阿里云数据库RDS中的白名单设置（ [RDS白名单设置说明](https://help.aliyun.com/document_detail/43185.html?spm=5176.19908528.help.dexternal.6c721450iLu0jH) )。

如果仅仅作为测试，可以将白名单配置成 `0.0.0.0/0`。（不要在生产环境使用!)

在生产环境，可以使用以下两种方式访问：

1. VPC方式（**推荐**） <br>
参考文档：https://help.aliyun.com/document_detail/2513584.html
2. 代理方式<br>
参考文档：https://help.aliyun.com/document_detail/2513538.html

## 备注
1. 本示例不是连接池方式，若要使用连接池，可以参考文档 [nodejs mysql pooling connections](https://github.com/mysqljs/mysql#pooling-connections)
