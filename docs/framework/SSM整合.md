# SSM整合

## 1、创建工程

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204191259204.png" alt="image-20220419125944656" style="zoom:50%;" />



| 工程名                       | 地位   | 说明                 |
| ---------------------------- | ------ | -------------------- |
| demo-imperial-court-ssm-show | 父工程 | 总体管理各个子工程   |
| demo-module01-web            | 子工程 | 唯一的 war 包工程    |
| demo-module02-component      | 子工程 | 管理项目中的各种组件 |
| demo-module03-entity         | 子工程 | 管理项目中的实体类   |
| demo-module04-util           | 子工程 | 管理项目中的工具类   |
| demo-module05-environment    | 子工程 | 框架环境所需依赖     |
| demo-module06-generate       | 子工程 | Mybatis 逆向工程     |

- 工程间关系

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204190923133.png" alt="image-20220419092333245" style="zoom: 40%;" />

逆向工程后

- 将生成的entity挪到entity工程的包下，
- xml挪到web工程下，
- 接口挪到component工程下

> TIP
>
> 配置文件为什么要放到 Web 工程里面？
>
> - Web 工程将来生成 war 包。
> - war 包直接部署到 Tomcat 运行。
> - Tomcat 从 war 包（解压目录）查找配置文件最直接。
> - 如果不是把配置文件放在 Web 工程，而是放在 Java 工程，那就等于将配置文件放在了 war 包内的 jar 包中。
> - 配置文件在 jar 包中读取相对困难。