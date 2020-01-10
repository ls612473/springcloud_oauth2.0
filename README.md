# springcloud_oauth2.0
一个基于Spring Cloud+OAuth2.0+Spring Security+Redis的统一认证脚手架


## 未认证:
![未认证](https://img-blog.csdnimg.cn/20181217161623990.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "未认证")

## 获取认证:
![获取认证](https://img-blog.csdnimg.cn/20181217161815491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "获取认证")

##　使用access_token请求auth服务下的用户信息接口
![使用access_token请求auth服务下的用户信息接口](https://img-blog.csdnimg.cn/20181217162029489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "用户信息")

##　使用access_token请求member服务下的用户信息接口
![使用access_token请求member服务下的用户信息接口](https://img-blog.csdnimg.cn/20181217162043657.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "用户信息")

## 请求member服务的query接口
![请求member服务的query接口](https://img-blog.csdnimg.cn/20181217162128297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "query接口")

## 请求member服务的hello接口，数据库里并没有给用户hello权限
![请求member服务的hello接口](https://img-blog.csdnimg.cn/20181217162225574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "hello接口")
##　刷新token
![刷新token](https://img-blog.csdnimg.cn/20181217162313457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "刷新token接口")
##　注销
![注销](https://img-blog.csdnimg.cn/2018121716234355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "注销")


统一回复一下，有很多人反映获取认证时返回401，如下：
{
    "timestamp": "2019-08-13T03:25:27.161+0000",
    "status": 401,
    "error": "Unauthorized",
    "message": "Unauthorized",
    "path": "/oauth/token"
}

![认证](https://img-blog.csdnimg.cn/2019081311263210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "认证")

添加Basic Auth认证后会在headers添加一个认证消息头：
![认证](https://img-blog.csdnimg.cn/20190813112807146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70 "认证")

添加Basic Auth认证的信息在代码中有体现：
![认证](https://img-blog.csdnimg.cn/20190813113045280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

## 客户端信息和token信息从MySQL数据库中获取
现在客户端信息都是存在内存中的，生产环境肯定不可以这么做，要支持客户端的动态添加或删除，所以我选择把客户端信息存到MySQL中。
首先，创建数据表，数据表的结构官方已经给出，地址:
https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql

###　其次，需要修改一下sql脚本，把主键的长度改为128，LONGVARBINARY类型改为blob，调整后的sql脚本：
create table oauth_client_details (
  client_id VARCHAR(128) PRIMARY KEY,
  resource_ids VARCHAR(256),
  client_secret VARCHAR(256),
  scope VARCHAR(256),
  authorized_grant_types VARCHAR(256),
  web_server_redirect_uri VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additional_information VARCHAR(4096),
  autoapprove VARCHAR(256)
);
 
create table oauth_client_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication_id VARCHAR(128) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256)
);
 
create table oauth_access_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication_id VARCHAR(128) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256),
  authentication BLOB,
  refresh_token VARCHAR(256)
);
 
create table oauth_refresh_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication BLOB
);
 
create table oauth_code (
  code VARCHAR(256), authentication BLOB
);
 
create table oauth_approvals (
	userId VARCHAR(256),
	clientId VARCHAR(256),
	scope VARCHAR(256),
	status VARCHAR(10),
	expiresAt TIMESTAMP,
	lastModifiedAt TIMESTAMP
);
 
 
-- customized oauth_client_details table
create table ClientDetails (
  appId VARCHAR(128) PRIMARY KEY,
  resourceIds VARCHAR(256),
  appSecret VARCHAR(256),
  scope VARCHAR(256),
  grantTypes VARCHAR(256),
  redirectUrl VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additionalInformation VARCHAR(4096),
  autoApproveScopes VARCHAR(256)
);

###　然后在eshop_member数据库创建数据表，将客户端信息添加到oauth_client_details表中
![认证](https://img-blog.csdnimg.cn/20190919101412584.png)

如果你的密码不是明文，记得client_secret需要加密后存储。

然后修改代码，配置从数据库读取客户端信息
![认证](https://img-blog.csdnimg.cn/2019091910151732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

接下来启动服务测试即可。
### 获取授权
![获取授权](https://img-blog.csdnimg.cn/20190919101545756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

### 获取用户信息
![获取用户信息](https://img-blog.csdnimg.cn/2019091910160498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)
### 刷新token
![刷新token](https://img-blog.csdnimg.cn/20190919101704552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

### 打开数据表发现token这些信息并没有存到表中，因为tokenStore使用的是redis方式，我们可以替换为从数据库读取。修改配置
![修改配置1](https://img-blog.csdnimg.cn/20190919102123440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)
![修改配置2](https://img-blog.csdnimg.cn/20190919102140868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

###重启服务再次测试
![修改配置3](https://img-blog.csdnimg.cn/20190919102157235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)
![从auth中获取用户](https://img-blog.csdnimg.cn/20190919102207200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1dZQTE5OTM=,size_16,color_FFFFFF,t_70)

###查看数据表，发现token数据已经存到表里了
![查看数据表](https://img-blog.csdnimg.cn/20190919102236199.png)