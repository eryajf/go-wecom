# go-wecom

[![GoDoc][doc-img]][doc] [![Build Status][ci-img]][ci] [![Coverage Status][cov-img]][cov] [![Go Report Card][report-card-img]][report-card]

[doc-img]: https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square
[doc]: https://pkg.go.dev/github.com/wenerme/go-wecom?tab=doc
[ci-img]: https://github.com/wenerme/go-wecom/actions/workflows/ci.yml/badge.svg
[ci]: https://github.com/wenerme/go-wecom/actions/workflows/ci.yml
[cov-img]: https://codecov.io/gh/wenerme/go-wecom/branch/main/graph/badge.svg
[cov]: https://codecov.io/gh/wenerme/go-wecom/branch/main
[report-card-img]: https://goreportcard.com/badge/github.com/wenerme/go-wecom
[report-card]: https://goreportcard.com/report/github.com/wenerme/go-wecom

Wechat Work/Wecom/企业微信 Golang SDK

## 特性

- 支持自建应用开发 - AccessToken
- 支持第三方应用开发 - AuthCorpAccessToken
- 支持缓存所有带时效的信息 - AccessToken, JsTicket, AgentTicket, SuiteToken, AuthCorpAccessToken, PreAuthCode, ProviderAccessToken
  - 缓存支持自定义存储 - 默认内存存储
- 支持从自定义的存储获取 密钥 信息 - SuiteTicket, PermanentCode
- 没有内部状态和 goroutine
- 自动尝试提前获取相应的 Token 和 Ticket - 有效期的 80%
- 实现逻辑清晰 - 没有实现的接口可直接调用
- wwcrypt - 企业微信回调加密实现 - 作用同 sbzhu/weworkapi_golang
- 数据模型大多基于官方接口文档生成 - 包含注释说明
- 包含 API+Event Mock 测试
- 接口
  - 通讯录管理 - 成员、部门、标签、异步批量、互联企业
  - 客户联系 - 客户管理、客户标签管理、在职继承、离职继承、客户群管理、消息推送、统计管理、变更回调
  - 应用
  - 消息会话内容存档
  - 日程、日历
- 事件
  - 通讯录
  - 客户联系
  - 第三方回调
  - 消息
  - 消息回复
  - 企业应用消息推送
  - 外部联系人变更
  - 消息会话内容存档 - msgaudit_notify
  - 日程、日历
- TBD
  - 将企业管理员添加为外部联系人 https://work.weixin.qq.com/api/doc/13613
  - change_external_tag shuffle
  - 批量安装应用 https://open.work.weixin.qq.com/api/doc/20990

```go
package wecom_test

import (
  "fmt"
  "os"

  "github.com/wenerme/go-req"
  "github.com/wenerme/go-wecom/wecom"
)

func ExampleNewClient() {
	// token store - 默认内存 Map - 可以使用数据库实现
	store := &wecom.SyncMapStore{}
	// 加载缓存 - 复用之前的 Token
	if bytes, err := os.ReadFile("wecom-cache.json"); err == nil {
		_ = store.Restore(bytes)
	}
	// 当 Token 变化时生成缓存文件
	store.OnChange = func(s *wecom.SyncMapStore) {
		_ = os.WriteFile("wecom-cache.json", s.Dump(), 0o600)
	}

	client := wecom.NewClient(wecom.Conf{
		CorpID:     "",
		AgentID:    "",
		CorpSecret: "",
		// 不配置默认使用 内存缓存
		TokenProvider: &wecom.TokenCache{
			Store: store,
		},
	})

	// 访问接口会自动获取或使用当前缓存
	token, err := client.AccessToken()
	if err != nil {
		panic(err)
	}
	fmt.Println("Token", token)
	ticket, err := client.JsAPITicket()
	if err != nil {
		panic(err)
	}
	fmt.Println("Ticket", ticket)

	// 访问没有实现的接口
	dto := wecom.IPListResponse{}
	err = client.Request.With(req.Request{
		URL:     "/cgi-bin/get_api_domain_ip",
		Options: []interface{}{
			// 如果不需要 access_token
			// wecom.WithoutAccessToken,
		},
	}).Fetch(&dto)
	if err != nil {
		panic(err)
	}
	fmt.Println("response", dto)
}
```

### 第三方应用开发配置
- 根据使用的接口不同，用到的信息也会不同

```go
client := wecom.NewClient(wecom.Conf{
  CorpID:   "",
  ProviderSecret: "",
  AuthCorpID:   "",
  AuthCorpPermanentCode: "",
  SuiteID:      "",
  SuiteSecret:  "",
  SuiteTicket:  "",
})
```

## Reference

- [wenerme/go-req](https://github.com/wenerme/go-req)
  - 接口底层库
- [xen0n/go-workwx](https://github.com/xen0n/go-workwx)
  - 比较成熟的 Golang 企业微信 SDK
  - 没有 第三方接口、服务商接口、会话存档
- [sbzhu/weworkapi_golang](https://github.com/sbzhu/weworkapi_golang)
  - 官方 Golang 加密库
