---
title: 强大的前后端解耦方式-openapi_generator代码生成
date: 2020-07-17 09:39:48
categories:
  - web
tags:
  - web
  - 效率
---

# 来龙去脉

对接口，接口联调，对字段，这些词汇对一个 web 开发者来说肯定不陌生，而且大多数时候听到这些词脑中就会下意识的觉得烦躁。

作为话不多更偏向就是干的程序员群体，对于联调和对接口这种涉及到大量沟通的事情多有抵触，而且这些事情本身的内容更偏向于劳动型而不是技术型，开发者本身的驱动性也并不强。  
但另一方面这个事情本身却又非常重要。不管是系统与系统之间，还是随着前后端分离的主流，系统内部的前后端开发之间，都涉及到大量的通过接口进行通信的功能。

当然，在代码的世界里，有问题，就绝对不缺少解决方案。多年来为来解决这个问题，社区诞生了很多解决方案。

早期 api 开发人员会手动编写文档，维护一个接口通信文档，谁需要使用 api 服务，就把文档扔给对方，api 有更新时，同时去更新文档。这种方式的缺点太多就不赘述了。 而且，现在都还有使用这种方式的公司，我最近就遇到了。微笑: )

为了解决可读性等问题，出现了类似 RAP,NEI 等 api 管理方案。这些平台提供了单独部署，管理 api 接口并提供清晰的 api 展示界面，mock 数据等功能。提供了非常方便且实用的特性，它们也很够用了。但依然有一点问题，需要 api 开发人员在修改代码后去更新文档，而这一点，难以保证。

swagger 等工具做了这件事，只要你按照它提供的规范编写 api 和注释，写完或更新了 api 的同时，就能自动生成对应的 api 接口文档。  

选用这些方式，适当的删减和组合，就可以大幅度提高接口对接的效率。似乎非常完美了，程序员们都很开心。

但技术的提升没有止境，闲下来的程序员们又开始想，要对接某个服务提供的 api，我要写很多的对接代码，这些代码有些是包装对方的 api，有的是为了做数据类型转换。比如作为一个前端，为了防止传递每个字段时我都需要确认一下它的类型，我想用 typescript 将接口调用进行封装，这样我就可以通过 typescript 的静态检查节约调用时繁琐的调参了。

但这些类型的编程，第一很枯燥，第二包含了大量的样板代码，第三，对方的接口文档已经提供了字段和字段类型，甚至有些告诉了我字段校验方式，我在包装接口时还要再写一次，好烦哦。

到这时，openapi-generator 就登场了。它基于 openapi 规范，为各种语言提供了生成这些代码的工具。 举个例子，api 开发人员写好了接口，此时有一个使用 JavaScript 的团队想要使用，只需要 run 一次 generator 命令，就能得到 typesript 包装好的方法，而且这些方法可以是返回 Promise，可以是 callback，可以是 async，这都由你决定。

# OpenAPI 是什么

其实它的前身就是 Swagger 规范(当然我们现在说到 swagger 更多指的 swagger UI 和 Codegen 等工具)，它是目前描述 RESTful API 最流行的标准。经过 Reverb Technologies 和 SmartBear 等公司多年的发展，后来 SmartBear 捐赠给了 OpenAPI Initiative，现在主要由社区驱动。

OpenApi 是一种规范，它与语言无关。用于描述 RESTful web 服务， 只要你遵照此规范，就能使用相关工具生成文档、创建模拟应用、生成代码。

而 openapi-generator，就是基于 openapi 规范编写的 api 生成代码。

# 举个栗子

说了这么多，那 openapi-generator 生成的代码长啥样呢？

举一个生成 typescript+fetch 的例子，一个登录接口：

```
//api
async login(requestParameters: LoginRequest): Promise<string> {
    const response = await this.loginRaw(requestParameters);
    return await response.value();
}

//model
export interface LoginRequest {
    loginParam?: LoginParam;
}

export interface LoginParam {
    /**
     *
     * @type {string}
     * @memberof LoginParam
     */
    mobile?: string;
    /**
     *
     * @type {string}
     * @memberof LoginParam
     */
    authCode?: string;
}
```

如上，这样你在调用接口的时候就可以不用操心那些琐事了，直接进行声明式的编程：

```
let params:LoginParam = {
  mobile:'18888888888',
  authCode:'000000'
}

const res = yield call(userApi.login, {loginParam:params});
```

而且因为是 ts，在你传递错误参数和参数类型错误时，ts 的静态类型检查会直接告诉你错误。真香。 进一步的，假如将 generator 命令做进 cicd 中，一旦后端修改了接口代码，自动 run 得到新的前端代码，修改部分的错误就会直接暴露出来，直接驱动对接方进行对应的修改。

# openapi-generator 介绍和使用

