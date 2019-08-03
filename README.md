# mysql-to-proto

使用grpc过的同学都知道，写proto文件比较繁琐，尤其是写数据库表的message，对应很多字段，为此写了一个简单的从mysql直接读取表结构，生成proto文件的工具。

工具的使用很简单，需要简单的配置，即可运行生成proto文件。

### 使用说明：

```
func main() {
	//模板文件存放路径
	tpl := "d:/gopath/src/mysql-to-proto/template/proto.go.tpl"
	//生成proto文件路径
	file := "d:/gopath/src/mysql-to-proto/sso.proto"
	//数据库名，这里填你自己的数据库名
	dbName := "user"
	//配置连接数据库信息
	db, err := Connect("mysql", "root:123456@tcp(127.0.0.1:3306)/"+dbName+"?charset=utf8mb4&parseTime=true")
	//Table names to be excluded
	//需要排除表，这里的表不会生成对应的proto文件
	exclude := map[string]int{"user_log": 1}
	if err != nil {
		fmt.Println(err)
	}
	if IsFile(file) {
		fmt.Fprintf(os.Stderr, "Fatal error: ", "proto file already exist")
		return
	}
	t := Table{}
	//配置message，Cat 
	t.Message = map[string]Detail{
		"Filter": {
			Name: "Filter",
			Cat:  "custom",
			Attr: []AttrDetail{{
				TypeName: "uint64", //类型
				AttrName: "id",//字段
			}},
		},
		"Request": {
			Name: "Request",
			Cat:  "all",
		},
		"Response": {
			Name: "Response",
			Cat:  "custom",
			Attr: []AttrDetail{
				{
					TypeName: "uint64",
					AttrName: "id",
				},
				{
					TypeName: "bool",
					AttrName: "success",
				},
			},
		},
	}
	//pachage名称
	t.PackageModels = "sso"
	//service名称
	t.ServiceName = "Sso"
	//配置services里面的rpc
	t.Method = map[string]MethodDetail{
		"Get":    {Request: t.Message["Filter"], Response: t.Message["Request"]},
		"Create": {Request: t.Message["Request"], Response: t.Message["Response"]},
		"Update": {Request: t.Message["Request"], Response: t.Message["Response"]},
	}
	//处理数据库表字段属性
	t.TableColumn(db, dbName, exclude)
	//生成proto
	t.Generate(file, tpl)
}
```







































## LICENSE

BSD License <http://creativecommons.org/licenses/BSD/>

Book License: [CC BY-SA 3.0 License](http://creativecommons.org/licenses/by-sa/3.0/)