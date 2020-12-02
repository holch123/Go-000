# 学习笔记

**作业：**

我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

**学习心得：**

sql.ErrNoRows 代表未查询到对应的数据，上层针对此错误，可能不同的业务有不同的处理，需要对这个错误进行断言，而 dao 层的 sql.ErrNoRows 与数据库的具体实现有关，dao 层本身应该是对底层数据库操作的抽象，所以此类错误应该重新声明为自定义错误，并向上传递；其他错误可以进行 Wrap 并抛给上层，在最顶层打印出具体信息。

示例伪代码如下：

```
package dao

import (
	"database/sql"
	xerror "errors"
	"github.com/pkg/errors"
)

var RecordNotFound = xerror.New("record not found")

type Dao struct {
	db *sql.DB
}

func NewDao() *Dao {
	return &Dao{
		db: &sql.DB{},
	}
}

func (d *Dao) Find() (*User, error) {
	user, err := d.db.Query("select * from users where ...")
	if err != nil {
		if xerror.Is(err, sql.ErrNoRows) {
			return nil, RecordNotFound
		}
		return nil, errors.Wrap(err, "find user error")
	}
	return &user, err
}
```