正式介绍一下正主 openapi-generator。它的[官方网站在这里](https://openapi-generator.tech/), [github 在这里](https://github.com/OpenAPITools/openapi-generator)。

## WHY

为什么要使用 openapi generator 呢，除了功能上的强大，还有一些值得注意的点：

- 完全免费，且使用的是 Apache 许可证 2.0 开发源代码。对于生成的代码文件没有任何约束，你可以任意使用。
- 已经被大量公司在产品中进行使用，这说明了它的功能非常稳定。
- 社区驱动，代码贡献者众多，更新快，维护者积极，生态完备。
- 语言支持非常广泛，基本覆盖所有主流的 api 客户端代码生成，并且还在增加。

## HOW

### 安装

安装命令行工具：  
openapi generator cli 的安装有多种方式

```
//npm package：全局安装最新版本的 openapi generator
npm install @openapitools/openapi-generator-cli -g

//homebrew
brew install openapi-generator
```

还有 docker，jar 包，bash 等方式。在官网就能直接找到。

### 运行

通过 `openapi-generator help`可以看到它所提供的命令
{% asset_img commands.png openapi-generator命令 %}

这里只说最重要的 generate 命令，也就是真正用于生成代码的一个命令。它提供的参数非常多，可以使用`openapi-generator help generate`查看，这里贴出简要版本：
{% asset_img generate-command.png generate命令 %}  
比如我最常使用到的一些参数：

1. -i 指定输入的符合 OpenAPI 规范的文档。openapi generator 代码的生成主要基于 yaml 文件，其中配置了 api 的相关信息。说起来比较干燥，这里有一个[官方提供的 yaml 的例子：petstore.yaml](https://raw.githubusercontent.com/openapitools/openapi-generator/master/modules/openapi-generator/src/test/resources/2_0/petstore.yaml)。对于一个具体的 api，文件中描述了 api 的 http method，描述，方法名，参数名，参数类型，返回值类型，url 等各种信息。
2. -g 指定使用的生成器 generator。 不同的 generator 对应不同的语言和语言内部使用的工具库，比如上面例子中用到的 typescipt-fetch。 还有类似 go,ruby,php,javascript,typescript-axios 等，[这里可以查看全部 generator](https://openapi-generator.tech/docs/generators/)。
3. -o 指定输出目录，即生成的代码放置的位置，默认为当前目录。
4. -p 或者--additional-properties 指定 generate 选项，指定了 generator 使用的配置。比如**allowUnicodeIdentifiers**控制 unicode 的标识是否可以在方法名或参数名中使用，**prependFormOrBodyParameters=true**表示将 form 或者 body 参数添加到参数列表的最上面，**modelPropertyNaming**则控制属性名称的命令方式，可以是 camelCase 驼峰，可以是 snake_case，也可以是 original 保留 api 使用的属性名等，还有**supportsES6=true**表示生成的代码可以使用 es6 特性等等。

一个例子：

```
openapi-generator generate -i http://localhost:xxxx/api/api-docs.yaml  --additional-properties allowUnicodeIdentifiers=true,prependFormOrBodyParameters=true,modelPropertyNaming=original,supportsES6=true -g typescript-fetch -o ./src/open-api
```

### yaml 生成

以应用最广泛的 java 为例，要得到这个 yaml 文件非常简单。只要你会使用 Swagger，就没有任何难点。

如下为 login 接口的 Controller:

```
@PostMapping(value = "/login")
@ApiOperation(value = "登录", httpMethod = "POST", response = String.class)
public String login(@RequestBody LoginParam param) {
    return loginInfoService.login(param.getMobile(), param.getAuthCode());
}
```

然后 swagger 生成的 yaml 元数据文件为：

```
/login:
    post:
      operationId: login
      requestBody:
        content:
          '*/*':
            schema:
              $ref: '#/components/schemas/LoginParam'
      responses:
        200:
          description: default response
          content:
            '*/*':
              schema:
                type: string

components:
  schemas:
    LoginParam:
          type: object
          properties:
            mobile:
              type: string
            authCode:
              type: string
```

这个文件就可以直接交给 openapi-generator 进行代码生成！ 假设访问这个 yaml 文件的 url 为`http://127.0.0.1:8080/v3/api-docs.yaml`。我们在命令中使用-i 指定这个地址即可。

到此就完成了代码的生成，是不是非常的简单？

### 生成的代码

对于生成的代码，我们有时肯定希望做一些修改，直接修改生成的文件肯定是不行的，因为你不可能每次重新生成后都再去改一次。

就 typescript-fetch 这个 generator 生成的代码而言，是支持很多扩展方式的。它暴露了一个 BaseAPI 类来提供这些封装好的 api 方法，而这个 BaseAPI 类除开这些访问 api 的函数外，还提供了如下几个方法：

- constructor

```
constructor(protected configuration = new Configuration()) {
  this.middleware = configuration.middleware;
}
```

实例化这个类的时候可以配置，包括用于在 url 前面添加统一 path 的 basePath, 设置 token 或 apiKey 的 accessToken 和 apiKey，设置 http header 信息的 headers 等。囊括了很多在调用 api 时需要做的自定义操作。比如用 basePath 设置/api 头方便做代理转发，用 headers 方法在 http header 中加入 token 令牌等。

- withMiddleware
  用于注册 fetch 的中间件，分为 withPreMiddleware 和 withPostMiddleware 两种，分别在 fetch 前和 fetch 之后执行。如果你希望在拦截所有请求做一些处理，或者在所有请求返回之后做一些统一的处理，在这里进行相关工作就非常方便。  
  比如，接口统一加'/api'前缀并对返回做错误处理：

```
const interceptFetchRes = (context: RequestContextWithRes) => {
  const { response } = context;

  if (response.status < 200 || response.status >= 300) {
    //统一错误处理
  }else{
    return Promise.resolve(response);
  }
};
let client = new DefaultApi(new Configuration({ basePath: '/api' }));
client = client.withPostMiddleware(interceptFetchRes);
```

# 结束

现在的互联网充满了各种碎片化的数据和服务，基于此也诞生了大量将这些服务进行整合的 mashup 程序。比如打通整个电影院-片单-排片-选座-支付流程的美团，就整合了这条线上各个参与者的接口。而如果大家都使用 openapi 的方式提供 api 服务，可想而知可以提高多少的开发效率。

当然，如果项目内部进行前后端分离，graphQL 这样的新贵也逐渐崭露头角，但目前我在自身对 graphQL 中的实践还处于摸索阶段，暂时就不献丑了～

btw. 电影院终于要重新开工了，憋死我了！
