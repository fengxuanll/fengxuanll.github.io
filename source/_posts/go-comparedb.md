---
title: Golang - 比较主从数据
date: 2017-09-30 23:56:09
tags: go
categories: go
---

### 说明
公司有两台SQL Server数据库做了发布订阅，之前一直存在偶尔会漏同步数据的问题。
刚好那时候对Go语言挺感兴趣的，所以就用Go写了这个简单的程序来比较主从库的数据，只是简单比较了表行数是否一致。

> 后面已经找到了同步失败的原因，是因为发布的时候，有张表同时存在于了两个发布里，导致总是同步出问题。


<!-- more -->

### 代码
```go
package main

import (
	"database/sql"
	"fmt"
	"strings"

	_ "github.com/denisenkom/go-mssqldb"
)

func main() {
	var tableList = map[string]map[string][]string{
		"ListingPrd": {
			"dbo": []string{"SimpleListingSetting"},
		},
		"OMSPrd": {
			"dbo":     []string{"Customer", "ShipmentOrder"},
			"Sales":   []string{"OptionalOrderTransaction", "Order"},
			"Setting": []string{"PlatformSite", "Store"},
		},
	}

	for dbName, dbItem := range tableList {
		for ownerName, ownerItem := range dbItem {
			for _, tableName := range ownerItem {
				var itemName = strings.Join([]string{dbName, ".", ownerName, ".[", tableName, "]"}, "")
				isEqual, err := compareRows(itemName)
				if err != nil {
					break
				}
				var result = "正常"
				if isEqual != true {
					result = "错误"
				}
				fmt.Println(strings.Join([]string{itemName, "：", result}, ""))
			}
		}
	}

}

func compareRows(itemName string) (bool, error) {
	var mainDbConnectionString = "server=*;port=*;database=*;user id=*;password=*;"
	var subDBConnectionString = "server=*;port=*;database=*;user id=*;password=*;"
	var sqlText = strings.Join([]string{"SELECT COUNT(1) AS Qty FROM ", itemName, " WITH(NOLOCK)"}, "")
	mainQty, err := queryDb(mainDbConnectionString, sqlText)
	if err != nil {
		fmt.Println(strings.Join([]string{itemName, " 主库查询失败："}, ""), err)
		return false, err
	}
	subQty, err := queryDb(subDBConnectionString, sqlText)
	if err != nil {
		fmt.Println(strings.Join([]string{itemName, " 从库查询失败："}, ""), err)
		return false, err
	}

	return mainQty == subQty, nil
}

func queryDb(connectionString string, sqlText string) (int, error) {
	conn, err := sql.Open("sqlserver", connectionString)
	defer conn.Close()

	var qty int
	err = conn.QueryRow(sqlText).Scan(&qty)

	return qty, err
}
```
