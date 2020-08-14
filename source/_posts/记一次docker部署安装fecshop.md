---
title: 记一次docker部署安装fecshop
tags: [docker, fecshop]
id: '361'
categories:
  - docker
date: 2018-12-21 18:28:34
---

最近公司需要在自有app中嵌入一个商城，在结合实际情况调研后选择了开源的商城项**[fecshop](http://www.fecshop.com/ "fecshop")**。

> Fecshop，是由Terry从2015年初开始，一直坚持到今天的开源电商项目，非商业化运作，是真正 开源的电商系统， 遵循BSD-3-Clause开源协议。

**Fecshop 全称为Fancy ECommerce Shop，是基于php Yii2框架之上开发的一款优秀的开源电商系统， 遵循BSD-3-Clause开源协议，Fecshop支持多语言，多货币，架构上支持pc，手机web，手机app，和erp对接等入口，您可以免费快速的定制和部署属于您的电商系统。 截止到2017-10月，appfront（pc前端web），appadmin（后台），apphtml5（wap端web）， appserver（VUE，手机app的api提供端），console（命令行）， appapi（和第三方系统通信端）都已经完成，现在已经趋于稳定， Terry在2010年开始进入跨境电商领域， 用了不少开源电商系统，譬如magento， 发现开源框架都有一定 的诟病，在并发方面差，后期扩展，业务发展后期重构难， 尤其是现在的移动端的发展，多入口的电商模式占据主流， 性能方面的要求越来越高，Fecshop基于Yii2的高效框架，在此基础上进一步封装，加入了 service层和block层，数据库采用了nosql和mysql结合的方式， 关系型表放到mysql中，譬如优惠券，购物车，订单等， 非关系型数据表（非关系型代表不会出现多表强事务类型操作） 放到mongodb中， 缓存用redis，搜索目前用的是mongodb的FullTextSearch功能， 支持一些主流语言的分词与搜索，对于中文搜索，使用的是xunsearch。** fecshop官方提供几种安装方法

*   Docker Compose 方式安装Fecshop(推荐)
    
*   Vagrant Box 方式安装Fecshop（2017.6月做的box，以后不再维护vagrant box）
    
*   linux下单个软件安装（包括mysql、php、mongodb、xunsearch、redis）
    

我这里选择的是以docker-compose方式来安装的，官方git有提供完整的文档，按照文档细心仔细的安装就可以了[fecshop docker安装方式](https://github.com/fecshop/yii2_fecshop_docker "fecshop docker安装方式") 遇到了几个问题：

1.  docker不是虚拟机 ，没有操作docker的经验下必须要有这个概念，它将每一个服务相互隔离开来互不影响，若要对服务进行操作需要通过命令`docker-compose exec xxx(容器名) bash` 进入到服务所在的容器内。
    
2.  docker-compose 将整个运行环境和服务的配置整合在了一起，可以通过docker-compose.yml文件进行配置。
    
3.  在vue前后端分离这个入口的搭建时遇到了跨域问题（应该官方bug）

`Access to XMLHttpRequest at 'http://appserver.fecshoptest.com/cms/home/index' from origin 'http://vue.fecshop.test' has been blocked by CORS policy: Request header field fecshop-currency is not allowed by Access-Control-Allow-Headers in preflight response.` 要在这个文件下修改，新增一个判断options请求https://github.com/fecshop/yii2\_fecshop/blob/master/services/helper/Appserver.php#L183

```php
public function getCors(){
        $fecshop_uuid = Yii::$service->session->fecshop_uuid;
        $cors_allow_headers = [$fecshop_uuid, 'fecshop-lang', 'fecshop-currency', 'access-token'];
        $cors = $this->appserver_cors;
        $corsFilterArr = [];
        if (is_array($cors) && !empty($cors)) {
            if (isset($cors['Origin']) && $cors['Origin']) {
                $corsFilterArr['Origin'] = $cors['Origin'];
            }
            if (isset($cors['Access-Control-Request-Method']) && $cors['Access-Control-Request-Method']) {
                $corsFilterArr['Access-Control-Request-Method'] = $cors['Access-Control-Request-Method'];
            }
            if (isset($cors['Access-Control-Max-Age']) && $cors['Access-Control-Max-Age']) {
                $corsFilterArr['Access-Control-Max-Age'] = $cors['Access-Control-Max-Age'];
            }
            if (isset($cors['Access-Control-Expose-Headers']) && $cors['Access-Control-Expose-Headers']) {
                $cors_allow_headers = array_merge($cors_allow_headers, $cors['Access-Control-Expose-Headers']);
            }
            $corsFilterArr['Access-Control-Expose-Headers'] = $cors_allow_headers;
            //新增这个判断
            if(Yii::$app->request->getMethod() === 'OPTIONS'){
                $corsFilterArr['Access-Control-Request-Headers'] = $cors_allow_headers;
            }
        }
        return $corsFilterArr;

    }
```